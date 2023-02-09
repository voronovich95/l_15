# 1. Внутри ВМ создать 2 контейнера LXD с именами node1 и node2
```sh
root@ubuntu:/home/ubuntu# lxc list
+-------+---------+-----------------------+------+------------+-----------+
| NAME  |  STATE  |         IPV4          | IPV6 |    TYPE    | SNAPSHOTS |
+-------+---------+-----------------------+------+------------+-----------+
| noda1 | RUNNING | 10.138.207.111 (eth0) |      | PERSISTENT | 0         |
+-------+---------+-----------------------+------+------------+-----------+
| noda2 | RUNNING | 10.138.207.185 (eth0) |      | PERSISTENT | 0         |
+-------+---------+-----------------------+------+------------+-----------+
```
# 2. Обеспечить доступ по ssh-ключам
```sh
root@ubuntu:/home/ubuntu# lxc exec noda1 -- sudo /bin/bash
root@noda1:~# cd .ssh
root@noda1:~/.ssh# ll
total 24
drwx------ 2 root root 4096 Jan 30 19:47 ./
drwx------ 6 root root 4096 Jan 30 19:47 ../
-rw------- 1 root root  399 Jan 30 19:47 authorized_keys
-rw------- 1 root root 1679 Jan 30 19:43 id_rsa
-rw-r--r-- 1 root root  402 Jan 30 19:43 id_rsa.pub
-rw-r--r-- 1 root root  222 Jan 30 19:47 known_hosts

root@ubuntu:/home/ubuntu# lxc exec noda2 -- sudo /bin/bas
root@noda2:~# cd .ssh
root@noda2:~/.ssh# ll
total 24
drwx------ 2 root root 4096 Jan 30 19:47 ./
drwx------ 6 root root 4096 Jan 30 19:47 ../
-rw------- 1 root root  402 Jan 30 19:47 authorized_keys
-rw------- 1 root root 1679 Jan 30 19:44 id_rsa
-rw-r--r-- 1 root root  399 Jan 30 19:44 id_rsa.pub
-rw-r--r-- 1 root root  222 Jan 30 19:45 known_hosts
```
# 3. Создать в домашних папках папку backup_logs и выполнить разовую синхронизацию папки /var/log/apt друг на друга в папку backup_logs исключив файл term.log
```sh
root@noda2:~# mkdir backup_logs
root@noda2:~/backup_logs# rsync -a --exclude term.log 10.138.207.111:/var/log/apt/ /root/backup_logs/
root@noda2:~/backup_logs# ll
total 32
drwxr-xr-x 2 root root  4096 Jan 12 15:54 ./
drwx------ 7 root root  4096 Jan 30 20:53 ../
-rw-r--r-- 1 root root 19864 Jan 12 15:54 eipp.log.xz
-rw-r--r-- 1 root root   972 Jan 12 15:54 history.log
root@noda2:~/backup_logs# ls -la /var/log/apt/
total 36
drwxr-xr-x 2 root root    4096 Jan 12 15:54 .
drwxrwxr-x 8 root syslog  4096 Jan 30 20:11 ..
-rw-r--r-- 1 root root   19864 Jan 12 15:54 eipp.log.xz
-rw-r--r-- 1 root root     972 Jan 12 15:54 history.log
-rw-r----- 1 root adm      201 Jan 12 15:54 term.log

root@noda1:~# mkdir backup_logs
root@noda1:~/backup_logs# rsync -a --exclude term.log 10.138.207.185:/var/log/apt/ /root/backup_logs/
root@noda1:~# ls -la /var/log/apt/
total 36
drwxr-xr-x 2 root root    4096 Jan 12 15:54 .
drwxrwxr-x 8 root syslog  4096 Jan 30 20:05 ..
-rw-r--r-- 1 root root   19864 Jan 12 15:54 eipp.log.xz
-rw-r--r-- 1 root root     972 Jan 12 15:54 history.log
-rw-r----- 1 root adm      201 Jan 12 15:54 term.log
root@noda1:~# ls -la backup_logs/
total 32
drwxr-xr-x 2 root root  4096 Jan 12 15:54 .
drwx------ 7 root root  4096 Jan 30 21:18 ..
-rw-r--r-- 1 root root 19864 Jan 12 15:54 eipp.log.xz
-rw-r--r-- 1 root root   972 Jan 12 15:54 history.log
```
# 4. Установить rsyncd и создать группы настроек для папки/var/log/apt
```sh
root@noda1:~# grep -vE "^$" /etc/rsyncd.conf
# Глобальные параметры, отвечающие за поведение демона в целом
pid file = /var/run/rsyncd.pid
# Пользователь, от имени которого ведется работа с файлами
uid = root
gid = root
# Удаленная система может записывать файлы на этот сервер
read only = yes
# исключаем файл term.log
exclude = term.log
[noda2]
comment = noda2 copy directory /var/log/apt on server noda2
# Путь к директории для копирования файлов
path = /var/log/apt
# К этому модулю можно обращаться только с компьютера noda2
hosts allow = 10.138.207.185
hosts deny = *

root@noda2:/home/backup_logs# grep -vE "^$" /etc/rsyncd.conf
# Глобальные параметры, отвечающие за поведение демона в целом
pid file = /var/run/rsyncd.pid
# Пользователь, от имени которого ведется работа с файлами
uid = root
gid = root
# Удаленная система может записывать файлы на этот сервер
read only = yes
[noda1]
comment = copy files from /var/log/apt into /home/backup_logs
# Путь к директории для копирования файлов
path = /home/backup_logs
# К этому серверу можно обращаться только с backup-компьютера
hosts allow  = 10.138.207.111
hosts deny = *
```
# 5. Настроить синхронизацию папки /var/log/apt в кроне с использованием rsyncd исключив файл term.log
```sh
root@noda2:/home/backup_logs# crontab -l | grep -vE "^$|^#"
MAILTO=""
* * * * * rsync -a 10.138.207.111::noda2 /home/backup_logs/

root@noda2:/home/backup_logs# ll
total 36
drwxr-xr-x 2 root root  4096 Jan 31 18:37 ./
drwxr-xr-x 4 root root  4096 Jan 31 21:17 ../
-rw-r--r-- 1 root root 19516 Jan 31 18:37 eipp.log.xz
-rw-r--r-- 1 root root  5147 Jan 31 18:37 history.log
root@noda2:/home/backup_logs# tail -n1 /var/log/syslog
Jan 31 21:48:01 useful-zebra CRON[1622]: (root) CMD (rsync -a 10.138.207.111::noda2 /home/backup_logs/)
```
# 6. Настроить синхронизацию папки /var/log/apt c помощью systemd timers с использованием rsyncd исключив файл term.log
```sh
root@noda2:~# systemctl status rsynd-apt.service
● rsynd-apt.service - /var/log/apt into /home/backup_logs
   Loaded: loaded (/etc/systemd/system/rsynd-apt.service; enabled; vendor preset: enabled)
   Active: activating (start) since Wed 2023-02-01 10:18:06 UTC; 1ms ago
 Main PID: 1558 ((rsync))
    Tasks: 0 (limit: 4069)
   CGroup: /system.slice/rsynd-apt.service
           └─1558 (rsync)

Feb 01 10:18:06 noda2 systemd[1]: Starting /var/log/apt into /home/backup_logs...

root@noda2:~# ll /home/backup_logs/
total 36
drwxr-xr-x 2 root root  4096 Jan 31 18:37 ./
drwxr-xr-x 4 root root  4096 Jan 31 21:17 ../
-rw-r--r-- 1 root root 19516 Jan 31 18:37 eipp.log.xz
-rw-r--r-- 1 root root  5147 Jan 31 18:37 history.log

root@noda2:~# systemctl status rsynd-apt.timer
● rsynd-apt.timer - rsync for backup_logs
   Loaded: loaded (/etc/systemd/system/rsynd-apt.timer; enabled; vendor preset: enabled)
   Active: active (waiting) since Wed 2023-02-01 10:00:09 UTC; 2s ago
  Trigger: Wed 2023-02-01 10:01:00 UTC; 48s left

Feb 01 10:00:09 noda2 systemd[1]: Started rsync for backup_logs.

root@noda2:~# cat /etc/systemd/system/rsynd-apt.service
[Unit]
Description=/var/log/apt into /home/backup_logs
After=network.target
[Service]
Type=oneshot
ExecStart=/usr/bin/rsync -a 10.138.207.111::noda2 /home/backup_logs/
[Install]
WantedBy=multi-user.target

root@noda2:~# cat /etc/systemd/system/rsynd-apt.timer
[Unit]
Description=rsync for backup_logs
Requires=rsynd-apt.service
[Timer]
Unit=rsynd-apt.service
OnCalendar=*-*-* *:*:00
[Install]
WantedBy=timers.target
root@noda2:~#
```
# 2.1 Выведите все номера телефонов.
```sh
root@ubuntu:/home/ubuntu# awk -F: ' {print $2}' info.txt | sed '/^$/d'
(510) 548-1278
(408) 538-2358
(206) 654-6279
(206) 548-1348
(206) 548-1278
(916) 343-6410
(406) 298-7744
(206) 548-1278
(916) 348-4278
(510) 548-5258
(408) 926-3456
(916) 440-1763
```
# 2.2 Выведите номер телефона, принадлежащий сотрудника Dan.
```sh
root@ubuntu:/home/ubuntu# awk '/^Dan/' info.txt | awk -F: '{print $1,$2}'
Dan Savage (406) 298-7744
root@ubuntu:/home/ubuntu# awk '/^Dan/' info.txt | awk -F: '{print $2}'
(406) 298-7744
```
# 2.3 Выведите имя, фамилию и номер телефона сотрудницы Susan.
```sh
root@ubuntu:/home/ubuntu# awk '/^Susan/' info.txt | awk -F: '{print$1,$2'
Susan Dalsass (206) 654-6279
```
# 2.4 Выведите все фамилии, начинающиеся с буквы D.
```sh
root@ubuntu:/home/ubuntu# sed '1,2d' info.txt | awk -F: '{print $1}'|awk -F" " '{print $2}'| awk '/^D/'
Dobbins
Dalsass
```
# 2.5 Выведите все имена, начинающиеся с буквы C или E.
```sh
root@ubuntu:/home/ubuntu# sed '1,2d' info.txt | awk -F: '{print $1}'|awk -F" " '{print $1}'| awk '/^C|^E/'
Christian
Chet
Elizabeth
```
# 2.6 Выведите все имена, состоящие только из четырех букв.
```s
root@ubuntu:/home/ubuntu# sed '1,2d' info.txt | awk -F: '{print $1}'|awk -F" " '{print $1}' | awk 'length ($0) == 4'
Mike
Jody
John
Chet
```
# 2.7 Выведите имена сотрудников, префикс номера телефона которых 916.
```sh
root@ubuntu:/home/ubuntu# sed '1,2d' info.txt | sed -n /916/p | awk '{print $1}'
Guy
John
Elizabeth
```
# 2.8 Выведите денежные вклады сотрудника Mike, предваряя каждую сумму знаком $
```sh
root@ubuntu:/home/ubuntu# sed '1,2d' info.txt | awk '/^M/'|awk -F: '{print "$"$3,"$"$4,"$"$5}'
$250 $100 $175
```


