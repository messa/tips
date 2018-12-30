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

const app = express();

app.use('/graphql', graphqlHTTP({
  schema: MyGraphQLSchema,
  graphiql: true
}))

app.listen(4000)
```

Dependencies:

```shell
$ npm install --save graphql express-graphql
```


Sample servers
--------------

https://github.com/apollographql/starwars-server/blob/master/data/swapiSchema.js


Links
-----

https://www.prisma.io/blog/graphql-server-basics-the-schema-ac5e2950214e

