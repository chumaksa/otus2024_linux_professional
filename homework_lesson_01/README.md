# otus2024_lesson_01

# Настройка ПК для создания стендов и выполнения домашних заданий

## Цель домашнего задания
Настроить среду для выполнения домашних работ.

## Описание домашнего задания
Подготовка ПК для выполнения домашних работ.

В этом методическом указании рассмотрены основные инструкции по установке   программ, которые понадобятся для выполнения домашних работ.
* Установка среды виртуализации VirtualBox.
* ПО для настройки виртуальной среды на базе VirtualBox и Hashicorp Vagrant.
* Среда автоматического конфигурирования — Ansible.
* Дополнительные программы, которые могут потребоваться для выполнения домашних работ.

### Решение

Для выполнения ДЗ будем использовать ОС Debian 12. Выполним подготовку.

Обновим список пакетов:
```

chumaksa@debpc:~$ sudo apt update
[sudo] password for chumaksa: 
Hit:1 http://ftp.ru.debian.org/debian bookworm InRelease
Get:2 http://ftp.ru.debian.org/debian bookworm-updates InRelease [55.4 kB]
Hit:3 http://security.debian.org/debian-security bookworm-security InRelease
Fetched 55.4 kB in 1s (97.8 kB/s)                        
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
80 packages can be upgraded. Run 'apt list --upgradable' to see them.
```

Установим текстовый редактор Vim:
```

chumaksa@debpc:~$ sudo apt install vim
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
vim is already the newest version (2:9.0.1378-2).
0 upgraded, 0 newly installed, 0 to remove and 80 not upgraded.
```

Далее установим дополнительные программы для анализа сетевого трафика:
```

chumaksa@debpc:~$ sudo apt install traceroute net-tools tcpdump 
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
traceroute is already the newest version (1:2.1.2-1).
The following NEW packages will be installed:
  net-tools tcpdump
0 upgraded, 2 newly installed, 0 to remove and 80 not upgraded.
Need to get 710 kB of archives.
After this operation, 2,365 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://ftp.ru.debian.org/debian bookworm/main amd64 net-tools amd64 2.10-0.1 [243 kB]
Get:2 http://ftp.ru.debian.org/debian bookworm/main amd64 tcpdump amd64 4.99.3-1 [467 kB]
Fetched 710 kB in 1s (847 kB/s)  
Selecting previously unselected package net-tools.
(Reading database ... 190121 files and directories currently installed.)
Preparing to unpack .../net-tools_2.10-0.1_amd64.deb ...
Unpacking net-tools (2.10-0.1) ...
Selecting previously unselected package tcpdump.
Preparing to unpack .../tcpdump_4.99.3-1_amd64.deb ...
Unpacking tcpdump (4.99.3-1) ...
Setting up tcpdump (4.99.3-1) ...
Setting up net-tools (2.10-0.1) ...
Processing triggers for man-db (2.11.2-2) ...
```

Установим программы для передачи файлов:
```

chumaksa@debpc:~$ sudo apt install curl wget
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
curl is already the newest version (7.88.1-10+deb12u5).
wget is already the newest version (1.21.3-1+b2).
0 upgraded, 0 newly installed, 0 to remove and 80 not upgraded.
```

Установим программу контроля версий:
```

chumaksa@debpc:~$ sudo apt install git
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
git is already the newest version (1:2.39.2-1.1).
0 upgraded, 0 newly installed, 0 to remove and 80 not upgraded.
```

В качестве редактора кода будем использовать Visual Studio Code. Загрузим установочный пакет и выполним его установку:
```

chumaksa@debpc:~$ sudo dpkg -i Downloads/code_1.89.0-1714530869_amd64.deb 
Selecting previously unselected package code.
(Reading database ... 190191 files and directories currently installed.)
Preparing to unpack .../code_1.89.0-1714530869_amd64.deb ...
Unpacking code (1.89.0-1714530869) ...
Setting up code (1.89.0-1714530869) ...
Processing triggers for gnome-menus (3.36.0-1.1) ...
Processing triggers for desktop-file-utils (0.26-1) ...
Processing triggers for mailcap (3.70+nmu1) ...
Processing triggers for shared-mime-info (2.2-1) ...
```

Далее выполним установку Oracle VirtualBox:
```

wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] http://download.virtualbox.org/virtualbox/debian $(lsb_release -cs) contrib"
sudo apt install -y virtualbox-7.0
```

Установка Hashicorp Vagrant:
```

chumaksa@debpc:~$ vagrant --version
Vagrant 2.4.0
```

Устанавливаем Ansible:
```

chumaksa@debpc:~$ sudo apt install ansible
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
ansible is already the newest version (7.3.0+dfsg-1).
0 upgraded, 0 newly installed, 0 to remove and 80 not upgraded.
chumaksa@debpc:~$ 
chumaksa@debpc:~$ 
chumaksa@debpc:~$ ansible --version
ansible [core 2.14.3]
  config file = None
  configured module search path = ['/home/chumaksa/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  ansible collection location = /home/chumaksa/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.11.2 (main, Mar 13 2023, 12:18:29) [GCC 12.2.0] (/usr/bin/python3)
  jinja version = 3.1.2
  libyaml = True
```