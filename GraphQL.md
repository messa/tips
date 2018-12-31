GraphQL
=======

Fragments
---------

https://www.apollographql.com/docs/react/advanced/fragments.html

A GraphQL fragment is a shared piece of query logic.

- Sharing fields between multiple queries, mutations or subscriptions.
- Breaking your queries up to allow you to co-locate field access with the places they are used.

```graphql
fragment NameParts on Person {
  firstName
  lastName
}

query GetPerson {
  people(id: "7") {
    ...NameParts
    avatar(size: LARGE)
  }
}
```

Simple server
-------------

Schema:

```javascript
import { 
  GraphQLSchema, GraphQLObjectType, GraphQLInterfaceType, GraphQLEnumType,
  GraphQLList, GraphQLNonNull, GraphQLString, GraphQLID
} from 'graphql'

const schema = new GraphQLSchema({
  query: new GraphQLObjectType({
    name: 'Query',
    fields: {
      user: {
        type: UserType,
        args: {
          id: { type: GraphQLID }
        },
        resolve: (root, args, context, info) => {
          const { id } = args      // the `id` argument for this field is declared above
          return fetchUserById(id) // hit the database
        }
      }
    }
  })
})
```

Server:

```javascript
const express = require('express')
const graphqlHTTP = require('express-graphql')

const app = express()

app.use('/graphql', graphqlHTTP({
  schema: schema,
  graphiql: true
}))

app.listen(4000)
```

Alternatively, options can also be provided as a function (or async function) which returns this options object:

```javascript
app.use('/graphql', graphqlHTTP(async (request, response, graphQLParams) => ({
  schema: schema,
  rootValue: await someFunctionToGetRootValue(request),
  graphiql: true
})))
```

Dependencies:

```shell
$ npm install --save graphql express-graphql
```

You can generate the client schema file `schema.graphql` ([example](https://github.com/zeit/next.js/blob/canary/examples/with-relay-modern-server-express/server/schema.graphql); `relay-compiler` needs this file) from the `GraphQLSchema` object by using [`printSchema` from `graphql/utilities`](https://graphql.org/graphql-js/utilities/#printschema). Example: [relayjs/relay-examples/todo/scripts/updateSchema.js](https://github.com/relayjs/relay-examples/blob/master/todo/scripts/updateSchema.js)


The resolver function
---------------------

In the example above:

```javascript
(root, args, context, info) => {
  const { id } = args
  return fetchUserById(id)
}
```

A resolver function takes four arguments (in that order):

1. `parent`: The result of the previous resolver call ([more info](https://www.prisma.io/blog/graphql-server-basics-the-schema-ac5e2950214e#9d03))
2. `args`: The arguments of the resolver’s field
3. `context`: A custom object each resolver can read from/write to
4. `info`: contains the query AST and more execution information ([more info](https://www.prisma.io/blog/graphql-server-basics-demystifying-the-info-argument-in-graphql-resolvers-6f26249f613a))


GraphQL server in Python
------------------------

https://github.com/graphql-python/graphql-core-next

```python
from graphql import GraphQLSchema, GraphQLObjectType, GraphQLField, GraphQLString

schema = GraphQLSchema(
    query=GraphQLObjectType(
        name='RootQueryType',
        fields={
            'hello': GraphQLField(
                GraphQLString,
                resolve=lambda obj, info: 'world')
        }))

from aiohttp_graphql import GraphQLView
from aiohttp import web

app = web.Application()
GraphQLView.attach(app, schema=schema, graphiql=True)
web.run_app(app)
```

If you need to create `schema.graphql` file, for example for `relay-compiler`:

```python
from graphql import print_schema
from pathlib import Path

Path('schema.graphql').write_text(print_schema(schema))
```

Alternatives:

- https://github.com/graphql-python/flask-graphql
- https://docs.graphene-python.org/projects/django/en/latest/
- https://github.com/graphql-python/sanic-graphql


Example GraphQL servers
-----------------------

https://github.com/graphql/express-graphql/blob/master/examples/index.js

https://github.com/apollographql/starwars-server/blob/master/data/swapiSchema.js


GraphQL Javascript React client
-------------------------------

Next.js:

- https://github.com/zeit/next.js/tree/canary/examples/with-relay-modern
- https://github.com/zeit/next.js/tree/canary/examples/with-relay-modern-server-express
- https://github.com/zeit/next.js/tree/canary/examples/with-apollo


Links
-----

https://www.prisma.io/blog/graphql-server-basics-the-schema-ac5e2950214e

