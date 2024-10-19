среда, 18 апреля 2018 г. в 21:59:16
![](img/Pasted%20image%2020241019193324.png)

После php и node начал писать на go, поэтому по-аналогии с unix-шпаргалкой, выпишу для себя основы..

### Запуск

go run main.go → компиляция и запуск exe  
go build main.go → только компиляция и создание exe, без запуска  
go get -u https://github.com/x/y → импортирование зависимостей

### Переменные и типы
|   |   |   |
|---|---|---|
|тип|декларация|значение по умолчанию|
|целые|int8, int 16, int32, int64, uint, uint64|0||
|бинарные|byte||
|бит|bool|false||
|с плавающей точкой|float, float32, float64|0|
|строка|string, rune|(пустая строка)|

```
var i int = 10;
var myfirstvar, mysecondvar int = 3, 4; // объявление разом
var autoInt = -10 // компилятор сам подберёт тип
var mystring = "test\n" // двойные для строк, всё в уникоде
var mysymbol = '\u2318' // один символ
var rawBinary = '\x27' // одинарные кавычки для символа

myNewDeclared := 42 // быстрая инициализация
myMathVar = 7+3i // математикам
```

### Приведение типов

```
int(myfloat)
float64(myInt)
string(48) //будет номер символа, а не строчка "48"
bin := []byte(mystr) //перевод строки в массив байтов для итерации по байтам
```

### Константа

```
const fullName = "Artjom"

//объявление нескольких переменных с инкрементом
const (
 one = iota + 1 //iota начало итерации с нуля
 _ // пропуск "двойки"
 KB uint64 = 1 << (10*iota)
)
```

### Массив

Низкоуровневый тип. Длина массива фиксирована.

```
var a1 [3]int //длина массива, сразу заполнится [0 0 0]
var a1 [2*size]bool //длина зависит от перменной
a3:=[...]int (1,2,3) //инициализирование значениями
var aa[3][3] int //массив массивов

len(mystring) //длина строк и массивов
```

### Слайс (резиновый массив)

Более сложная структура. Состоит из массива с фиксированной длиной (capacity) в которую записано меньшее количество элементов (length). Как только length превышает capacity, размер capacity удваивается

```
var s1 []int //без размера
append(s1, 100) // добавление
len(s1)
cap(s1) // узнать capacity
s2:= []int(1,2,3,4); //создание со значениями

s1 = append (s1, s2...) //Соединение слайсов через оператор развёртывания значений
s3 := make([]int, 10) //Создание слайса сразу нужной длины, значения уже будут заполнены
s4 := make([]int, 10, 15) // Создание слайса с нужной capacity
copy(s7, s6) // копирование значений, но размер целевого слайса должен совпадать
fmt.Println( s7[1:5] ) // часть слайса

// слайс из массива
a:=[...]int(5,6) 
b:=a[:] //слайс теперь ссылается на массив
```

### Мап (ассоциативный массив)

```
var myEmptyMap map[string]string // [тип ключа] и тип данных, но пустой, в него нельзя писать

var mm1 map[string]string = map[string]string{} //пустой интерфейс
mm2 := map[string]string{}
mm3 = make(map[string]string)

mm2["firstName"] = "Artjom" //write
fmt.Println(mm2["firstName"]) //read
delete(mm3, "somekey") //delete

fmt.Println(mm3["nonexistingkey"]) //выдаст значение по умолчанию (пустая строка)

_, exists = mm3["missing"] // проверка есть ли такой ключ
```

### Ключевые слова в управляющих конструкциях

- fallthrough - проваливание в switch case
- break - выход из  switch
- range - итерирование по символам строки, например

- for index﻿val := range mystr{}

### Функция

init() - вызывается при старте программы в любом из пакетов (порядок неконтролируемый)

```
//аргументы передаются по значению
func avg(x int) float{ //возвращаемый тип в конце
}

func multipleArgs(severalInts ...int){ //развёртывание аргументов в слайс
}

func avg2(y int) (res float){ //сокращение return res в конце
    res := 3
}

func mysql(file File, bool x){
    file.open()
    defer file.close() //заранее объявляет что надо вызвать в конце исполнения функции
    
    if(x) return 0
    //do more lots of stuff
    return 1
}
```

См. также

- [​Go by example](https://gobyexample.com/)
- [Go tour](https://tour.golang.org/list)
- [Slice tricks](https://github.com/golang/go/wiki/SliceTricks)
<iframe width="934" height="350" src="https://www.youtube.com/embed/9Pk7xAT_aCU" title="1. Программирование на Go. Введение" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
