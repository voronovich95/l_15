# Cоздать файл config в котором перечислены значения вида key:value, создать скрипт который будет читать этот файл и выводить на экран в виде key=value
```sh
ubuntu@ubuntu:~/lesson15$ cat file.sh
#!/bin/bash
exec 1> info.txt
    for var in  {10..20}
        do
        echo key$var:value$var
    done

ubuntu@ubuntu:~/lesson15$ cat info.txt
key10:value10
key11:value11
key12:value12
key13:value13
key14:value14
key15:value15
key16:value16
key17:value17
key18:value18
key19:value19
key20:value20

ubuntu@ubuntu:~/lesson15$ cat file2.sh
#!/bin/bash
exec 0< info.txt
    while read Line
        do
        echo $Line | sed 's/:/=/g'
    done

ubuntu@ubuntu:~/lesson15$ bash file2.sh
key10=value10
key11=value11
key12=value12
key13=value13
key14=value14
key15=value15
key16=value16
key17=value17
key18=value18
key19=value19
key20=value20
```
# Cоздать файл commands в котором перечислены значения вида name:command, создать скрипт который будет читать этот файл и запускать команды из него
```sh
ubuntu@ubuntu:~/lesson15$ cat file.sh
#!/bin/bash
exec 1> command.txt
    for var in pwd "df -h" lsblk "free -h"  "ls -la"
        do
        echo name: $var
    done

ubuntu@ubuntu:~/lesson15$ cat command.txt
name: pwd
name: df -h
name: lsblk
name: free -h
name: ls -la

ubuntu@ubuntu:~/lesson15$ cat file1.sh
#!/bin/bash
awk -F: '{print $2}' command.txt > 6
exec 0<6
        while read line
        do
        echo -e "\n`$line`\n"
        done

ubuntu@ubuntu:~/lesson15$ bash file1.sh

/home/ubuntu/lesson15


Filesystem                         Size  Used Avail Use% Mounted on
udev                               1.7G     0  1.7G   0% /dev
tmpfs                              346M  756K  345M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv   20G  6.0G   13G  33% /
tmpfs                              1.7G     0  1.7G   0% /dev/shm
tmpfs                              5.0M     0  5.0M   0% /run/lock
tmpfs                              1.7G     0  1.7G   0% /sys/fs/cgroup
/dev/sda2                          974M  149M  759M  17% /boot
tmpfs                              346M     0  346M   0% /run/user/1000


NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                         8:0    0   25G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0    1G  0 part /boot
└─sda3                      8:3    0   24G  0 part
  └─ubuntu--vg-ubuntu--lv 253:0    0   20G  0 lvm  /
sr0                        11:0    1 1024M  0 rom


              total        used        free      shared  buff/cache   available
Mem:           3.4G        120M        2.8G        756K        517M        3.1G
Swap:          3.9G          0B        3.9G


total 44
drwxrwxr-x 2 ubuntu ubuntu 4096 Feb  5 20:40 .
drwxr-xr-x 6 ubuntu ubuntu 4096 Feb  5 15:08 ..
-rw-rw-r-- 1 ubuntu ubuntu   31 Feb  5 20:35 0
-rw-rw-r-- 1 ubuntu ubuntu   31 Feb  5 20:41 6
-rw-rw-r-- 1 ubuntu ubuntu   31 Feb  3 21:52 commandout
-rw-rw-r-- 1 ubuntu ubuntu   56 Feb  3 21:24 command.txt
-rw-rw-r-- 1 ubuntu ubuntu  113 Feb  5 20:40 file1.sh
-rw-rw-r-- 1 ubuntu ubuntu   94 Feb  3 21:53 file1.sh.save
-rw-rw-r-- 1 ubuntu ubuntu  112 Feb  3 18:44 file2.sh
-rw-rw-r-- 1 ubuntu ubuntu  104 Feb  3 18:27 file.sh
-rw-rw-r-- 1 ubuntu ubuntu  154 Feb  3 18:43 info.txt
```
# Открыть консоль и узнать pid текущего процесса оболочки, открыть вторую консоль и отправить в первую консоль любое сообщение с помощью дескрипторов, тем же способом послать вывод команды lsblk
```sh
ubuntu@ubuntu:~$ echo "massage " > /proc/2037/fd/1
ubuntu@ubuntu:~$ massage

ubuntu@ubuntu:~$ ps
  PID TTY          TIME CMD
 2037 pts/0    00:00:00 bash
 2984 pts/0    00:00:00 ps

ubuntu@ubuntu:~$ lsblk > /proc/2037/fd/1

ubuntu@ubuntu:~$ NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                         8:0    0   25G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0    1G  0 part /boot
└─sda3                      8:3    0   24G  0 part
  └─ubuntu--vg-ubuntu--lv 253:0    0   20G  0 lvm  /
sr0                        11:0    1 1024M  0 rom

```
# Cоздать приглашение консоли в виде “полный путь - перенос строки - зеленый знак “$” если не root, а если root то тоже самое но красный цвет и знак “#”
```sh
ubuntu@ubuntu:~$ echo "PS1='\[\e[0m\]`pwd`\[\e[0m\]\n\033[32m$ \033[39m'" >> /home/.bashrc
/home/ubuntu
$

root@ubuntu:~# echo "PS1='\[\e[0m\]`pwd`\[\e[0m\]\n\033[31m# \033[39m'" >> /root/.bashrc
/root 
# 
```
# Cоздать пользователя со стандартным расположением домашней директории и сменить ему домашнюю директорию на /tmp на постоянной основе 
```sh
ubuntu@ubuntu:/home$ sudo useradd -m user1
ubuntu@ubuntu:/home$ sudo passwd user1
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
ubuntu@ubuntu:/home$ su user1
Password:
$ ls -l
total 8
drwxr-xr-x 4 ubuntu ubuntu 4096 Feb  5 21:32 ubuntu
drwxr-xr-x 2 user1  user1  4096 Feb  5 21:36 user1
$ pwd
/home

ubuntu@ubuntu:~$ cat /etc/passwd
_______________________________________________
ubuntu:x:1000:1000:ubuntu:/home/ubuntu:/bin/bash
user1:x:1001:1001::/home/user1:/bin/sh

ubuntu@ubuntu:~$ sudo usermod -d /tmp user1
ubuntu@ubuntu:~$ cat /etc/passwd
________________________________________________
ubuntu:x:1000:1000:ubuntu:/home/ubuntu:/bin/bash
user1:x:1001:1001::/tmp:/bin/sh
```
# создать задание в CRON, запускающее скрипт, выполняющий lsblk с выводом в файл, с подавлением вывода stdout и stderr в самом кроне
```sh
ubuntu@ubuntu:~$ cat lsblk.sh
#!/bin/bash
lsblk > info.txt
ubuntu@ubuntu:~$ chmod +x lsblk.sh
ubuntu@ubuntu:~$ cat info.txt
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                         8:0    0   25G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0    1G  0 part /boot
└─sda3                      8:3    0   24G  0 part
  └─ubuntu--vg-ubuntu--lv 253:0    0   20G  0 lvm  /
sr0                        11:0    1 1024M  0 rom

ubuntu@ubuntu:~$ crontab -l | grep -v "^#"
* * * * * /bin/bash /home/ununtu/lsblk.sh &> /dev/null

ubuntu@ubuntu:~$ journalctl -f
-- Logs begin at Wed 2023-01-18 17:53:14 UTC. --
Feb 05 21:47:11 ubuntu crontab[17474]: (ubuntu) END EDIT (ubuntu)
Feb 05 21:50:46 ubuntu crontab[17548]: (ubuntu) BEGIN EDIT (ubuntu)
Feb 05 21:54:27 ubuntu crontab[17548]: (ubuntu) REPLACE (ubuntu)
Feb 05 21:54:27 ubuntu crontab[17548]: (ubuntu) END EDIT (ubuntu)
Feb 05 21:54:31 ubuntu crontab[17561]: (ubuntu) LIST (ubuntu)
Feb 05 21:54:52 ubuntu crontab[17562]: (ubuntu) LIST (ubuntu)
Feb 05 21:55:01 ubuntu cron[18155]: (ubuntu) RELOAD (crontabs/ubuntu)
Feb 05 21:55:01 ubuntu CRON[17564]: pam_unix(cron:session): session opened for user ubuntu by (uid=0)
Feb 05 21:55:01 ubuntu CRON[17567]: (ubuntu) CMD (/bin/bash /home/ununtu/lsblk.sh &> /dev/null)
Feb 05 21:55:01 ubuntu CRON[17564]: pam_unix(cron:session): session closed for user ubuntu
Feb 05 21:56:01 ubuntu CRON[17575]: pam_unix(cron:session): session opened for user ubuntu by (uid=0)
Feb 05 21:56:01 ubuntu CRON[17576]: (ubuntu) CMD (/bin/bash /home/ununtu/lsblk.sh &> /dev/null)
Feb 05 21:56:01 ubuntu CRON[17575]: pam_unix(cron:session): session closed for user ubuntu
```
# Cкопировать свои скрипты в /tmp и добавить в PATH путь до /tmp, попробовать запускать просто вводя имя скрипта находясь в домашней директории
```sh
ubuntu@ubuntu:~$ cp file1.sh /tmp/
ubuntu@ubuntu:~$ file1.sh
-bash: /tmp/file1.sh: Permission denied
ubuntu@ubuntu:~$ rm file1.sh
ubuntu@ubuntu:~$ chmod +x /tmp/file1.sh

ubuntu@ubuntu:~$ file1.sh

/home/ubuntu


Filesystem                         Size  Used Avail Use% Mounted on
udev                               1.7G     0  1.7G   0% /dev
tmpfs                              346M  756K  345M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv   20G  6.0G   13G  33% /
tmpfs                              1.7G     0  1.7G   0% /dev/shm
tmpfs                              5.0M     0  5.0M   0% /run/lock
tmpfs                              1.7G     0  1.7G   0% /sys/fs/cgroup
/dev/sda2                          974M  149M  759M  17% /boot
tmpfs                              346M     0  346M   0% /run/user/1000


NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                         8:0    0   25G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0    1G  0 part /boot
└─sda3                      8:3    0   24G  0 part
  └─ubuntu--vg-ubuntu--lv 253:0    0   20G  0 lvm  /
sr0                        11:0    1 1024M  0 rom


              total        used        free      shared  buff/cache   available
Mem:           3.4G        116M        3.0G        756K        265M        3.1G
Swap:          3.9G          0B        3.9G


total 68
drwxr-xr-x 6 ubuntu ubuntu 4096 Feb  6 09:48 .
drwxr-xr-x 4 root   root   4096 Feb  5 21:36 ..
-rw-r--r-- 1 root   root    179 Jan 18 18:01 1
-rw-rw-r-- 1 ubuntu ubuntu   36 Feb  6 09:48 6
-rw------- 1 ubuntu ubuntu  565 Feb  5 21:58 .bash_history
-rw-r--r-- 1 ubuntu ubuntu  220 Apr  4  2018 .bash_logout
-rw-r--r-- 1 ubuntu ubuntu 3771 Apr  4  2018 .bashrc
drwx------ 2 ubuntu ubuntu 4096 Jan 18 17:53 .cache
-rw-rw-r-- 1 ubuntu ubuntu   61 Feb  6 09:47 command.txt
-rw-rw-r-- 1 ubuntu ubuntu  129 Feb  6 09:47 file.sh
drwx------ 3 ubuntu ubuntu 4096 Jan 18 17:53 .gnupg
-rw-rw-r-- 1 ubuntu ubuntu  396 Feb  6 09:45 info.txt
drwxrwxr-x 3 ubuntu ubuntu 4096 Feb  5 21:44 .local
-rw-r--r-- 1 ubuntu ubuntu  807 Apr  4  2018 .profile
drwxrwxr-x 2 ubuntu ubuntu 4096 Feb  6 09:35 scrips
-rw-rw-r-- 1 ubuntu ubuntu   66 Feb  5 21:44 .selected_editor
-rw-r--r-- 1 ubuntu ubuntu    0 Jan 18 17:54 .sudo_as_admin_successful
-rw------- 1 ubuntu ubuntu   52 Feb  6 09:32 .Xauthority
```
first
second
third
fourth
fivethsome changes
