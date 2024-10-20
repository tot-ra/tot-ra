понедельник, 19 января 2015 г. в 16:37:51

[Vagrant](https://docs.vagrantup.com/v2/getting-started/index.html) это программка управляющая виртуальными машинами, бегающих на VirtualBox. В веб-разработке виртуалки очень полезны тем что среду разработки и весь стэк необходимых системных сервисов можно изолировать для каждого проекта.

Это позволяет избегать проблем с конфликтом версий какой-нибудь OpenSSL библиотеки между разными приложениями. Разработчикам психологически становиться понятней, что именно проект требует. Записывать процесс установки становиться обязательным, зато запуск проекта автоматизируется и сделав раз, новому человеку настроить работу проекта становиться очень легко. Если в среде что-то добавляется или обновляется, то devopsы могут отследить это, ведь файл настроек (Vagrantfile) и файл установки (т.н. [provisioning](https://docs.vagrantup.com/v2/provisioning/shell.html) ) хранятся вместе с проектом в системе версионирования

### Комманды

**vagrant box add** MyCentOS [http://puppet-vagrant-boxes.puppetlabs.com/centos-65-x64-virtualbox-puppet.box](http://puppet-vagrant-boxes.puppetlabs.com/centos-65-x64-virtualbox-puppet.box) — скачивает [основу виртуальной машинки](http://www.vagrantbox.es/), кеширует и обзывает её как вы скажете  
**vagrant init** MyCentOS — генерит файл настроек для проекта, где указываются порты, путь к скрипту установки, общие папки…  
**vagrant up** — стартует виртуальную машину и применяет настройки  
**vagrant ssh** — логинится в работающую ОСь  
vagrant halt — останавливает, полезно если хочется сберечь ресурсы и надо проектом не работаете  
vagrant destroy — удаляет ОСь, но скачанная основа остаётся. Полезно когда вы пишете пишете скрипт для установки и что-то сделали не так, а откатиться не получается

### Vagrantfile

Для поднятия LAMP-проекта я использовал CentOS как вы видите выше

В конфиге самое главное это

- путь к скрипту установки
- перенаправление портов для nginx и mysql
- шаринг папок из самого проекта для nginx с предоставлением возможности записи

```
config.vm.provision "shell", path: "scripts/vagrant_provision.sh" 
config.vm.synced_folder ".", "/var/www/html/", :mount_options => ["dmode=777","fmode=666"]

config.vm.network :forwarded_port, guest: 80, host: 8080
config.vm.network :forwarded_port, guest: 443, host: 8443
config.vm.network :forwarded_port, guest: 3306, host: 3306

config.vm.provider "virtualbox" do |v|
  v.memory = 1024
  v.cpus = 2
end
```

### Provisioning

Shell-скрипт установки использует специальный репозиторий для php-билдов под centos - webtactic. Mysql приходится заменять на более новую версию. Кроме того я перезаписываю конфиг для nginx. Заметьте флаг yum -y, который позволяет избежать интерактивного режима в установке. Мне также пришлось вручную генерировать SSL сертификаты и их устанавливать (но это я опустил тут), т.к. там интерактивного режима избежать не удалось.

Заметьте также что host-ОСь не любит перезаписывать порты меньше 1024, поэтому форвардинг приходится делать через 8k порты + ipfw правила.

Также, скрипт установки имеет смысл группировать по сервисам. Точечную настройку конфиг-файлов имеет смысл заменять с помощью sed'а, либо заготавливать целиком и заменять.

```bash
sudo rpm -Uvh https://mirror.webtatic.com/yum/el6/latest.rpm

#php, extensions
sudo rpm -Uvh https://mirror.webtatic.com/yum/el6/latest.rpm
yum install -y php55w php55w-cli php55w-opcache php55w-fpm php55w-gd php55w-mbstring php55w-mysql php-memcached php55w-pdo php55w-pecl-xdebug php55w-pecl-memcache php55w-xml php55w-pecl-imagick
sudo sed -i 's/upload_max_filesize = 2M/upload_max_filesize = 8M/g' /etc/php.ini
sudo service php-fpm start

#php - mongo extension
sudo yum install -y php55w-devel
sudo pecl channel-update pecl.php.net
printf "\n" | pecl install mongo
sudo touch /etc/php.d/mongo.ini
sudo chown vagrant /etc/php.d/mongo.ini
echo "extension=mongo.so" > /etc/php.d/mongo.ini

#php - composer
curl -sS https://getcomposer.org/installer | php
sudo ln -s /home/vagrant/composer.phar /usr/bin/composer

#php - phpunit
wget https://phar.phpunit.de/phpunit.phar
chmod +x phpunit.phar
sudo mv phpunit.phar /usr/bin/phpunit

#Nginx
sudo cp /var/www/html/scripts/vagrant/nginx.conf /etc/nginx/nginx.conf
sudo service nginx start

#MySQL
yum install -y yum-plugin-replace
yum replace -y mysql-libs --replace-with mysql55w-libs
yum install -y nginx mysql55w-server
sudo service mysqld start
mysql -u root -e "GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root' WITH GRANT OPTION; FLUSH PRIVILEGES;"
sudo service mysqld restart

#MongoDB
sudo yum-config-manager --add-repo=http://downloads-distro.mongodb.org/repo/redhat/os/x86_64
yum install --nogpgcheck -y mongo-10gen mongo-10gen-server mongo-10gen-shell
echo 'db.createCollection("myCustomCollection");' > mongoInit.js
mongo MyMongoDB mongoInit.js
sudo mkdir /data
sudo mkdir /data/db
sudo mongod --profile=1 --slowms=1 --smallfiles & & sleep 3 && mongo MyMongoDB mongoInit.js


#Nodejs, java required for some packages
sudo yum install -y java-1.7.0-openjdk
curl https://raw.githubusercontent.com/creationix/nvm/v0.23.3/install.sh | bash
export NVM_DIR="$HOME/.nvm"
source $HOME/.nvm/nvm.sh
nvm install 0.10
echo ". ~/.nvm/nvm.sh" >> .bash_profile
echo "nvm use 0.10" >> ~/.bash_profile
npm install forever -g
npm install phantomjs -g
npm install grunt-cli -g

#ffmpeg 0.6.5
rpm --import http://packages.atrpms.net/RPM-GPG-KEY.atrpms
sudo rpm -ivh http://dl.atrpms.net/all/atrpms-repo-6-7.el6.x86_64.rpm
sudo yum install -y epel-release ffmpeg ffmpeg-devel
sudo chown vagrant /etc/ld.so.conf
sudo echo "/usr/lib" >> /etc/ld.so.conf
sudo echo "/usr/local/lib" >> /etc/ld.so.conf
sudo ldconfig

php -f /var/www/html/install.php
grunt build
phpunit -c phpunit.xml

#ON HOST MACHINE - CONNECT PORTS (optional)
#sudo ipfw add 100 fwd 127.0.0.1,8080 tcp from any to me 80
#sudo ipfw add 101 fwd 127.0.0.1,8443 tcp from any to me 443
```

Установку с bash можно заменить на puppet или chef.

### Снапшоты

С помощью [плагина](https://github.com/dergachev/vagrant-vbox-snapshot), можно зафиксировать установленную версию, что-бы каждый раз не качать нужные зависимости.

```
vagrant plugin install vagrant-vbox-snapshot
vagrant snapshot take MyCentOS freshInstallation
vagrant snapshot go MyCentOS freshInstallation
```

Если вы часто обновляете виртуалку, тогда имеет смысл файл установки структурировать так, что-бы часто меняющиеся области были внизу, а снапшоты соответсвовали бы глубине изменения этого файла. Например если nginx/mysql/mongo вы не меняете, а расширения для php и его версии меняете часто, то у вас будут снапшоты без пхп и с ним, потому что установка некоторых пакетов может достигать 15 минут и выше.  

Второй вариант - первыми ставить наиболее критичные сервисы и запускать проект как можно быстрее, а потом доустанавливать остальное. В моём случае например, приложение может работать без mongodb куда складируются логи и без ffmpeg, который используется в редких случаях

Если вам понравился [Vagrant](https://www.vagrantup.com/), гляньте на [Docker](https://www.docker.com/) — он изолирует сервисы ещё лучше.

[PuPHPet](https://puphpet.com/) — интересный сервис по упрощению настройки вашей виртуалки

См. также

- [Getting started with Vagrant](http://www.todaysoftmag.com/article/749/getting-started-with-vagrant)
- [Настройка Vagrant для удобной разработки](http://freetonik.com/blog/all/vagrant/)