# Автоматизация развёртывания инфраструктуры веб-сервиса с использованием оркестратора Ansible и реализация плана её аварийного восстановления.
## Задание
&ensp;&ensp;Развернуть веб-проект отвечающий следующим требованиям:
  - включен HTTPS;
  - основная инфраструктура находится в DMZ;
  - настроен firewall;
  - настроен сбор логов и мониторинг с уведомлениями;
  - организован бэкап.<br/>
## Ход выполнения  
&ensp;&ensp;Развёртывание необходимой для выполнения проекта инфраструктуры проводились с использованием Vagrant 2.4.0, VirtualBox 7.0.18, Ansible 9.4.0 и образа ubuntu/jammy64 версии 20240301.0.0.<br/> 
&ensp;&ensp;Стенд с проектом представлен шестью серверами:
  - сервер балансировки нагрузки NGINX (hostname: **frontweb**, IP: 192.168.1.145, 192.168.56.145);
  - первый бэкенд-сервер Apache (hostname: **backend1**, IP: 192.168.1.222);
  - второй бэкенд-сервер Apache (hostname: **backend2**, IP: 192.168.1.212);
  - основной сервер базы данных (БД) MySQL (hostname: **mastersdb**, IP: 192.168.1.69);
  - cервер репликации БД MySQL (hostname: **slavesdb**, IP: 192.168.1.98);
  - cервер мониторинга и логирования (hostname: **monitoring**, IP: 192.168.1.60).
### Схема проекта
![Prof_project](https://github.com/user-attachments/assets/3ff4e6fd-d107-4537-bd4b-0a7761463ecb)

&ensp;&ensp; Развёртывание стенда осуществляется командой:
```shell
vagrant up
```
&ensp;&ensp;Перечень устанавливаемого программного обеспечения (ПО) для каждого из серверов:
- **frontweb** (192.168.1.145, 192.168.56.145):<br/>
  nginx, prometheus-node-exporter, filebeat_8.9.1, mc;
- **backend1** (192.168.56.222):<br/>
  apache2, php, php-mysql, libapache2-mod-rpaf, libapache2-mod-php, php-cli, php-cgi, php-gd,<br/>
  wordpress_6.6.2, prometheus-node-exporter, mc;
- **backend2** (192.168.56.212):<br/>
apache2, php, php-mysql, libapache2-mod-rpaf, libapache2-mod-php, php-cli, php-cgi, php-gd,<br/>
wordpress_6.6.2, prometheus-node-exporter mc;
- **mastersdb** (192.168.56.69):<br/>
  mysql-server-8.0, prometheus-node-exporter, mc;
- **slavesdb** (192.168.56.98):<br/>
  mysql-server-8.0, prometheus-node-exporter, mc;
- **monitoring** (192.168.56.90):<br/>
  prometheus, prometheus-alertmanager, prometheus-node-exporter, default-jdk, grafana_10.2.2,<br/>
  logstash-8.9.1, elasticsearch-8.9.1, kibana-8.9.1, mc.

&ensp;&ensp;Конфигурирование серверов производится посредством оркестратора Ansible. Для этого подготовлены соответствующие сценарии в виде ролей и плэйбук provision.yml.<br/>
&ensp;&ensp;Запуск плейбука осуществляется командой:
```shell
ansible-playbook provision.yml
```
&ensp;&ensp;Структура ansible-сценария представлена на диаграмме:
```shell
├── ansible.cfg
├── backend
├── frontweb
├── hosts
├── mastersdb
├── monitoring
├── preinstall
├── provision.yml
├── slavesdb
└── vault_pass
```
- **ansible.cfg** - файл конфигурации Ansiblel;
- **hosts** - файл реестра;
- **provision.yml** - плейбук Ansible;
- **vault_pass** - файл с паролем, необходимым для шифрования посредством ansible-vault. 
### Реализованные роли Ansible:
**preinstall**<br/>
&ensp;&ensp;Первоначальная настройка серверов инфраструктуры:
- настройка часового пояса;
- установка prometheus-node-exporter;
- установка программы mc.<br/>

**frontweb** <br/>
&ensp;&ensp;Настройка фронтенд сервера NGINX:
- установка и конфигурирование веб-сервера NGINX;
- установка filebeat;
- настройка правил iptables.<br/>

**backend**<br/>
&ensp;&ensp;Настройка бэкенд серверов:
- установка и конфигурирование Apache;
- установка и конфигурирование WordPress;
- настройка правил iptables.<br/>

**mastersdb**<br/>
&ensp;&ensp;Настройка главного сервера БД MySQL:
- установка и конфигурирование СУБД MySQL;
- развёртывание БД WordPress;
- настройка правил iptables.<br/>

**slavesdb**<br/>
&ensp;&ensp;Настройка сервера репликации БД MySQL;
- установка и конфигурирование СУБД MySQL;
- развёртывание БД WordPress;
- настройка и запуск репликации;
- настройка инфраструктуры для резервного копирования БД;
- настройка сервиса Cron;
- настройка правил iptables.<br/>

**monitoring**<br/>
&ensp;&ensp;Настройка сервера мониторинга и логирования:
- установка и конфигурирование ПО GAP-стека;
- установка и конфигурирование ПО ELK-стека;
- настройка правил iptables.<br/>

&ensp;&ensp;В проекте для выполнения только определённых ролей используются ansible-тэги. Названия тэгов соответствуют названиям ролей. Типовой пример использования тэгов для конфигурирования мастер-сервера БД MySQL:
```shell
ansible-playbook --tags=preinstall,mastersdb provision.yml
```
## Проверка работы стенда
&ensp;&ensp;В результате запуска всех виртуальных хостов, составляющих инфраструктуру веб-сервиса, и отработки всех задач плэйбука Ansible, получены: работающий на двух бэкенд серверах Apache CMS WordPress, использующий для своей работы СУБД MySQL; фронтенд сервер NGINX, осуществляющий балансировку нагрузки; сервер мониторинга (стек GAP) и логирования (стек ELK).  
&ensp;&ensp;Для проверки работы балансировки необходимо перейти на сайт по внешнему адресу фронтенд сервера. На главной странице, среди прочего, будет отображаться имя первого бэкенд сервера. После обновления главной страницы отобразится имя второго бэкенд сервера.
![изображение](https://github.com/user-attachments/assets/624b3c5a-e288-4200-b2c6-9da2d1ec81d8)

![изображение](https://github.com/user-attachments/assets/a2375720-224d-435a-ae05-f140d917b5f3)



