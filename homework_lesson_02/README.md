# otus2024_lesson_02

# Vagrant-стенд для обновления ядра и создания образа системы

## Цель домашнего задания
Научиться обновлять ядро в ОС Linux. Получение навыков работы с Vagrant.

## Описание домашнего задания
1. Запустить ВМ с помощью Vagrant.
2. Обновить ядро ОС из репозитория.
3. Оформить отчет в README-файле в GitHub-репозитории.

### Решение

Для выполнения ДЗ будем использовать виртуальную машину с ОС Ubuntu 22.04. \
Разворачивать виртуальную машину будем с помощью Vagrant. Используем box ubuntu/jammy64 Vagrant box v20240426.0.0. \
Box был загружен по ссылке - https://app.vagrantup.com/ubuntu/boxes/jammy64/versions/20240426.0.0/providers/virtualbox/unknown/vagrant.box.

Обновлять ядро будем с помощью утилиты mainline из репозитория ppa:cappelikan/ppa. \
В Vagrantfile добавим необходимые команды для provisioning. Итоговый Vagrantfile:
```

# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"

  config.vm.define "otus" do |otus|
    otus.vm.hostname = "otus"
    otus.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
    end
    otus.vm.provision "shell", inline: <<-SHELL
      uname -a
      sudo add-apt-repository ppa:cappelikan/ppa
      sudo apt update
      sudo apt install -y mainline
      sudo mainline install-latest
      sudo systemctl start reboot.target
    SHELL
  end

end
```

Выполняем запуск и проверяем. Выполняем vagrant up и смотрим на вывод. \
После создания виртуальной машины начинается provisioning. \
Видим, что изначально ядро имеет версию 5.15.0.
```
otus: Linux otus 5.15.0-105-generic #115-Ubuntu SMP Mon Apr 15 09:52:04 UTC 2024 x86_64 x86_64 x86_64 GNU/Linux
```

Далее выполняется добавление репозитория ppa:cappelikan/ppa, обновление всех пакетов и установка mainline. \
Далее выполняется загрузка и установка нового ядра.
```

    otus: mainline 1.4.10
    otus: Updating Kernels...
▓▓▓▓▓▓▓▓░░░░░░░░░░░░░░░░░░░░░░░░░░░ 23%                               
▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓░░░░░░░░░░░░░░░░░░░░ 45%                               
▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓░░░░░░░░░░░░░░░░ 54%                               
▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓░░░░░░░░░░░░░ 63%                               
▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓░░░░░░░░░░░░ 67%                               
▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓░░░░░░░░░ 75%                               
▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓░░░░░░░ 82%                               
▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓░░░ 92%                               
Downloading 6.8.8  
```
После установки нового ядра система перезагружается. \
Далее заходим с помощью vagrant ssh на нашу виртуалку и проверяем версию ядра.
```

vagrant@otus:~$ uname -a
Linux otus 6.8.8-060808-generic #202404271536 SMP PREEMPT_DYNAMIC Sat Apr 27 15:54:17 UTC 2024 x86_64 x86_64 x86_64 GNU/Linux
```

Видим, что наше ядро успешно обновилось до версии 6.8.8.