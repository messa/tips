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


Sample servers
--------------

https://github.com/apollographql/starwars-server/blob/master/data/swapiSchema.js


Links
-----

https://www.prisma.io/blog/graphql-server-basics-the-schema-ac5e2950214e

