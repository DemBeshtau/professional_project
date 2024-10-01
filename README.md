# Автоматизация развёртывания инфраструктуры веб-сервиса с использованием оркестратора Ansible и реализация плана её аварийного восстановления.
### Задание
  Развернуть веб-проект отвечающий следующим требованиям:
  - включен HTTPS;
  - основная инфраструктура находится в DMZ;
  - настроен firewall;
  - настроен сбор логов и мониторинг с уведомлениями;
  - организован бэкап.<br/>
  Серверная инфраструктура представлена шестью серверами:
  - сервер балансировки нагрузки NGINX (hostname: **frontweb**, IP: 192.168.1.145, 192.168.56.145);
	- первый бэкенд-сервер Apache (hostname: **backend1**, IP: 192.168.1.222);
	- второй бэкенд-сервер Apache (hostname: **backend2**, IP: 192.168.1.212);
	- основной сервер базы данных (БД) MySQL (hostname: **mastersdb**, IP: 192.168.1.69);
	- cервер репликации БД MySQL (hostname: **slavesdb**, IP: 192.168.1.98);
	- cервер мониторинга и логирования (hostname: **monitoring**, IP: 192.168.1.60).
   
