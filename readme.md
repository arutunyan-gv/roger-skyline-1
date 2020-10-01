# <a name="up">ROGER-SKILINE-1</a>

#ssh #linux #admin #apache


[TOC]

# Основная часть

## Контрольная сумма:

```bash
cd /Users/solid/Dropbox/42/roger-skyline-1/vm/rs1
shasum < rs1.vdi
2b1c69283a4a108be7086c7628af8eb546c7f118  -
```

## Установка гостевой системы

### На память

На память. Конечно больше ни где не использую. Юзеры:

-   `root : admin`
-   `baegon : dragon`
-   `ssh passphrase : pass`
-   Имя гостевой системы: `deb`




### Образ системы

Для `intel mac` подойдет этот:
![](readme/iso.png)

### Разделы диска

Всего 8гб из них:
* 4.2gb /
* 3.2gb /home
* 1.2gb swap

### Пакеты

При выборе пакетов для установки отмечаем:
* SSH
* WEB SERVER

Не отмечаем:
* Графическую оболочку

<span style="color:red">Иначе не хватит места и система выдаст ошибку при установке, но она будет не информативна.</span>

## Настройка

### Установка дополнительных пакетов

Под `root` запускаем следующие команды:
```bash
apt-get update -y
apt-get upgrade -y
apt-get install vim -y
```



### Выдаем права администратора пользователю

Устанавливаем утилиту `sudo`:

```bash
apt-get install sudo -y
```

Редактируем файл `sudoers`:

```bash
cd etc/
chmod +w sudoers
```

Добавляем строку `baegon ALL=(ALL:ALL) ALL`:

![sudoers](readme/sudoers-6881197.png)

## Настройка сети

### Установка статического ip

Ставим сетевые утилиты:
```bash
sudo apt-get install net-tools -y
```

Проверяем состояние сети:
```bash
sudo ifconfig
```

Прописываем настройки статического ip:
```bash
cd /etc/network/interfaces.d
sudo vim enp0s3
```
![](readme/enp0s3.png)

Перезагружаем:
```bash
sudo service networking restart
```

### Настройка порта для SSH подключения

редактируем настоечный файл:
```bash
sudo vim /etc/ssh/sshd_config
```

Отключаем комментарий с строки `#port 22` и выбираем порт `44444`:
![](readme/ssh_config.png)

### Подключение к гостевой системе 

Есть несколько вариантов подключится с Гостевой Системе:

*   `Bridged Adapter` - при изменении расположения может изменится ip адрес хоста и связь будет потеряна;
*   `NAT Network` - Сохранит работоспособность при смене расположения;
*   Требуется найти еще более надежный способ;

#### Bridged Adapter

Выбираем следующие параметры в `VirtualBox`:
![настроки сетевого интерфейса](readme/vb-network-settings.png)

Настраиваем `ip` адрес хоста на `192.168.0.41`
![сетевые настроки хоста](readme/host-network-settings.png)
__Перезагружаем системы!__
Появилась ли виртуальная машина в списке доступных устройств:
![](readme/arp-a.png)
Проверяем доступность: 
![](readme/ping.png)Подключаемся по ssh:
![](readme/ssh.png)

#### NAT Network

1. Настраиваем статический ip как и раньше;
2. Настройки сетевого интерфейса гостевой системы в `VirtualBox`:
 ![](readme/vb_net_new.png)
3. Настройки сети `VirtualBox`: 
![](readme/vb_net_pref_new.png)
4. Настройки маршрутизации: 
![](readme/vb_net_pref_new_routing.png)
5. Смотрим свой `ip` адрес: 
![](readme/mac_net_prefs_new.png)
6. Подключаемся по `SSH` через свой же `ip`:
![](readme/vb_net_new_ssh.png)

![](readme/sudoers.png)

## Безопастность

###  SSH root login disable

Необходимо добавить строку `PermitRootLogin no` в файл `/etc/ssh/sshd_config`

Перезагрузить `sshd`:
```bash
sudo service sshd restart
```

###  SSH Public Key Authentication

Посмотрим что за ключи у нас есть:
```bash
cd ~/.ssh/
ls -la
```

Ключи создаются парами:
* Приватный - остается на клиенте;
* Публичный - отправляется на сервер;

Создадим ключ:
```bash
sudo ssh-keygen -t rsa -f ~/.ssh/vb-roger-syline-1.key -C "key for roger-skiline-1"
```
Где:
* `-f  ~/.ssh/vb-roger-syline-1.key` - название файла;
* `-C "key for roger-skiline-1"` - комментарий;

![](readme/ssh_gen.png)
Посмотрим, что вышло:
```bash
ls -la
```
![ssh_ls_after](readme/ssh_ls_after.png)
Теперь необходимо перенести публичный ключ на удаленную машину:

```bash
sudo ssh-copy-id  -i  ~/.ssh/vb-roger-syline-1.key.pub  baegon@192.168.0.42  -p  44444
```
Где:
* `-i ~/.ssh/vb-roger-syline-1.key.pub` - Выбранный ключ;
* `baegon@192.168.0.42 -p 44444`  - те же логин, адрес и порт, с которыми будет производится  `ssh` подключение;
![ssh_pub_copy](readme/ssh_pub_copy.png)
Подключаемся:
```bash
sudo ssh  baegon@192.168.0.42  -p  44444
```
![ssh_rsa_success](readme/ssh_rsa_success.png)
Статьи:

* <https://www.cyberciti.biz/faq/ubuntu-18-04-setup-ssh-public-key-authentication/>
* <https://www.cyberciti.biz/faq/how-to-set-up-ssh-keys-on-linux-unix/>



###  Закрываем порты

Ставим простой Firewall:
```bash
sudo apt-get install ufw -y
```

Авторизуем подключения и запускам **firewall**:
```bash
sudo ufw allow 44444/tcp #ssh
sudo ufw allow 80/tcp    #http
sudo ufw allow 443       #https
sudo ufw enable
sudo ufw status
```
![ufw_status](readme/ufw_status.png)

### Защита от DOS атак

Статьи:

*   [Fail2Ban Port 80 to protect sites from DOS Attacks](http://www.tothenew.com/blog/fail2ban-port-80-to-protect-sites-from-dos-attacks/)
*   [Настройка Fail2ban](https://vps.ua/wiki/configuring-fail2ban/)
*   [Install fail2ban to protect your site from DOS attacks](https://www.garron.me/en/go2linux/fail2ban-protect-web-server-http-dos-attack.html)
*   https://www.garron.me/en/go2linux/fail2ban-protect-web-server-http-dos-attack.html

Установим необходимые пакеты:

```bash
sudo apt-get install iptables fail2ban apache2
```

Создаем локальный файл с настройками `Fail2ban` и настроим его:

```bash
cd /etc/fail2ban
sudo cp jail.conf jail.local
sudo vim jail.local
```

Создадим фильтр для `ssh`, `http/https`:

```bash
cd /etc/fail2ban/filter.d/
sudo touch http-get-dos.conf
sudo vim http-get-dos.conf
```

Вот, что получилось:

![jail2ban_ssh](readme/jail2ban_ssh.png)
![jail2ban_http](readme/jail2ban_http.png)

Перезагружаем и запускаем сервисы:

```bash
sudo ufw reload
sudo service fail2ban restart
sudo service fail2ban status
```

При наличии ошибок:

```bash
sudo ail2ban-client start
# или
sudo fail2ban-client -vvv start
```

 Заработало:

```bash
sudo fail2ban-client status
```

![fail2ban_ok](readme/fail2ban_ok.png)

### Предотвращаем сканирование портов

Статьи:

*    [To protect against the scan of ports with portsentry](https://en-wiki.ikoula.com/en/To_protect_against_the_scan_of_ports_with_portsentry) 
*    [How to protect against port scanners?](https://unix.stackexchange.com/questions/345114/how-to-protect-against-port-scanners) 

```bash
sudo apt-get install portsentry
```

![portsentry_install](readme/portsentry_install.png)

Устанавливаем режимы `tcp` и `udp`:

```bash
sudo vim /etc/default/portsentry
```

![portsentry_settings](readme/portsentry_settings.png)

Активируем блокировку:

```bash
sudo vim /etc/portsentry/portsentry.conf
```

![portsentry_block](readme/portsentry_block.png)

 Необходимо закомментировать все строки кроме:

*   `KILL_ROUTE="/sbin/iptables -I INPUT -s $TARGET$ -j DROP"`

```bash
sudo vim /etc/portsentry/portsentry.conf
```

Для быстрой навигации по файлу используйте командный режим и паттерны поиска:

*   `/\nKILL`
*   `/DROP`

```bash
cat portsentry.conf | grep KILL_ROUTE | grep -v "#"
```

Перезапускаем:

```bash
sudo /etc/init.d/portsentry start
```

Логи `portsentry` можно посмотреть:

```bash
sudo cat /var/log/syslog
```



## Оптимизация

### Остановка лишних сервисов

Список сервисов:

```bash
ls /etc/init.d
```

![services](readme/services.png)

Отключаем ненужные сервисы:

```
sudo systemctl disable bluetooth.service
sudo systemctl disable console-setup.service
sudo systemctl disable keyboard-setup.service
```

Для проверки запущенный служб:

```bash
sudo service --status-all
```

![services_stat](readme/services_stat.png)

### Настройка CRONTAB

#### Настройка скрипта обновлений

Создадим скрипт с командами на обновления:

```bash
vim update.sh
```

![update_script_log](readme/update_script_log.png)

```bash
sudo sh update.sh
cat /var/log/update_script.log
```

![update_script_log_cat](readme/update_script_log_cat.png)

Создадим задание в `crontub`

```bash
sudo crontab -e
```

![crontab](readme/crontab.png)

Даем права скрипту, без них он не запускался:

```bash
sudo chmod 775 ~/update.sh
```

Ожидать 60 секунд после загрузки `sleep 60` пришлось поставить, так-как без него у `apt-get` не получалось достучатся до репозиториев, видимо сеть не успевала подняться. 



#### Настройка EMAIL уведомлений

Статья:

* [Setting Up Local Mail Delivery on Ubuntu with Postfix and Mutt](https://www.cmsimike.com/blog/2011/10/30/setting-up-local-mail-delivery-on-ubuntu-with-postfix-and-mutt/)

Создадим скрипт:

```bash
vim ~/check_cron.sh
```

![cron_check_sh](readme/cron_check_sh.png)

```bash
sudo chmod 775 ~/check_cron.sh
```

Добавляем в `crontab`:

![crontab_add_last](readme/crontab_add_last.png)

Установим пакеты необходимые для настройки почты:

```bash
 sudo apt install bsd-mailx
 sudo apt install postfix
```

Графический процесс установки:

![mail_gui](readme/mail_gui.png)

Редактируем `/etc/aliases`:

![mail_settings](readme/mail_settings.png)

```bash
sudo newaliases
```

Установка домашний директории для почты:

```bash
sudo postconf -e "home_mailbox = mail/"
sudo service postfix restart
```

Устанавливаем почтовый клиент `mutt`:

```bash
sudo apt install mutt
```

Создадим конфигурационный файл:

```bash
sudo vim /root/.muttrc
```

![mail_sett_2](readme/mail_sett_2.png)

Отправим тестовое письмо:

```bash
echo "Text" | sudo mail -s "Subject" root@debian.lan
```



# Веб часть



## Загркзка сайта



### Установка `git`:

```bash
sudo apt-get install git -y
```

### Загрузка сайта в `~/saturn`:

```bash
cd /var/www/html
sudo rm index.html
git clone https://github.com/liquid245/saturn.git /var/www/html
ls -al
```



### Настройка `ssl` сертификата

*   [Создание самоподписанных сертификатов SSL для Apache в Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-apache-in-ubuntu-18-04-ru)

Создадим сертификат и файл:

 ```bash
baegon@deb:~$ sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt

[sudo] password for baegon: 
Generating a RSA private key
.............................+++++
..............................................................................................................................................................+++++
writing new private key to '/etc/ssl/private/apache-selfsigned.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:RU
State or Province Name (full name) [Some-State]:Moscow
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:42
Organizational Unit Name (eg, section) []:21
Common Name (e.g. server FQDN or YOUR name) []:192.168.0.42
Email Address []:root@deb
 ```

Настроим сервер `apache`:

```bash
sudo touch /etc/apache2/conf-available/ssl-params.conf
```

![apache_ssh_config](readme/apache_ssh_config.png)

```bash
sudo vim /etc/apache2/sites-available/default-ssl.conf 
```

![](readme/default-ssl-conf.png)

```bash
sudo vim /etc/apache2/sites-available/000-default.conf
```

![000-default.conf](readme/000-default.conf.png)

Перезапустим сервисы:

```bash
sudo a2enmod ssl
sudo a2enmod headers
sudo a2ensite default-ssl
sudo a2enconf ssl-params
sudo apache2ctl configtest
sudo systemctl restart apache2
```

Перейдем по ссылке:

*   [Сатурн](http://192.168.0.42)



# Деплоймент

Создадим скрипт:

```bash
cd ~/
vim saturn_update.sh
```



Который будет дергать `git` и обновлять сайт:

```bash
cd /var/www/html
sudo git pull
```



Закинем его на хрон:

```bash
sudo crontab -e
```



Такой записью:

```bash
* * * * * /home/baegon/saturne_update.sh
```



Теперь достаточно просто запушить в репозиторий и в течении минуты сайт обновится.

Конечно это решение подходит только для учебного проекта. Так что дергать кого-то не прилично. 



[Наверх](#up)