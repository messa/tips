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




