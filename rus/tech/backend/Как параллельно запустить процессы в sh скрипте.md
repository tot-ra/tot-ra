вторник, 3 февраля 2015 г. в 12:20:42

Порой надо написать какой-то установочный скрипт, который требует одновременный запуск нескольких задач, из которых некоторые достаточно долгие — сервер, билд проекта. Это не то же самое что [последовательный запуск](http://stackoverflow.com/questions/13077241/execute-combine-multiple-linux-commands-in-one-line) с «&&» или «;» разделителями. Shell-скрипты могут и это с помощью комманды **trap.**

Комманда работает на прерывании процесса (второй параметр). В данном случае это EXIT и TERM. Основная комманда - убиение запущенных этим терминалом в фоне процессов ([jobs -p](http://www.unix.com/man-page/opensolaris/1/jobs/)). Сами они перечисляются дальше с [группировкой фигурными скобками](http://tiswww.case.edu/php/chet/bash/bashref.html#SEC22).

Первый trap таким образом вызывает последующие вызовы. Заметьте что второй вызов с инициализацией базы mongo стоит с задержкой в 3 секунды, что-бы сервер успел подняться (который уже будет бежать бесконечно)

```
echo 'db.createCollection("myCollection");' > mongoInit.js

trap 'echo Starting MongoDB with preinstalled collections; kill $(jobs -p)' EXIT
{ trap '/usr/local/bin/mongod --dbpath myMongoDBStorage' TERM; sleep 5 & wait; } &
{ trap 'sleep 3 && mongo myProjectDB mongoInit.js' TERM; sleep 5 & wait; } &
sleep 1
```

 

На практике получается впрочем так, что комманды бегут бесконечно, т.е. mongod запускается, вы нажимаете Ctrl+C, а он всё ещё работает.. **jobs** ничего не показывает, но если возникает соединение - контекст возвращается. Приходится использовать killall mongod

Вдохновлено [stackoverflow](http://stackoverflow.com/questions/2525855/how-to-propagate-a-signal-through-an-arborescence-of-scripts-bash/3414377#3414377). См. теорию - [Signals](http://tldp.org/LDP/Bash-Beginners-Guide/html/sect_12_01.html) & [Traps](http://tldp.org/LDP/Bash-Beginners-Guide/html/sect_12_02.html)   

upd. как оказалось, более простой и эффективный способ - использовать [одинарный &](http://user.hashcode.ru/questions/4573/bash-%D0%B2%D0%BE%D0%B7%D0%BC%D0%BE%D0%B6%D0%BD%D0%BE-%D0%BB%D0%B8-%D0%B2-linux-%D0%BF%D0%B0%D1%80%D0%B0%D0%BB%D0%BB%D0%B5%D0%BB%D1%8C%D0%BD%D0%BE%D0%B5-%D0%B2%D1%8B%D0%BF%D0%BE%D0%BB%D0%BD%D0%B5%D0%BD%D0%B8%D0%B5-sh-%D1%81%D0%BA%D1%80%D0%B8%D0%BF%D1%82%D0%B0), который [пушит комманды в фоновый режим](http://bashitout.com/2013/05/18/Ampersands-on-the-command-line.html) и запускает дальше следующие

```
/usr/local/bin/mongod --dbpath myMongoDBStorage & sleep 3 && mongo myProjectDB mongoInit.js
```

   

В таком случае jobs показывает работающий mongod и вобщем то его тоже приходится убивать вручную.