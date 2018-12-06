# Zabbix monitoring of Bareos's backup jobs and its processes
Мониторинг заданий Bareos производится посредством bash скриптов и perl скриптов. Perl скрипты используются в шаблонах Zabbix для низкоуровнего определия клиентов и списка заданий. мониторинг производится по каждому клиенту в разрезе каждого задания по уровням архивирования. Bash скрипты собирают информацию о выполненном задании из Bareos Catalog и передают в Zabbix. Создавалось и тестировалось на Bareos 17.2.4, Zabbix 4.0.1, Postgresql 10.6, в Centos 7.5 selinux=disable Linux.

##Основные возможности
- Низкоуровневое определение клиентов Bareos в Zabbix;
- низкоуровневое определения заданий по клиентам Bareos в Zabbix;
- раздельный учет каждого клиента Bareos в Zabbix;
- раздельный учет каждого задания архивирования Bareos в Zabbix;
- раздельный учет каждого уровня архивирования Bareos в Zabbix.

##Особенности
###Собираемые данные
- статус завершения задания;
- объем каждого задания;
- количество файлов в задании;
- время выполения задания;
- скорость выполнения задания.

##Состав

###Шаблоны Zabbix
- *Template Bareos Сlients* - шаблон для определения списка клиентов Bareos и создания хостов на Zabbix;
- *Template Bareos Processes* - шаблон для мониторинга состояния процессов Bareos;
- *Template Bareos* - шаблон для определения списка заданий по клиентам Bareos и создание элеменов в Zabbix хостах.

###Скрипты
- *bareos-zabbix.bash* - сбор данных из задания и отправка в Zabbix
- *bareos.pl* - скрипт для пользовательских параметров Zabbix агента, для создания элементов заданий в хостах Zabbix
- *bareos_hosts.pl* -скрипт для пользовательских параметров Zabbix агента, для создания хостов Zabbix


##Установка

1. Создать файлonf
    chmod 640 /etc/bareos/bareos-zabbix.conf `/etc/bareos/bareos-zabbix.conf`,как в примере репозитория, отредактировать в соответствии со своими реалиями и установить права:
  ```
    chown root:bareos /etc/bareos/bareos-zabbix.conf
  ```

2. Создать файл `/var/spool/bareos/bareos-zabbix.bash` копирование его из репозитория и установить права:
  ```
  chown bareos:bareos /var/spool/bareos/bareos-zabbix.bash
  chmod 700 /var/spool/bareos/bareos-zabbix.bash
  ```
3. Создать файл Zabbix.conf в каталоге messages  `/etc/bareos/messages/Zabbix.conf`, следующее:

```
chown bareos:bareos /etc/bareos/messages/Zabbix.conf
```
таким образом после каждого задания будет вызываться bash скрипт в котроый будет передваться ID задания.
не забудьте активировать скрипт для Job'ы

4. Скопировать из репозитория файлы `bareos.pl` и `bareos_hosts.pl` в папку со скриптами для агента Zabbix, В моем случае это: `/etc/zabbix/scripts/`, и установить права:

  ```
  chown zabbix:zabbix /etc/zabbix/scripts/bareos.pl
  chown zabbix:zabbix /etc/zabbix/scripts/bareos_hosts.pl
  chmod 700 /etc/zabbix/scripts/bareos.pl
  chmod 700 /etc/zabbix/scripts/bareos_hosts.pl  
  ```
  
5. Отредактировать конфигурационный файл Zabbix агента `/etc/zabbix/zabbix-agentd.conf`, добавив следующее:
  
  ```
  UserParameter=bareos.jobs[*],/usr/bin/perl /etc/zabbix/scripts/bareos.pl $1                                       
  UserParameter=bareos.hosts,/usr/bin/perl /etc/zabbix/scripts/bareos_hosts.pl
  ```
  
6. В домашний каталог для пользователей zabbix и bareos положить файл .pgpass  следующего содержания:
```
127.0.0.1:*:*:bareos:yourpasswordhere
```
Выставить права 0600
7. Перезаргузить настройки Bareos Director, для этого сделать следующее:

  ```
   systemctl restart bareos-dir 
  ```
  или 
  ```
    bconsole
    reload
    exit
  ```
8. Перезапустить zabbix-agent:

  ```
  systemctl restart zabbix-agent
  ```
9. Импортировать в Zabbix шаблоны `bareos-template.xml`.
10. В Zabbix подключить шаблон *Template Bareos Сlients* к хосту для которого правился конфиг агента.
11. В Zabbix подключить шаблон *Template Bareos Processes* к хоста для которых нужно мониторить сотояние процессов.

## Ссылки

- **Bareos**:
  - http://doc.bareos.org/master/html/bareos-manual-main-reference.html
- **Zabbix**:
  - https://www.zabbix.com/documentation/4.0/start


## Feedback

Вопросы, замечания и предложения:

- https://github.com/ssv1982/bacula-zabbix/issues

## P.S.
Создано на основе:
    https://github.com/ssv1982/bareos-zabbix
