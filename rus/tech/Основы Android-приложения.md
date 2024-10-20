пятница, 3 сентября 2010 г. в 10:52:44

![](img/Pasted%20image%2020241020175759.png)

Android - основанная на linux платформа для мобильных устройств использующая изменённую виртуальную машину Java построенную для учёта компактности файлов и энергоэффективности **Dalvik**. Из-за этого используется не Mobile Edition и тем более не Java SE, а свои библиотеки.

Приложения соответсвенно пишутся на Java, либо же через обходные пути — [Titanium](http://www.appcelerator.com/), [AppInventor](http://www.appinventor.org/), [Adobe AIR](http://labs.adobe.com/technologies/air2/android/). Интересно что из библиотек есть SQLite и OpenGL - не прийдётся изобретать велосипеды. Кроме того в системе есть менеджеры для обмена данными между приложениями, сенсорами и тп.

Многозадачность системы реализуется на программном уровне. Если в ПК приложения постоянно висят в оперативной памяти, а процессор прыгает между ними, то здесь каждое приложение это экземпляр класса Activity который может быть в разных состояниях

Обучающий материал для андроида пишется в двух разных мирах — для игр и для приложений.

<iframe width="816" height="350" src="https://www.youtube.com/embed/RNaOWBBvAGo" title="Android Phone Development - Lecture 1" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Я рассматриваю именно приложения.   Итак - скачиваем и устанавливаем JRE, JDK, SDK, Eclipse + plugin и открываем новый Android-проект. Я думаю до этого вы сами дойдёте. Типичное приложение может  состоять из 4 основных мега-блоков — **Activity, Intent Receiver, Service, Content Provider** которые описываются в AndroidManifest.xml, вместе с запрашиваемыми привилегиями.

#### Делаем страницу (Activity)

_Страница_ делается просто наследованием Activity класса и использовании менеджера ресурсов указывающем на внешний вид

```
public class MyKillerActivity extends Activity { //моя убийственная страничка
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);        //использует шаблон main.xml через менеджер ресурсов
    }
}
```

<iframe width="816" height="350" src="https://www.youtube.com/embed/nhyoGR3ndQE" title="Android Phone Development - Lecture 3" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
### Продумываем внешний вид (View)

Шаблоны отвечающие за внешний вид как и прочие данные лежат в XML-формате в /res папке. Локализация под другие страны и языки делается визардом который просто добавляет префиксы к файлам или папкам.

- layout/*.xml — собственно шаблоны UI элементы со всякими кнопочками и текстовыми полями  
    
- values/strings.xml где лежат переводы для текстов UI
- colors.xml, menu.xml, settings.xml

Для размеров и местоположения элементов используются разрывающие мозг единицы — px, pt, dip, sp.. Для дебага есть [Hierarchy Viewer](http://developer.android.com/guide/developing/tools/hierarchy-viewer.html). В отличие от умного HTML в андроиде  [позиционирование элементов](http://developer.android.com/guide/topics/ui/layout-objects.html) задаётся структурными элементами

- **LinearLayout** — дети ставятся друг за другом внутри родителя в заданном направлении. Если задан weight, то свободное место родителя перераспределяется между детьми  
    
- **RelativeLayout** — дети ставятся относительно родителя или ранее поставленного элемента
- FrameLayout — дети ставятся сверху слева относительно и поверх родителя  
    
- TableLayout — аналог HTML-таблиц но без объединения ячеек
- AbsoluteLayout — устарел из-за разных размеров устройств

#### Наполняем данными (Adapters)

Адаптеры нужны для показа динамического содержания на **наследующих [ListActivity](http://developer.android.com/resources/tutorials/views/hello-listview.html)** страницах, которые могут содержать контейнер элементов — вертикальный список (**ListView**), таблицу (GridView), галерею (Gallery), выпадающий список (Spinner) или сгруппированный список (ExpandableListView). Адаптеры же передают данные из массивов/БД/интернета в шаблон. Связывается адаптер вызовом метода..

```
setListAdapter(new ArrayAdapter(this,  
    android.R.layout.simple_list_item_1,    //встроенный шаблон для каждого ряда
    new String[]{"item 1", "item 2"}        //собственно данные
));
```

Есть много готовых адаптеров — _ListAdapter, [SimpleAdapter](http://megadarja.blogspot.com/2010/07/android.html),_ _[ArrayAdaptor](http://sudarmuthu.com/blog/2010/03/02/using-arrayadapter-and-listview-in-android-applications.html), CursorAdapter, SimpleCursorAdapter..._ и всегда можно написать свой, унаследовав абстрактный BaseAdapter.

При наполнении данными могут понадобится всякие фичи. Например форматирование дат зависит от настроек пользователя..

```
Calendar c = Calendar.getInstance();

c.setTimeInMillis(1000*json_row.getLong("timestamp")); //конвертируем int-timestamp полученный из mysql по json. Из секунд в миллисекунды

android.text.format.DateFormat.getTimeFormat(getApplicationContext()).format(date_added); //время вида 12:10AM

android.text.format.DateFormat.getDateFormat(getApplicationContext()).format(date_added); //дата вида 5/12/2010
```

Или например если вы хотите ссылки в своём TextView, то HTML частично преобразовывается с помощью Html.fromHtml() метода и android:autoLink="web" параметра у элемента. А если надо вручную у "X ребёнка" что-то сделать то есть _getChildAt_ метод.


![](img/Pasted%20image%2020241020175848.png)

![](img/Pasted%20image%2020241020175854.png)

![](img/Pasted%20image%2020241020175900.png)

![](img/Pasted%20image%2020241020175905.png)

С помощью **Intent** пользователя можно переносить на другую страницу, в телефонную книжку, браузер и тп. При этом текущая Activity приостанавливается и в неё можно будет вернуться back-кнопкой.  В передаваемую activity можно передавать дополнительные данные не только в формате key-value но и используя [Bundle](http://www.anddev.org/tinytut_-_passing_data_to_subactivities-t305.html) объект. Вот как это выглядит..

```
startActivity(new Intent(MyKillerActivity.this, MyWeakMinionActivity.class)); //перейдёт к другой странице по именам классов
startActivity(new Intent(Intent.ACTION_VIEW, Uri.parse("http://kurapov.name"))); // открыть сайт в браузере
Intent search = new Intent(Intent.ACTION_WEB_SEARCH);  //браузерный поиск
search.putExtra(SearchManager.QUERY, "куплю снег");
startActivity(search);
```

Ещё один мощный тип управления это [меню](http://developer.android.com/guide/topics/ui/menus.html) (Submenu) выскакивающее при нажатии одноимённой кнопки устройства. Ещё есть его расширение (вертикальный список) и контекстные меню при долгом нажатии на View-элемент, но сейчас не об этом. Меню добавляется как xml файл и прикрепляется к Activity как метод _onCreateOptionsMenu_, а для обработки нажатия используется _onOptionsItemSelected_ метод, куда и нужно вставлять Intent в зависимости от выбранного элемента.  

#### Подгружаем данные (AsyncTask)

К этому моменту у нас есть шаблон, есть возможность передачи данных в него и даже переходы между страницами, но нет самой основы — подгрузки данных из интернета или БД. Первое надо при [простейшей небезопасной авторизации](http://www.instropy.com/2010/06/14/reading-a-json-login-response-with-android-sdk/).. Долгие процессы надо выделять в асинхронные запросы ([AsyncTask](http://developer.android.com/reference/android/os/AsyncTask.html)) что-бы приложение не подвисало. Кроме того к авторизации надо прикрутить SSL, абстрагироваться http-запроса выделив его в отдельный класс если данные будут запрашиваться часто. И встроить в Activity производный от него класс что-бы получать доступ к Activity-методам.. выглядит это в примерно так

![](img/Pasted%20image%2020241020175916.png)

А что-бы процесс было видно пользователю — можно добавить [ProgressBar](http://developer.android.com/reference/android/widget/ProgressBar.html) диалог..

```
this.progressDialog = new ProgressDialog(MyKillerActivity.this); //диалог в контексте страницы

//учитываем локализацию
this.progressDialog.setMessage(getApplicationContext().getResources().getString(R.string.dialog_logging_in));
this.progressDialog.show(); 
...
protected void onPostExecute(Object responce){
    this.progressDialog.dismiss(); //когдм AsyncTask заканчивается - прячем диалог
}
```

#### Храним настройки (SharedPreferences)

Настройки приложения вовсе не значат управление ими пользователем через интерфейс (для этого есть странички типа PreferenceActivity). Настройки это один из видов постоянного [хранения данных](http://developer.android.com/guide/topics/data/data-storage.html) наряду с файлами и SQLite. С их помощью можно хранить данные между запусками

```
SharedPreferences settings = getSharedPreferences("MyKillerAppSettings", MODE_PRIVATE);
settings.edit().putString("MyMinionName", MyKillerActivity.minionName).commit(); //пишем
settings.getString("MyMinionName",""); //читаем
```

Полезно по теме

- [Основные понятия андроид-приложений](http://ondroid.info/osnovnyie-ponyatiya-android-prilozheniy/)
- Дарья Ряжских — [адаптеры](http://megadarja.blogspot.com/2010/07/android.html) и [написание игры с Surface](http://megadarja.blogspot.com/2009/03/android-1-surface.html)
- [Exploring the world of android](http://blog.jteam.nl/2009/09/17/exploring-the-world-of-android-part-2/) и [Как реализовать загрузку изображений](http://habrahabr.ru/blogs/android/78747/) — динамическая подгрузка изображений  
    
- [Getting Reference to Calling Activity from AsyncTask](http://efreedom.com/Question/1-2719433/Getting-Reference-Calling-Activity-AsyncTask-Inner-Class)
![](img/Pasted%20image%2020241020175928.png)
![](img/Pasted%20image%2020241020175932.png)

<iframe width="408" height="260" src="https://www.youtube.com/embed/RNaOWBBvAGo" title="Android Phone Development - Lecture 1" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

<iframe width="408" height="260" src="https://www.youtube.com/embed/MgZHhG9Nrt0" title="Android Phone Development - Lecture 2" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

<iframe width="816" height="350" src="https://www.youtube.com/embed/nhyoGR3ndQE" title="Android Phone Development - Lecture 3" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

