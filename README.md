
# Домашнее задание к занятию "Система мониторинга Zabbix часть 2" - Бобков Александр

## Задание 1
Установите Zabbix Server с веб-интерфейсом.

### Процесс выполнения
1. Выполняя ДЗ, сверяйтесь с процессом отражённым в записи лекции.
2. Установите PostgreSQL. Для установки достаточна та версия, что есть в системном репозитороии Debian 11.
3. Пользуясь конфигуратором команд с официального сайта, составьте набор команд для установки последней версии Zabbix с поддержкой PostgreSQL и Apache.
4. Выполните все необходимые команды для установки Zabbix Server и Zabbix Web Server.

### Требования к результатам
* Прикрепите в файл README.md скриншот авторизации в админке.
* Приложите в файл README.md текст использованных команд в GitHub.

## ОТВЕТ:
<details>
<summary>Нажми, чтобы увидеть скриншот установки</summary>
<img src="img/1.jpg" width = 100%>
<img src="img/2.jpg" width = 100%>
</details>

* **Используемые команды:**

**а. Установите репозиторий Zabbix:**
```bash
wget https://repo.zabbix.com
dpkg -i zabbix-release_latest_7.4+debian12_all.deb
apt update 
```

**б. Установите Zabbix сервер, веб-интерфейс и агент**
```bash
apt install zabbix-server-pgsql zabbix-frontend-php php8.2-pgsql zabbix-apache-conf zabbix-sql-scripts zabbix-agent
```

**в. Установка postgresql**
```bash
sudo apt install postgresql postgresql-contrib -y
```

**г. Создайте базу данных**

* **Установите и запустите сервер базы данных. Выполните следующие комманды на хосте, где будет распологаться база данных.**
```bash
sudo -u postgres createuser --pwprompt zabbix
sudo -u postgres createdb -O zabbix zabbix
```
* **На хосте Zabbix сервера импортируйте начальную схему и данные. Вам будет предложено ввести недавно созданный пароль.**
```bash  
zcat /usr/share/zabbix/sql-scripts/postgresql/server.sql.gz | sudo -u zabbix psql zabbix 
```

**д. Настройте базу данных для Zabbix сервера**

* **Отредактируйте файл /etc/zabbix/zabbix_server.conf**

DBPassword=password  #(ввести свой пароль)


 
**e. Запустите процессы Zabbix сервера и агента**

* **Запустите процессы Zabbix сервера и агента и настройте их запуск при загрузке ОС.**
```bash
systemctl restart zabbix-server zabbix-agent apache2
systemctl enable zabbix-server zabbix-agent apache2 
```

***Открыть страницу с zabbix http://host/zabbix*** 

**################################################################################################**


# Задание 2

### Установите Zabbix Agent на два хоста.
Процесс выполнения

    Выполняя ДЗ, сверяйтесь с процессом отражённым в записи лекции.
    Установите Zabbix Agent на 2 вирт.машины, одной из них может быть ваш Zabbix Server.
    Добавьте Zabbix Server в список разрешенных серверов ваших Zabbix Agentов.
    Добавьте Zabbix Agentов в раздел Configuration > Hosts вашего Zabbix Servera.
    Проверьте, что в разделе Latest Data начали появляться данные с добавленных агентов.

### Требования к результатам

    Приложите в файл README.md скриншот раздела Configuration > Hosts, где видно, что агенты подключены к серверу
    Приложите в файл README.md скриншот лога zabbix agent, где видно, что он работает с сервером
    Приложите в файл README.md скриншот раздела Monitoring > Latest data для обоих хостов, где видны поступающие от агентов данные.
    Приложите в файл README.md текст использованных команд в GitHub


## ОТВЕТ:

**Cкриншот раздела Configuration > Hosts:**

<img src="img/_3.jpg" width= 100%>


**Cкриншот лога zabbix agent:**	

<img src="img/5.jpg" width="100%">
   
**Cкриншот раздела Monitoring > Latest data:**

<img src="img/_4.jpg" width="100%">
 
**Покажу еще панели (не стал менять ip для агента установленного на самом сервере zabbix):**

<img src="img/4.jpg" width="100%">


**Текст использованных команд**

1. **Устанавливаем сам Агент:**

```bash
apt update && apt install zabbix-agent -y
```
2. **Правим конфигурационный файл:** `nano /etc/zabbix/zabbix_agentd.conf`

```ini
ServerActive=10.129.0.5 — IP сервера (куда агент сам шлет данные);
Hostname=zabbixclient — это имя должно точно совпадать с полем "Host name" в веб-интерфейсе Zabbix.
Server=10.129.0.5 — для пассивных проверок (чтобы сервер мог опрашивать агент).
```
3. **Произодим рестарт сервиса Агента:**

```bash
systemctl restart zabbix-agent
```

4. **Добавляем сервис в автозагрузку:**

```bash
systemctl enable zabbix-agent
```

##### Столкнулся с проблемой, т.к. виртульная машина развернута в YandexCloud, на сервере Zabbix были включены правила iptables, из за чего не проходили пакеты на порт 10051, на нем Zabbix слушает информацию от Агентов, это нужно в томслучае, если Агент отсылает информаицю в Zabbix, а не только Zabbix запрашивает информацию от Агента. Пришлось изменить правила iptables.


