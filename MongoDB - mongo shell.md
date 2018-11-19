Mongo (the CLI javascript client) tips
======================================

Essential links
---------------

https://docs.mongodb.com/manual/reference/mongo-shell/

https://docs.mongodb.com/manual/reference/method/

:point_up_2: You really should at leask look at those links


How to install MongoDB
----------------------

macOS: `brew install mongodb` or `port install mongodb`

Linux: [Install MongoDB Community Edition on Linux](https://docs.mongodb.com/manual/administration/install-on-linux/)

Docker: [mongo](https://hub.docker.com/_/mongo/)


How to run mongo shell
----------------------

```shell
$ mongo
```


Basic stuff
-----------

List all databases:

```
show dbs
```


Switch to database `foo`:

```
use foo
```


List all colections in the database you are currently in:

```
show collections
```

List some documents in collection `products`:

```
db.products.find()
db.products.find({ price: { $gte: 123 } })
db.products.find({ price: { $gte: 123 } }).pretty()
db.products.find({ price: { $gte: 123 } }).limit(5)
```

You can convert the results to JS Array:

```
db.products.find().toArray().filter(doc => doc.stock < doc.min_stock)
```

Tips
----

When you create an index, do it in the background to not disrupt other clients:

```
db.persons.createIndex({ name: 1 }, { background: true })
```

ObjectId
--------

ObjectId is a 12-byte value that consists of: 4-byte timestamp (seconds since Unix epoch), 5-byte random value and 3-byte counter (starting with a random value).

[Before mid-2018](https://github.com/mongodb/docs/commit/17d93fd2e3347d307d91e1fc9cdc0b3887d1d3fe) it was 3-byte machine identifier and 2-byte process id instead of the 5-byte random value.

How to create ObjectId object:

```
> ObjectId()
ObjectId("5bf2d543a324d9e1a71aee53")

> ObjectId('123456789012345678901234')
ObjectId("123456789012345678901234")

> ObjectId.fromDate(new Date('2018-11-12T00:00:00Z'))
ObjectId("5be8c2800000000000000000")
```

Attributes of the ObjectId object:

```
> ObjectId("507f191e810c19729de860ea").str
507f191e810c19729de860ea

> ObjectId("507f191e810c19729de860ea").valueOf()
507f191e810c19729de860ea

> ObjectId("507f191e810c19729de860ea").getTimestamp()
ISODate("2012-10-17T20:46:22Z")
```


Replica sets
------------

Show status of replica set:

```
rs.status()
```

When you find yourself connected to a secondary but you still want to perform read operations:

```
rs.slaveOk()
```

[Replica Set Deployment Tutorials](https://docs.mongodb.com/manual/administration/replica-set-deployment/)


.mongorc.js
-----------


Tweaking
--------

To increase the number of results that you get back from the query (the default is 20):

```
DBQuery.shellBatchSize = 100; 
```


Links
-----

https://testlio.com/blog/handy-tips-for-mongodb-shell-queries/

http://mo.github.io/2017/01/22/mongo-db-tips-and-tricks.html




