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

###Шаблоны Zabbix
- *Template Bareos Сlients* - шаблон для определения списка клиентов Bareos и создания хостов на Zabbix;
- *Template Bareos Processes* - шаблон для мониторинга состояния процессов Bareos;
- *Template Bareos* - шаблон для определения списка заданий по клиентам Bareos и создание элеменов в Zabbix хостах.

##Установка

1. Создать файлonf
    chmod 640 /etc/bareos/bareos-zabbix.conf `/etc/bareos/bareos-zabbix.conf`,как в примере репозитория, отредактировать в соответствии со своими реалиями и установить права:
  ```
    chown root:bareos /etc/bareos/bareos-zabbix.c
  ```

2. Создать файл `/var/spool/bareos/bareos-zabbix.bash` копирование его из репозитория и установить права:
  ```
  chown bareos:bareos /var/spool/bareos/bareos-zabbix.bash
  chmod 700 /var/spool/bareos/bareos-zabbix.bash
  ```
3. Скопировать из репозитория файлы `bareos.pl` и `bareos_hosts.pl` в папку со скриптами для агента Zabbix
