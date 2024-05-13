# otus2024_lesson_03

# Первые шаги с Ansible

## Цель домашнего задания
Написать первые шаги с Ansible.

## Описание домашнего задания
Подготовить стенд на Vagrant как минимум с одним сервером. На этом сервере, используя Ansible необходимо развернуть nginx со следующими условиями:
1. Необходимо использовать модуль yum/apt;
2. Конфигурационный файлы должны быть взяты из шаблона jinja2 с переменными;
3. После установки nginx должен быть в режиме enabled в systemd;
4. Должен быть использован notify для старта nginx после установки;
5. Сайт должен слушать на нестандартном порту - 8080, для этого использовать переменные в Ansible;

### Решение

Для выполнения ДЗ будем использовать виртуальную машину с ОС Ubuntu 22.04. \
Разворачивать виртуальную машину будем с помощью Vagrant. Используем box ubuntu/jammy64 Vagrant box v20240426.0.0. \
Box был загружен по ссылке - https://app.vagrantup.com/ubuntu/boxes/jammy64/versions/20240426.0.0/providers/virtualbox/unknown/vagrant.box.

Развернём виртуальную машину с помощью Vagrantfile следующего содержания:
```

# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"

  config.vm.define "otus" do |otus|
    otus.vm.hostname = "otus"
    otus.vm.network "private_network", ip: "192.168.56.11"
    otus.vm.provider "virtualbox" do |virtualbox|
      virtualbox.cpus = "2"
      virtualbox.memory = "2048"
      virtualbox.name = "homework_lesson_03"
    end
  end

end 
```

Далее создадим файл инвентаризации для Ansible. ./inventory/vagrant.ini
```

[webservers]
testserver ansible_port=2222

[webservers:vars]
ansible_host=127.0.0.1
ansible_user=vagrant
ansible_private_key_file=.vagrant/machines/otus/virtualbox/private_key
```

Создадим файл ansible.cfg. Укажем путь до файла инвентаризации и определим несколько дополнительных параметров.
```

[defaults]
inventory = inventory/vagrant.ini
host_key_checking = False
stdout_callback = yaml
callback_enabled = timer
```

Проверим возможность подключения к тестовому серверу.
```

chumaksa@debpc:~/otus2024_linux_professional/homework_lesson_03$ ansible testserver -m ping
testserver | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

Далее создадим файл конфигурации nginx в папке templates. Порт будем передавать в переменной nginx_listen_port.
```

server {
        listen {{ nginx_listen_port }} default_server;
        listen [::]:{{ nginx_listen_port }} default_server ipv6only=on;

        root /usr/share/nginx/html;
        index index.html;

        server_name {{ server_name }};

        location / {
                try_files $uri $uri/ =404;
        }
}
```

Далее создадим playbook.
```

---
- name: Configure webserver with Nginx
  hosts: webservers
  become: true

  vars:
    conf_file: /etc/nginx/sites-available/default
    server_name: localhost
    nginx_listen_port: 8080

  handlers:

    - name: Restart nginx
      service:
        name: nginx
        state: restarted

  tasks:
    - name: Ensure nginx is installed
      package:
        name: nginx
        update_cache: true
      notify: Restart nginx

    - name: Manage nginx config template
      template:
        src: nginx.conf.j2
        dest: "{{ conf_file }}"
        mode: 0644
      notify: Restart nginx
...

```

После окончания работ сценария Ansible проверяем на хосте. Заходим в браузер по http://192.168.56.11:8080/