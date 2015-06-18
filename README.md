# Zabbix monitoring of Bareos's backup jobs and its processes
Мониторинг заданий Bareos производится посредством bash скриптов и perl скриптов. Perl скрипты используются в шаблонах Zabbix для низкоуровнего определия клиентов и списка заданий. мониторинг производится по каждому клиенту в разрезе каждого задания по уровням архивирования. Bash скрипты собирают информацию о выполненном задании из Bareos Catalog и передают в Zabbix. Создавалось и тестировалось на Bareos 15.2.0, Zabbix 2.4.5, Postgresql 9.4.3, в Calculate Linux.

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
3. Оредактировать секцию `Messages` в `/etc/bareos/bareos-dir.conf`, добавив следующее:

  ```
  Messages {
    ...
    mailcommand = "/var/spool/bareos/bareos-zabbix.bash %i"
    mail = 127.0.0.1 = all, !skipped
    ...
  }
  ```
таким образом после каждого задания будет вызываться bash скрипт в котроый будет передваться ID задания.

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
6. Перезаргузить настройки Bareos Director, для этого сделать следующее:

  ```
    /etc/init.d/bacula-dir restart
  ```
  или 
  ```
    bconsole
    reload
    exit
  ```
7. Перезапустить zabbix-agent:

  ```
  /etc/init.d/zabbix-agentd restart
  ```
8. Импортировать в Zabbix шаблоны `bareos-template.xml`.
9. В Zabbix подключить шаблон *Template Bareos Сlients* к хосту для которого правился конфиг агента.
10. В Zabbix подключить шаблон *Template Bareos Processes* к хоста для которых нужно мониторить сотояние процессов.

## Ссылки

- **Bareos**:
  - http://doc.bareos.org/master/html/bareos-manual-main-reference.html
- **Zabbix**:
  - https://www.zabbix.com/documentation/2.4/start


## Feedback

Вопросы, замечания и предложения:

- https://github.com/ssv1982/bacula-zabbix/issues

## P.S.
Создано на основе:
    https://github.com/germanodlf/bacula-zabbix.git
