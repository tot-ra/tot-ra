понедельник, 13 апреля 2015 г. в 12:08:42

Если вы заботитесь о качестве своего проекта и кода, то пишете unit-тесты. Но с ними всегда есть «особые случаи». Один из них - работа с файловой системой и ресурсами. Решение «в лоб» - параллельно создавать папку/дерево специально для тестов и надеяться что они не прыгнут на настоящие пути и ничего не удалят.

Более правильный подход - использование in-memory виртуальной файловой системы, vfs. А поскольку ресурсы это по сути потоки, то и название для этой мок-библиотеки - [vfsStream](https://github.com/mikey179/vfsStream/wiki). Ставим..

composer install mikey179/vfsStream

Теперь инициализируем vfs. Заметьте что все функции оперируют в контексте своего корня (root). Если вы будете создавать виртуальные папки без его указания, то и в дереве они не появятся.

```php
public function setUp() {
    //vfsStream::setup();
    //vfsStreamWrapper::register();
    //vfsStreamWrapper::setRoot(vfsStream::newDirectory('root', 0777));
    //$this->rootDir = vfsStreamWrapper::getRoot();
    $this->rootDir = vfsStream::setup('root', 0777);
}
```

  

### Дерево

Теперь в самом тесте надо создать заготовку, изначальное состояние дерева..

```php
vfsStream::create([
    'library' => [
        'bb2075d7d7023ebd5929f6a3f4c4d499' => [
            'original.jpg'=>'erferf',
            'size' => [
                '160.jpg',
                '320.jpg'
                ]
            ]
        ]
    ], 
    $this->rootDir
);
unlink(vfsStream::url('root/library/bb2075d7d7023ebd5929f6a3f4c4d499/size/160.jpg'));

#PHPUnit_Framework_Error_Warning : unlink(vfs://root/library/bb2075d7d7023ebd5929f6a3f4c4d499/size/160.jpg): No such file or directory
```

  

Если бы мы попробовали сразу же удалить файл.. но получили бы ошибку. Всё дело в том что vfs эмулирует и **права и пользователей**. Просто так файл удалить не получится - код его не достучиться своим пользователем. Из этого следует и вторая проблема с быстрым синтаксисом создания дерева - в нём не указываются права по умолчанию.

Самое простое решение - создать всё дерево и потом вручную создать файл с правами 777.

```php
vfsStream::newFile('original.jpg', 0777)->setContent('test')->at(
    $this->rootDir->getChild('library')->getChild('bb2075d7d7023ebd5929f6a3f4c4d499')
    //$this->rootDir->getChildByPath('library/bb2075d7d7023ebd5929f6a3f4c4d499')
);
```

  

Аналогично есть метод **newDirectory** для создания папок. Из-за ограничений прав, возникает проблема - как дерево увидеть. Для этого созданы классы *Visitor. Например можно получить дерево в виде ассоциативного массива..

```php
$result = vfsStream::inspect(new \org\bovigo\vfs\visitor\vfsStreamStructureVisitor())->getStructure();

$this->assertEquals([
    'root' => [
        'library' => [
            'f420b5caa94fb3ac74fe4fb602e38fe8' => []
        ]
    ]
], $result);
```

  

Я в свою очередь [сделал класс](https://github.com/mikey179/vfsStream/pull/111) который распечатывает дерево в виде строки, возможно станет доступным скоро

```
\=root @777
.\=library @777
..\=f420b5caa94fb3ac74fe4fb602e38fe8 @755
```

 

### Тесты

Основная проблема - пути. Библиотечка vfsStream не поддерживает chdir() и realpath()  
Но хуже всего то, что все пути надо обёртывать на такой вот формат, **внутри самого кода**..

vfsStream::url('root/test.txt'); //вида vfs://root/test.txt

Это значит что вы не можете работать с относительными путями, т.е. код вида

unlink("./readme.txt");

Должен обёртываться с помощью какой-то функции. Как следствие - вы не можете соединять части путей (concat), т.к. как только заинжектите путь из теста, получите

PHPUnit_Framework_Error : Object of class orgbovigovfsStreamDirectory could not be converted to string

Как это я обхожу? Во-первых приходится менять код, добавляя обёртку. Это не очень красиво, но в какой-то мере получается абстакция от путей. Для обычного кода возвращается сам путь, для тестов - я могу внедрить свою функцию-трансформер путей

```php
public function removeSizeDir($path){
    if (is_dir($this->fullPath($path . '/size/'))) {
        rmdir($this->fullPath($path . '/size/'));
    }
}

public $pathRewrite = false;

/**
 * Wrap all filesystem access to change path in one place
 * Used actively with unit tests to wrap absolute paths to virtual file system
 *
 * @param string $path
 * @return string
 */
public function fullPath($path){
    if(is_callable($this->pathRewrite)){
        $tmp = $this->pathRewrite;
        return $tmp($path);
    }
    else{
        return $path;
    }
}
```

  

Обёртка путей ставится один раз в setUp и дальше проблем уже нет..

```php
public function setUp() {
//..
    $this->o              = new myObjectUnderTest($this->rootDir);
    $this->o->pathRewrite = function ($path) {
        return vfsStream::url('root/' . $path);
    };
}

/**
 * @test
 */
public function removeSizeDir(){
    vfsStream::create([
            'library' => [
                'bb2075d7d7023ebd5929f6a3f4c4d499' => [
                    'size' => []
                ]
            ]
        ], $this->rootDir
    );
    $this->o->removeSizeDir('library/bb2075d7d7023ebd5929f6a3f4c4d499');
    $result   = vfsStream::inspect(new \org\bovigo\vfs\visitor\vfsStreamAssertVisitor())->getStructure();

        $expected = <<<EOF
\=root @777
.\=library @777
..\=bb2075d7d7023ebd5929f6a3f4c4d499 @755
EOF;
        $this->assertEquals($expected, $result, $result);
}
```

 

Больших и качественных вам покрытий кода :)