суббота, 15 декабря 2007 г. в 01:04:37

Чем профессиональней становится web-ресурс тем больше необходимость использовать собственных технологий, или по крайней мере своего стиля. Применимо к видео и аудио это значит что внешний вид, функциональность и хостинг - не от youtube/rutube/vimeo а свой. Особенно это заметно когда у вас 500 статей и у большинства - ролики с ютуба, и можно с высокой долей вероятности утверждать что несколько из них уже не работают потому что автор или ютуб их удалил.

Поэтому преимущества держать файл у себя и показывать в своём плеере:

- правовую независимость и надёжность неудаляемости роликов
- инновативность, например показ только собственных связанных видео или HD-канал
- удержание аудитории от публичного сервиса
- свой дизайн и стиль плеера

В качестве примера таких решений в рунете можно привести [ТНТ](http://tht.ru/programs/ComedyClub/video/131/video09.flv) и [Absolute Games](http://ag.ru/).

---

### Аудио-плеер

Для mp3 есть [JW MP3 Player](http://www.jeroenwijering.com/?item=jw_mp3_player) и [Macloo player](http://www.macloo.com/examples/audio_player/), оба на flash естественно. Второй мне особенно показался симпатичным, отчасти из-за его схожести с используемом в сервисе boomp3. Код достаточно простой - в качестве параметра передаётся URL источника. В своём блоге я уже прикрутил - смотрите ниже, как и поиск по эстонско-русскому словарику, но это не в тему. Так что подумайте о ведении подкастинга или видео-кастинга. Уникальность содержимого очень ценится. Есть ещё [dewplayer](http://www.alsacreations.fr/dewplayer-en).

Для того что-бы встроить такой элемент в rss 2.0 надо вставить внутрь item-блока примерно такой код

`<enclosure url="http://kurapov.name/pathkrasivosleva.mp3" length="7332316" type="audio/mpeg" />`

### Видео-плеер

Появление flv-видео значительно укрепило перспективы flash. И хотя для этого надо конвертировать стандартные форматы (avi, mov, mpg) с помощью ffmpeg в качестве консоли на стороне сервера и могут возникнуть проблемы с кодеками, размер и удобство стало очевидным.

|   |   |
|---|---|
Некоторые из плееров
|[Spring player](http://www.flashcomponents.net/component/springplayer_pro_as3_flash_video_player/downloads.html)|HD, поточность, есть поддержка MOV и MP4|
|[flv-mp3.com](http://flv-mp3.com/ru/)|[Никита](http://kitich.ru/blog/2007/12/12/775/) для wordpress советует|
|[Flowplayer](http://flowplayer.org/index.html)|немножко игрушечный|
|[JW FLV Player](http://jeroenwijering.com/?item=JW_FLV_Media_Player)|достаточно заезженный|
|[FLV-player](http://flv-player.net/)|слишком примитивный|
|[uppod](http://uppod.ru/player/intro/)|очень схож с flv-mp3|
|[xmoov](http://xmoov.com/xmoov-flv-player/demo/)|с лишними иконками|
|[Sonnetic](http://sonettic.com/cinema/)|c HD, немного игрушечный|
|[Agriya](http://www.agriya.com/flvPlayerMore.html)|платный|
|[AS-Flash Media player](http://www.flashden.net/item/asflash-media-player/19643?redirect_back=true&clickthrough_id=1559405&ref=djankey)|платный, отличный скин|
|[Proxus FLV component](http://www.proxus.com/components/)||

Сложность возникает с перемоткой, буфферизацией и тп. Здесь нельзя обойтись без возможности сервера в поточной раздаче (streaming), т.е. с подстраиванием под возможности клиента. Для этого нужны специальные серверные программы типа [xmoovStream](http://stream.xmoov.com/) и [mammoth server](http://mammothserver.org/). По этой теме советую послушать [Андрея Смирнова](http://www.smira.ru/2008/09/28/rit-highload-2008/) на РИТ 2007, где он советует модуль для nginx [lighttpd](http://www.lighttpd.net/). Теоретически конвертация из командной строки должна выглядеть так..

`ffmpeg -i movie.[avi] -s 320x240 -ar 44100 -r 12 movie.flv   cat movie.flv | flvtool2 -U stdin movie.flv`

Подробней о конверторах смотрите:

- [ffmpeg](http://ffmpeg.mplayerhq.hu/) (в помощь [ffmpeg-php](http://ffmpeg-php.sourceforge.net/), [getid3](http://www.getid3.org/) )
- [mplayer](http://www.mplayerhq.hu/design7/info.html) (в помощь [flvtool2](http://rubyforge.org/projects/flvtool2/) под ruby)
- [mencoder](http://blog.kovyrin.net/2006/10/08/lighttpd-memcoder-flvtool-for-streaming/lang/ru/)
- [Vixy](http://vixy.net/)-Конвертор flv в родные форматы
- [Sorenson Squeeze](http://www.sorensonmedia.com/) и [Riva FLV encoder](http://www.rivavx.com/) — программки для windows

### Будущее

Если основная функциональность имеется, то во многих видео-сайтах уже прослеживаются инновации типа

- HD-разрешение
- субтитров
- слоёв, зависящих от времени
- комментариев, зависящих от времени
- geo-tagging
- поиск лиц
- полный 360° обзор со специальной камерой

По теме читайте также:

- Расширение для работы php [мета информацией в mp3](http://www.softtime.ru/info/articlephp.php?id_article=64)
- Ярослав Бирзул пишет об [идеальном flv-плеере](http://www.birzool.com/ideal-videoplayer-2/)
- [phpMyTube](http://anton.shevchuk.name/php/php-cookbook-myphptubecom-en/) теоретический проект от Антона Шевчука, выходит в 10 тыс $
- Аналог youtube - [clipshare](http://www.clip-share.com/product/demo/)
- [Wimpy](http://www.wimpyplayer.com/) для музыки, интегрируется на php, asp и coldfusion
- [Создание flv-player'a](http://www.helpexe.ru/effect_18.php) с конвертацией при помощи sothnk swf quicker
- Будущее за большими видео-роликами - [Russia.ru](http://russia.ru/), [vimeo](http://www.vimeo.com/362259), [adobe](http://onair.adobe.com/blogs/videos/2007/06/14/tinic-uro-shows-new-fullscreen-hd-video-in-flash-player/), [madeinrussia](http://madeinrussia.tv/).