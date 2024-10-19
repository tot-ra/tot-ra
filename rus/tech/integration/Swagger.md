пятница, 12 октября 2012 г. в 10:15:54

[Swagger UI](http://swagger.wordnik.com/) это небольшая коллекция скриптов для создания интерактивной документации для API веб-приложений с REST протоколом. Очень полезно, если вы пишете приложение которое должно взаимодействовать с внешней системой, а договориться друг с другом в текстовом формате мало. Интерактивность проявляется в том что из документации можно делать HTTP-запросы, а сама документация зависит от комментариев в вашем коде.

### Визуализатор

Документация основывается на JSON-схеме. Есть [пример кода](https://github.com/wordnik/swagger-core/tree/master/samples/no-server/src/main/html) статичной документации с четырьмя файлами. В общем же случае надо скачать .zip архив последней стабильной ветки [Swagger-UI](https://github.com/wordnik/swagger-ui/downloads), либо делать чекаут мастера. Визуализатор написан на coffee script + backbone.js

Поскольку документация подразумевается динамической, то и генерироваться она должна из кода с аннотациями, а не переписываться каждый раз. В отличие от docblox и прочих документаций, эта расчитана именно на серверное взаимодействие с GET, POST, PUT и DELETE методами и сама позволяет эмулировать запросы.

![](img/Pasted%20image%2020241020020600.png)
### Схема

Описание всех доступных файлов поставляется в [resources.json](https://github.com/wordnik/swagger-core/wiki/Resource-Listing), который ссылается на остальные файлы за деталями..

```
{
  apiVersion: "0.2",
  swaggerVersion: "1.1",
  basePath: "http://petstore.swagger.wordnik.com/api",
  apis: [
    {
      path: "/pet.{format}",
      description: "Operations about pets"
    },
    {
      path: "/user.{format}",
      description: "Operations about user"
    }
  ]
}
```

Сам файл с [детальным описанием](https://github.com/wordnik/swagger-core/wiki/Api-Declaration) повторяет то что в **resources.json**, только теперь со всем списком методов=операций и используемыми входными параметрами..

```
{
  apiVersion: "0.2",
  swaggerVersion: "1.1",
  basePath: "http://petstore.swagger.wordnik.com/api",
  resourcePath: "/pet.{format}",
  
apis:[
  {
    path:"/pet.{format}/{pet_name}",
    description:"Operations about pets",
    operations:[
      {
        httpMethod:"GET",
        nickname:"getPetByName",
        responseClass:"Pet",
        "parameters":[
            {
            "name":"pet_name",
            "description":"Any A-Za-z characters please",
            "required":"true",
            "allowMultiple":"false",
            "dataType":"string",
            "paramType":"path"
            }
        ],
        summary:"Find pet by its name"
        notes: "Only Pets which you have permission to see will be returned",
        errorResponses:[
            {
            "code":"400",
            "reason":"Invalid Name characters provided"
            },
            {
            "code":"404",
            "reason":"No pets were found"
            }
        ]
      }
    ]
  }
  ],
"produces":["application/json"],
"models":[]
}
```

### Генератор  

Стабильные генераторы написаны в основном для java (в т.ч. для Play! фреймворка), scala и node.js. Не густо. Для php есть три независимых библиотеки которые как-то позволяют генерировать JSON-схему для визуализации. Ключевое слово **как-то**.

- [Swagger-PHP](http://packagist.org/packages/zircote/swagger-php) PHP Composer
- [Symfony-2](https://github.com/nelmio/NelmioApiDocBundle) Bundle
- [Restler](https://github.com/Luracast/Restler) PHP framework, swagger support planned in 3.0

Я выбрал первую.. устанавливается легко.

`git clone git@github.com:zircote/swagger-php.git swagger cd swagger composer install`

**Первая проблема** - при запуске генерации json'а на основе аннотаций вылезли баги - сначала что -e (exclude) флага нехватало, потом что require не работает. А работает он так, что инклудит подряд все файлы и начинает с них читать, поэтому если вы инклудите файл с помощью констант каких-то, то он тут же рухнет. Поэтому он должен проходится по чистым классам-контроллерам и на основе них всё дампить.

`php swagger.phar -p ../my_api/ -o ../my_api/doc_json/ -e ../index.php -f`

С этой коммандой, в папке doc_json появятся результаты.

**Вторая проблема** в том как эти результаты подключить к Swagger UI. Дело в том, что обычно json файлы можно генерировать и в корень API, но тогда с Апачем возникает проблема, что не работает mod_rewrite. Во всяком случае у меня. Ведь мне надо что-бы скажем /comment/findByName/ перезаписывалось в index.php. 

Апач же настырно находит comment.json и пытается у файла найти подпапки и падает. Идиотизм конечно, но я ни с какими флагами -F не смог его образумить. Поэтому всё пускаю через index.php. Даже выдачу этих json файлов. Конечно, если у вас сам API называется comment.json то тут уже сложно что придумать.

Схема описанная выше генерируется из аннотаций - resource.php на основе комментария к классу..

```


/**
 *@Swaggerresource(
 *     basePath="http://org.local/v1",
 *     swaggerVersion="1.0",
 *     apiVersion="1"
 * )
 *@Swagger (
 *     path="/leadresponder",
 *     value="Gets collection of leadresponders",
 *     description="This is a long description of what it does"
 *     )
 *@SwaggerProduces (
 *     'application/json',
 *     'application/xml'
 *     )
 *
 * @category   Organic
 * @package    Organic_V1
 * @subpackage Controller
 */
class LeadResponder_RoutesController{}
```

А API на основе комментариев к методам..

```

    /**
     *
     * @PUT
     *@SwaggerPath /{leadresponder_id}
     *@SwaggerOperation(
     *     value="Updates the existing leadresponder designated by the {leadresponder_id}",
     *     responseClass="leadresonder_route",
     *     multiValueResponse=false,
     *     tags="MLR"
     * )
     *@SwaggerError(code=400,reason="Invalid ID Provided")
     *@SwaggerError(code=403,reason="User Not Authorized")
     *@SwaggerError(code=404,reason="Lead Responder Not Found")
     *@SwaggerParam(
     *     description="ID of the leadresponder being requested",
     *     required=true,
     *     allowMultiple=false,
     *     dataType="integer",
     *     name="leadresponder_id",
     *     paramType="path"
     * )
     *@SwaggerParam(
     *     description="leadresponder_route being updated",
     *     required=true,
     *     allowMultiple=false,
     *     dataType="leadresponder_route",
     *     name="leadresponder_route",
     *     paramType="body"
     * )
     * @responseTypeInternal Model_LeadResponder_Route
     */
    public function putAction(){}
```

### Модели

Параметры на выходе (return) для каждого метода прописываются в поле **responseClass**. Это фактически ссылка на возвращаемый тип данных, который может быть примитивом - строкой, числом и тп, либо составным классом. Эти классы описываются в свою очередь в поле **models**.

Модели аналогично объявляются в swagger-php..

```

/**
 * @SwaggerModel(
 *     id="leadresonder_route",
 *     description="some long description of the model"
 * )
 *
 * @property integer $usr_mlr_route_id some long winded description.
 * @property string $route some long description of the model.
 * @property string $createdDate
 * @property array<ref:tag> $tags this is a reference to `tag`
 * @property array<string> $arrayItem This is an array of strings
 * @property array<integer> $refArr This is an array of integers.
 * @property string<'Two Pigs','Duck', 'And 1 Cow'> $enumVal This is an enum value.
 *
 */
class Model_LeadResponder_Route{}
```
