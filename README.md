# m103

# Clustering MongoDB

## Notes

### Chapter 1

```
db.createUser()
db.dropUser()
db.createCollection()
db.runCommand()
db.newCollection("collection")
```
#### Indexes

Improve query performance at the expense of memory and must be updated.
The index for _Id field is always created but others must be created manually.

#### Documents are limited to 16MB.

MongoDB JSON is extended to support all BSON data types.

##### Configuration File

https://docs.mongodb.com/v3.6/reference/configuration-options/

Lab - Launching MongoDB
```
$ mongod --port 27000 --dbpath /data/db --bind_ip 192.168.103.100,localhost --auth --smallfiles --logpath logs/mongod.log --fork

mongo admin --host localhost:27000 --eval '
  db.createUser({
    user: "m103-admin",
    pwd: "m103-pass",
    roles: [
      {role: "root", db: "admin"}
    ]
  })
'

$ mongo localhost:27000/admin -u m103-admin -p m103-pass
MongoDB shell version v3.6.8
connecting to: mongodb://localhost:27000/admin
MongoDB server version: 3.6.8
Server has startup warnings: 
2018-09-23T19:59:54.199+0000 I STORAGE  [initandlisten] 
2018-09-23T19:59:54.199+0000 I STORAGE  [initandlisten] ** WARNING: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine
2018-09-23T19:59:54.199+0000 I STORAGE  [initandlisten] **          See http://dochub.mongodb.org/core/prodnotes-filesystem
MongoDB Enterprise > show dbs
admin   0.000GB
config  0.000GB
local   0.000GB

MongoDB Enterprise > db.getUsers()
[
	{
		"_id" : "admin.m103-admin",
		"user" : "m103-admin",
		"db" : "admin",
		"roles" : [
			{
				"role" : "root",
				"db" : "admin"
			}
		]
	}
]
```
Lab - Configuration File

Example
```
$ mongod --port 27000 --dbpath /data/db --bind_ip 192.168.103.100,localhost --auth --smallfiles --logpath logs/mongod.log --fork

net:
  port: 27000
  bindIp: "192.168.103.100,localhost"
systemLog:
  path: logs/mongod.log
processManagement:
  fork: True
security:
  authorization: enabled
storage:
  dbPath: /data/db
```
Lab Assignment
```
$ mongod --port 27000 --dbpath /data/db --bind_ip 192.168.103.100,localhost --auth 
```
net:
  port: 27000
  bindIp: "192.168.103.100,localhost"
security:
  authorization: enabled
storage:
  dbPath: /data/db
```

Lab - Logging to a Different Facility
Need to set db.setProfilingLevel( 1, { slowms: 50 } )

```
net:
  port: 27000
  bindIp: "192.168.103.100,localhost"
security:
  authorization: enabled
storage:
  dbPath: /data/db
systemLog:
  destination: file
  path: /var/mongodb/db/mongod.log
  logAppend: true
processManagement:
  fork: true
operationProfiling:
  mode: slowOp
  slowOpThresholdMs: 50

$ mongo localhost:27000/admin -u m103-admin -p m103-pass
```

