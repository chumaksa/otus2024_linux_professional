# otus2024_lesson_05

# Работа с mdadm

## Цель домашнего задания
Научиться использовать утилиту для управления программными RAID-массивами в Linux

## Описание домашнего задания
1. Добавить в Vagrantfile еще дисков;
2. Собрать R0/R5/R10 на выбор;
3. Прописать собранный рейд в конф, чтобы рейд собирался при загрузке;
4. Сломать/починить raid;
5. Создать GPT раздел и 5 партиций и смонтировать их на диск;

### Решение

Для выполнения ДЗ будем использовать виртуальную машину с ОС Ubuntu 22.04. \
Разворачивать виртуальную машину будем с помощью Vagrant. Используем box ubuntu/jammy64 Vagrant box v20240426.0.0. \
Box был загружен по ссылке - https://app.vagrantup.com/ubuntu/boxes/jammy64/versions/20240426.0.0/providers/virtualbox/unknown/vagrant.box.

Развёртование виртуальной машины будем выполнять с помощью следующего Vagranfile:
```

# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"

  config.vm.define "otus" do |otus|
    otus.vm.hostname = "otus"
    config.vm.disk :disk, name: "testdisk1", size: "1GB"
    config.vm.disk :disk, name: "testdisk2", size: "1GB"
    config.vm.disk :disk, name: "testdisk3", size: "1GB"
    otus.vm.provider "virtualbox" do |virtualbox|
      virtualbox.cpus = "2"
      virtualbox.memory = "1024"
      virtualbox.name = "homework_lesson_05"
    end
    otus.vm.provision "shell", inline: <<-SHELL
      sudo mdadm --zero-superblock /dev/sd[c-e]
      echo y | sudo mdadm --create /dev/md0 -l 5 -n 3 /dev/sd[c-e]
      sudo mdadm --detail --scan | sudo tee /etc/mdadm/mdadm.conf
      sudo update-initramfs -u
    SHELL
  end

end 
```

После развёртования у нас должна получиться тестовая машина с подключенными тремя дополнительными дисками, \
который собраны в raid 5. Автоматически происходит конфигурирование файла mdadm.conf. Зайдём на машину и проверим. \
```

vagrant@otus:~$ lsblk 
NAME   MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
loop0    7:0    0 63.9M  1 loop  /snap/core20/2264
loop1    7:1    0   87M  1 loop  /snap/lxd/28373
loop2    7:2    0 38.7M  1 loop  /snap/snapd/21465
sda      8:0    0   40G  0 disk  
└─sda1   8:1    0   40G  0 part  /
sdb      8:16   0   10M  0 disk  
sdc      8:32   0    1G  0 disk  
└─md0    9:0    0    2G  0 raid5 
sdd      8:48   0    1G  0 disk  
└─md0    9:0    0    2G  0 raid5 
sde      8:64   0    1G  0 disk  
└─md0    9:0    0    2G  0 raid5 
vagrant@otus:~$ 
vagrant@otus:~$ 
vagrant@otus:~$ cat /etc/mdadm/mdadm.conf 
ARRAY /dev/md0 metadata=1.2 spares=1 name=otus:0 UUID=7c7b6d88:ef59c447:f9939690:7995ad12
```

Далее сломаем наш raid, пометим один из дисков как fail. \
```

vagrant@otus:~$ sudo mdadm /dev/md0 --fail /dev/sdc
mdadm: set /dev/sdc faulty in /dev/md0
vagrant@otus:~$ 
vagrant@otus:~$ 
vagrant@otus:~$ sudo mdadm --detail /dev/md0 
/dev/md0:
           Version : 1.2
     Creation Time : Sun May 19 06:31:57 2024
        Raid Level : raid5
        Array Size : 2093056 (2044.00 MiB 2143.29 MB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 3
     Total Devices : 3
       Persistence : Superblock is persistent

       Update Time : Sun May 19 06:43:41 2024
             State : clean, degraded 
    Active Devices : 2
   Working Devices : 2
    Failed Devices : 1
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : otus:0  (local to host otus)
              UUID : 7c7b6d88:ef59c447:f9939690:7995ad12
            Events : 20

    Number   Major   Minor   RaidDevice State
       -       0        0        0      removed
       1       8       48        1      active sync   /dev/sdd
       3       8       64        2      active sync   /dev/sde

       0       8       32        -      faulty   /dev/sdc
```

Видим, что наш массив остался рабочим, но перешёл в статус degraded. \
Для восстановления вытащим сломанных диск и вставим новый.
```

vagrant@otus:~$ sudo mdadm /dev/md0 --remove /dev/sdc
mdadm: hot removed /dev/sdc from /dev/md0
vagrant@otus:~$ 
vagrant@otus:~$
vagrant@otus:~$ sudo mdadm /dev/md0 --add /dev/sdc
mdadm: added /dev/sdc
vagrant@otus:~$ 
vagrant@otus:~$
vagrant@otus:~$ sudo mdadm --detail /dev/md0 
/dev/md0:
           Version : 1.2
     Creation Time : Sun May 19 06:31:57 2024
        Raid Level : raid5
        Array Size : 2093056 (2044.00 MiB 2143.29 MB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 3
     Total Devices : 3
       Persistence : Superblock is persistent

       Update Time : Sun May 19 06:48:06 2024
             State : clean 
    Active Devices : 3
   Working Devices : 3
    Failed Devices : 0
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : otus:0  (local to host otus)
              UUID : 7c7b6d88:ef59c447:f9939690:7995ad12
            Events : 40

    Number   Major   Minor   RaidDevice State
       4       8       32        0      active sync   /dev/sdc
       1       8       48        1      active sync   /dev/sdd
       3       8       64        2      active sync   /dev/sde
```

После добавления нового диска массив переходи в статус recovering и через некоторое время полностью восстанавливается.

Далее нам необходимо создать GPT и дополнительно 5 разделов на нашем raid 5. \
Для теста создадим 5 разделов по 200M.
```

vagrant@otus:~$ sudo fdisk /dev/md0 

Welcome to fdisk (util-linux 2.37.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0xda74c099.

Command (m for help): g
Created a new GPT disklabel (GUID: BEFD693C-10E6-904E-A3B1-64FF7C251BAA).

Command (m for help): n
Partition number (1-128, default 1): 
First sector (2048-4186078, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-4186078, default 4186078): 200M
Value out of range.
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-4186078, default 4186078): +200M

Created a new partition 1 of type 'Linux filesystem' and of size 200 MiB.

Command (m for help): n
Partition number (2-128, default 2): 
First sector (411648-4186078, default 411648): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (411648-4186078, default 4186078): +200M

Created a new partition 2 of type 'Linux filesystem' and of size 200 MiB.

Command (m for help): n
Partition number (3-128, default 3): 
First sector (821248-4186078, default 821248): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (821248-4186078, default 4186078): +200M

Created a new partition 3 of type 'Linux filesystem' and of size 200 MiB.

Command (m for help): n
Partition number (4-128, default 4): 
First sector (1230848-4186078, default 1230848): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (1230848-4186078, default 4186078): +200M

Created a new partition 4 of type 'Linux filesystem' and of size 200 MiB.

Command (m for help): n
Partition number (5-128, default 5): 
First sector (1640448-4186078, default 1640448): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (1640448-4186078, default 4186078): +200M

Created a new partition 5 of type 'Linux filesystem' and of size 200 MiB.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

Выполним проверку.
```

vagrant@otus:~$ sudo fdisk -l /dev/md0
Disk /dev/md0: 2 GiB, 2143289344 bytes, 4186112 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 524288 bytes / 1048576 bytes
Disklabel type: gpt
Disk identifier: BEFD693C-10E6-904E-A3B1-64FF7C251BAA

Device       Start     End Sectors  Size Type
/dev/md0p1    2048  411647  409600  200M Linux filesystem
/dev/md0p2  411648  821247  409600  200M Linux filesystem
/dev/md0p3  821248 1230847  409600  200M Linux filesystem
/dev/md0p4 1230848 1640447  409600  200M Linux filesystem
/dev/md0p5 1640448 2050047  409600  200M Linux filesystem
```

Видим, что наш диск (массив raid 5) имеет таблицу разделов type: gpt и дополнительные 5 разделов. \
Далее создадим на разделах файловую систему ext4.
```

vagrant@otus:~$ sudo mkfs.ext4 /dev/md0p1
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 51200 4k blocks and 51200 inodes
Filesystem UUID: 734928cc-6413-4112-aca0-e6cb91d2ecfe
Superblock backups stored on blocks: 
	32768

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

vagrant@otus:~$ sudo mkfs.ext4 /dev/md0p2
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 51200 4k blocks and 51200 inodes
Filesystem UUID: d2a576b0-4528-4b65-903b-608a4124a6d2
Superblock backups stored on blocks: 
	32768

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

vagrant@otus:~$ sudo mkfs.ext4 /dev/md0p3
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 51200 4k blocks and 51200 inodes
Filesystem UUID: 51c3c6eb-f3bf-420b-b907-e59e2b636920
Superblock backups stored on blocks: 
	32768

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

vagrant@otus:~$ sudo mkfs.ext4 /dev/md0p4
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 51200 4k blocks and 51200 inodes
Filesystem UUID: 43be87d2-a09b-4d28-84f9-a43c19c5f7d3
Superblock backups stored on blocks: 
	32768

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

vagrant@otus:~$ sudo mkfs.ext4 /dev/md0p5
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 51200 4k blocks and 51200 inodes
Filesystem UUID: b4e9a832-14b8-4281-bb84-6863ba211378
Superblock backups stored on blocks: 
	32768

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done
```

Проверим успешность создания файловой системы.
```

vagrant@otus:~$ lsblk -f /dev/md0
NAME    FSTYPE FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
md0                                                                            
├─md0p1 ext4   1.0         734928cc-6413-4112-aca0-e6cb91d2ecfe                
├─md0p2 ext4   1.0         d2a576b0-4528-4b65-903b-608a4124a6d2                
├─md0p3 ext4   1.0         51c3c6eb-f3bf-420b-b907-e59e2b636920                
├─md0p4 ext4   1.0         43be87d2-a09b-4d28-84f9-a43c19c5f7d3                
└─md0p5 ext4   1.0         b4e9a832-14b8-4281-bb84-6863ba211378                
```

Далее создадим 5 директорий, которые будем использовать в качестве точек монитрования и \
выполним монтирование наших файловых систем на разделах.
```

vagrant@otus:~$ sudo mkdir -p /mnt/dir{1,2,3,4,5}
vagrant@otus:~$ ls -1 /mnt/
dir1
dir2
dir3
dir4
dir5
vagrant@otus:~$ sudo mount -t ext4 /dev/md0p1 /mnt/dir1
vagrant@otus:~$ sudo mount -t ext4 /dev/md0p2 /mnt/dir2
vagrant@otus:~$ sudo mount -t ext4 /dev/md0p3 /mnt/dir3
vagrant@otus:~$ sudo mount -t ext4 /dev/md0p4 /mnt/dir4
vagrant@otus:~$ sudo mount -t ext4 /dev/md0p5 /mnt/dir5
vagrant@otus:~$ lsblk -f /dev/md0
NAME    FSTYPE FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
md0                                                                            
├─md0p1 ext4   1.0         734928cc-6413-4112-aca0-e6cb91d2ecfe  157.3M     0% /mnt/dir1
├─md0p2 ext4   1.0         d2a576b0-4528-4b65-903b-608a4124a6d2  157.3M     0% /mnt/dir2
├─md0p3 ext4   1.0         51c3c6eb-f3bf-420b-b907-e59e2b636920  157.3M     0% /mnt/dir3
├─md0p4 ext4   1.0         43be87d2-a09b-4d28-84f9-a43c19c5f7d3  157.3M     0% /mnt/dir4
└─md0p5 ext4   1.0         b4e9a832-14b8-4281-bb84-6863ba211378  157.3M     0% /mnt/dir5
```