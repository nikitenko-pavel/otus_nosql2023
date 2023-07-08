> /*
> mongo yandex cloud
> 1GB RAM 20% ssd 5Gb
> */

#### Task list
- [x] set mongo (1)
- [x] set configsvr (3)
- [x] initiate
- [x] init replicaset 3х3
- [x] create shard 27000, 27100
- [x] start shard on DB
- [x] load DB person + set shard Sex
- [x] STOP service
- [x] CREATE USER


# set mongo
`wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add - \
&& echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list \
&& apt update \
&& apt-get install -y mongodb-org`

# set configsvr
###### 3 конфига сервера
`sudo mkdir /home/mongo && sudo mkdir /home/mongo/{dbc1,dbc2,dbc3} && sudo chmod 777 /home/mongo/{dbc1,dbc2,dbc3}`

`mongod --configsvr --dbpath /home/mongo/dbc1 --port 27001 --replSet RScfg --fork --logpath /home/mongo/dbc1/dbc1.log --pidfilepath /home/mongo/dbc1/dbc1.pid`

`mongod --configsvr --dbpath /home/mongo/dbc2 --port 27002 --replSet RScfg --fork --logpath /home/mongo/dbc2/dbc2.log --pidfilepath /home/mongo/dbc2/dbc2.pid`

`mongod --configsvr --dbpath /home/mongo/dbc3 --port 27003 --replSet RScfg --fork --logpath /home/mongo/dbc3/dbc3.log --pidfilepath /home/mongo/dbc3/dbc3.pid`

# initiate
`ps aux | grep mongo| grep -Ev "grep" `
`mongosh --port 27001`

`rs.initiate({"_id" : "RScfg", configsvr: true, members : [{"_id" : 0, priority : 3, host : "127.0.0.1:27001"},{"_id" : 1, host : "127.0.0.1:27002"},{"_id" : 2, host : "127.0.0.1:27003"}]});
`
> 	--all SECONDARY
	--WAIT
	--'127.0.0.1:27001' PRIMARY


#  init replicaset 3х3
###### Создаем репликасеты 3х3
`sudo sudo mkdir /home/mongo/{db1,db2,db3,db4,db5,db6,db7,db8,db9} && sudo chmod 777 /home/mongo/{db1,db2,db3,db4,db5,db6,db7,db8,db9}`

--1
`mongod --shardsvr --dbpath /home/mongo/db1 --port 27011 --replSet RS1 --fork --logpath /home/mongo/db1/db1.log --pidfilepath /home/mongo/db1/db1.pid
mongod --shardsvr --dbpath /home/mongo/db2 --port 27012 --replSet RS1 --fork --logpath /home/mongo/db2/db2.log --pidfilepath /home/mongo/db2/db2.pid
mongod --shardsvr --dbpath /home/mongo/db3 --port 27013 --replSet RS1 --fork --logpath /home/mongo/db3/db3.log --pidfilepath /home/mongo/db3/db3.pid`
--2
`mongod --shardsvr --dbpath /home/mongo/db4 --port 27021 --replSet RS2 --fork --logpath /home/mongo/db4/db4.log --pidfilepath /home/mongo/db4/db4.pid
mongod --shardsvr --dbpath /home/mongo/db5 --port 27022 --replSet RS2 --fork --logpath /home/mongo/db5/db5.log --pidfilepath /home/mongo/db5/db5.pid
mongod --shardsvr --dbpath /home/mongo/db6 --port 27023 --replSet RS2 --fork --logpath /home/mongo/db6/db6.log --pidfilepath /home/mongo/db6/db6.pid`
--3
`mongod --shardsvr --dbpath /home/mongo/db7 --port 27031 --replSet RS3 --fork --logpath /home/mongo/db7/db7.log --pidfilepath /home/mongo/db7/db7.pid
mongod --shardsvr --dbpath /home/mongo/db8 --port 27032 --replSet RS3 --fork --logpath /home/mongo/db8/db8.log --pidfilepath /home/mongo/db8/db8.pid
mongod --shardsvr --dbpath /home/mongo/db9 --port 27033 --replSet RS3 --fork --logpath /home/mongo/db9/db9.log --pidfilepath /home/mongo/db9/db9.pid`

--1
`mongosh --port 27011
rs.initiate({"_id" : "RS1", members : [{"_id" : 0, priority : 3, host : "127.0.0.1:27011"},{"_id" : 1, host : "127.0.0.1:27012"},{"_id" : 2, host : "127.0.0.1:27013", arbiterOnly : true}]});`

--2
`mongosh --port 27021
rs.initiate({"_id" : "RS2", members : [{"_id" : 0, priority : 3, host : "127.0.0.1:27021"},{"_id" : 1, host : "127.0.0.1:27022"},{"_id" : 2, host : "127.0.0.1:27023", arbiterOnly : true}]});`

--3
`mongosh --port 27031
rs.initiate({"_id" : "RS3", members : [{"_id" : 0, priority : 3, host : "127.0.0.1:27031"},{"_id" : 1, host : "127.0.0.1:27032"},{"_id" : 2, host : "127.0.0.1:27033", arbiterOnly : true}]});`



# create shard 27000, 27100
###### Связываем шарды
mongos --configdb RScfg/127.0.0.1:27001,127.0.0.1:27002,127.0.0.1:27003 --port 27000 --fork --logpath /home/mongo/dbc1/dbs.log --pidfilepath /home/mongo/dbc1/dbs.pid 
mongos --configdb RScfg/127.0.0.1:27001,127.0.0.1:27002,127.0.0.1:27003 --port 27100 --fork --logpath /home/mongo/dbc1/dbs2.log --pidfilepath /home/mongo/dbc1/dbs2.pid 

mongosh --port 27000
sh.addShard("RS1/127.0.0.1:27011,127.0.0.1:27012,127.0.0.1:27013")
--ERROR: MongoServerError: Cannot add RS1/127.0.0.1:27011,127.0.0.1:27012,127.0.0.1:27013 as a shard since the implicit default write concern on this shard is set to {w : 1}, because number of arbiters in the shard's configuration caused the number of writable voting members not to be strictly more than the voting majority. Change the shard configuration or set the cluster-wide write concern using the setDefaultRWConcern command and try again.

db.adminCommand({ "setDefaultRWConcern": 1, "defaultWriteConcern": { "w": 1 }, "defaultReadConcern": { "level": "majority" } })

sh.addShard("RS1/127.0.0.1:27011,127.0.0.1:27012,127.0.0.1:27013")
sh.addShard("RS2/127.0.0.1:27021,127.0.0.1:27022,127.0.0.1:27023")
sh.addShard("RS3/127.0.0.1:27031,127.0.0.1:27032,127.0.0.1:27033") 

sh.status()
--result:
shards
[
  {
    _id: 'RS1',
    host: 'RS1/127.0.0.1:27011,127.0.0.1:27012',
    state: 1,
    topologyTime: Timestamp({ t: 1688848549, i: 1 })
  },
  {
    _id: 'RS2',
    host: 'RS2/127.0.0.1:27021,127.0.0.1:27022',
    state: 1,
    topologyTime: Timestamp({ t: 1688848572, i: 1 })
  },
  {
    _id: 'RS3',
    host: 'RS3/127.0.0.1:27031,127.0.0.1:27032',
    state: 1,
    topologyTime: Timestamp({ t: 1688848788, i: 1 })
  }
]

# start shard on DB
`use titanic
sh.enableSharding("titanic") `
###### Загрузили 3 типа строк. М(30000 строк)/Ж(10000)/Нет пола (50тыс)
`--NO Sex
for (var i=0; i<50000; i++) { db.person.insert({name: "Name", Age: Math.random()}) }
--Sex =  female 10000
for (var i=0; i<10000; i++) { db.person.insert({name: "Name FeMale", Age: Math.random(), Sex :"female"}) }
--Sex =  male 30000
for (var i=0; i<30000; i++) { db.person.insert({name: "Name Male", Age: Math.random(), Sex :"male"}) }
`

###### CHECK db
`connect -27000
db.person.count()`
--80000
`db.person.createIndex({Sex: 1})
db.person.stats()`
--sharded -false

use admin
`db.runCommand({shardCollection: "titanic.person", key: {Sex: 1}})`
use titanic
`db.person.stats()`
--sharded -true

sh.status()
###### Разбиты на 3 чанки, на трех шардах. каждый тип на своем шарде
--RS1 nChunk 1
--RS2 nChunk 1
--RS3 nChunk 1
 chunks: [
          { min: { Sex: MinKey() }, max: { Sex: 'female' }, 'on shard': 'RS3', 'last modified': Timestamp({ t: 3, i: 1 }), jumbo: 'yes' },
          { min: { Sex: 'female' }, max: { Sex: 'male' }, 'on shard': 'RS1', 'last modified': Timestamp({ t: 2, i: 0 }) },
          { min: { Sex: 'male' }, max: { Sex: MaxKey() }, 'on shard': 'RS2', 'last modified': Timestamp({ t: 3, i: 0 }) }
        ],

`sh.balancerCollectionStatus("titanic.person")`

`sh.status()`
--RS1 nChunk 1
--RS2 nChunk 1
--RS3 nChunk 1

# STOP service
--27011
use admin
db.ShutdownServer()
###### Погасили сервер 27011
--27000
use titanic 
db.person.count() 
###### Кол-во строк выросло??
-- 110000

--27012
use admin
db.ShutdownServer()
###### При попытке погасить 27012 получаю TypeError: db.ShutdownServer is not a function


# CREATE USER
only read
db.createUser( { user: "read1", pwd: "123", roles: [{ role: "write", db: "titanic" }] })
write+read
db.createUser( { user: "write1", pwd: "123", roles: [{ role: "readWrite", db: "titanic" }] })

mongo titanic --host 127.0.0.1:27100 -u read1 -p 123 --authenticationDatabase admin
###### Ошибка отсутсвия модуля error install mongodb-clients

apt install mongodb-clients
mongo titanic --host 127.0.0.1:27100 -u read1 -p 123 --authenticationDatabase admin
connect MONGOS
###### вошли в MONGOS через пароль





