Mongo (the CLI javascript client) tips
======================================

Essential links
---------------

https://docs.mongodb.com/manual/reference/mongo-shell/


How to run mongo shell
----------------------




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


Tweaking
--------

To increase the number of results that you get back from the query (the default is 20):

```
DBQuery.shellBatchSize = 100; 
```


Links
-----

https://testlio.com/blog/handy-tips-for-mongodb-shell-queries/



