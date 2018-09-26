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

Security

Authentication: Who are you? 
Authorization: What can you do?

4 client auth methods:
  - scram (default) basic password security
  - x509 cert
  - ldap
  - kerberos

cluster to cluster 

RBAC
  - each user has 1 or more roles
  - each role has 1 or more privs
Users are associated with a DB.
Localhost exception
Create a super user first, then auth as that user.

use admin
db.createUser{user: root, pwd: root, roles: {root}}
See lecture notes.
```
use admin
db.createUser({
  user: "root",
  pwd: "root123",
  roles : [ "root" ]
})

mongo --username root --password root123 --authenticationDatabase admin

db.stats()
```
Built-in roles
Set of privs (actions over a resource)
Network restrictions.

Roles
read, readWrite
dbAdmin, userAdmin, dbOwner
clusterAdmin, clusterManager, clusterMonitor, hostManager
backup, restore
root
readAnyDatabase, readWriteAnyDatabase
dbAdminAnyDatabase, userAdminAnyDatabase
root

Lab - Create dbUser user
```
use admin
db.createUser({
  user: "m103-application-user",
  pwd: "m103-application-pass",
  roles : [ { db: "applicationData",
role: "readWrite" } ]
})

Successfully added user: {
	"user" : "m103-application-user",
	"roles" : [
		{
			"db" : "applicationData",
			"role" : "readWrite"
		}
	]
}
```

Server Tools

mongostat --port 27000
mongorestore, mongodump (bson utils)

mongorestore <options> dump/

mongoexport/import (json-based)

Lab - Import 
$ mongoimport --host=127.0.0.1:27000 --authenticationDatabase=admin --db=applicationData --collection=products --username=m103-application-user --password=m103-application-pass /shared/products.json```

2018-09-25T03:15:52.700+0000	imported 516784 documents
```

#### Replica Sets

##### Config options

Use openssl to create a security.keyFile: (used to auth to each other)
chmod 400 keyfile
replication.replSetName: set-name

Each node requires a unique:
 - dbpath
 - logpath
 - port

Lab - Replica Set

Config file for node-1
```
net:
  port: 27001
  bindIp: "192.168.103.100,localhost"
security:
  authorization: enabled
  keyFile: /var/mongodb/pki/m103-keyfile
storage:
  dbPath: /var/mongodb/db/1
systemLog:
  destination: file
  path: /var/mongodb/db/mongod1.log
  logAppend: true
processManagement:
  fork: true
operationProfiling:
  mode: slowOp
  slowOpThresholdMs: 50
replication:
  replSetName: m103-repl
```

Add the replicas.
```
mongo 192.168.103.100:27001/admin -u m103-admin -p m103-pass
MongoDB Enterprise m103-repl:PRIMARY> rs.add("192.168.103.100:27002")
```