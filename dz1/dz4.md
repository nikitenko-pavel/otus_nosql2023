> /*
> yandex cloud
> 1 x 5GB RAM 20% hdd 15Gb
> 3 x 4GB RAM 20% hdd 15Gb
> */

#### Task list
- [x] Устанавливаем Couchbase на 4 нодах
- [x] открываем UI. Кейс 1. Берем ноду с 5GB и создаем там кластер. и пытаемся добавить ноду с меньшим размером памяти. 
- [x] открываем UI. Создаем кластер на ноде, где минимальные ресурсы.
- [x] добавляем Sample Buckets 
- [x] sql запросы
- [x] отказоусточивость
- [x] заметки


# set Couchbase x 4

sudo apt-get update && sudo apt-get upgrade -y && curl -O https://packages.couchbase.com/releases/couchbase-release/couchbase-release-1.0-amd64.deb && sudo dpkg -i ./couchbase-release-1.0-amd64.deb && sudo apt-get update && sudo apt-get install couchbase-server -y

# Кейс 1. Берем ноду с 5GB и создаем там кластер. и пытаемся добавить ноду с меньшим размером памяти. 

1 x 5GB RAM 20% hdd 15Gb
https://84.201.158.227:18091/

Указываем максимум памяти для этой ноды.

Attention: Prepare join failed. This server does not have sufficient memory to support requested memory quota. Total quota is 3155MB (services: fts, index, kv, n1ql), maximum allowed quota for the node is 3135MB.

# открываем UI. Создаем кластер на ноде, где минимальные ресурсы.

на минимальное ноде создали кластер. и в нее добавили другие ноды.
Запустили балансировку.


# добавляем Sample Buckets 
в UI Buckets добавить Sample Buckets. beer-sample

# sql запросы
select * from `beer-sample`.`_default`.`_default` data order by meta().id limit 10
select * from `beer-sample`.`_default`.`_default` data where type='brewery' limit 10

# отказоусточивость

1.  отключение сервера
- отключаем Auto-failover
- отключаем сервер
- видим что сервер не доступен в UI. Пометка о необходимости FAILOVER. запросы не работают
- делаем FAILOVER. 
- запросы заработали. ребалансировка убирает ноду из списка.
 
2.  Подключение сервера со старой БД к новым данным
- удаляем баккет. создаем другую БД
- добавляем обратно сервер со старой БД из п1.2. 
- Успешно добавлется и балансируется


# заметки
Простая настройка.
Балансировка. 

Ощущение что балансировка, приводит к ПОЛНОМУ движению данных на всех нодах, а не частичное для "старых" нод
