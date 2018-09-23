# m103

# Clustering MongoDB

## Notes

### Chapter 1

db.createUser()
db.dropUser()
db.createCollection()
db.runCommand()
db.newCollection("collection")

#### Indexes
Improve query performance at the expense of memory and must be updated.
The index for _Id field is always created but others must be created manually.

#### Documents are limited to 16MB.

MongoDB JSON is extended to support all BSON data types.

##### Configuration File

https://docs.mongodb.com/v3.6/reference/configuration-options/

Lab - Launching MongoDB

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

$ mongo --port 27000 --authenticationDatabase admin -u m103-admin -p m103-pass
