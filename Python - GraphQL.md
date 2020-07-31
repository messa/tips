Python and GraphQL
==================

Graphene
--------

https://graphene-python.org/

```python
#!/usr/bin/env python3

'''
https://blog.graphqleditor.com/top-3-python-libraries-for-graphql/
'''

from aiohttp.web import run_app, Application
#from aiohttp_graphql import GraphQLView
from graphql_server.aiohttp import GraphQLView
import graphene


class Query(graphene.ObjectType):
    hello = graphene.String(description='A typical hello world')

    def resolve_hello(self, info):
        return 'World'

schema = graphene.Schema(query=Query)


def main():
    setup_logging()
    app = Application()
    print(type(schema))
    GraphQLView.attach(app, schema=schema, graphiql=True)
    run_app(app)



def setup_logging():
    from logging import DEBUG, basicConfig
    basicConfig(level=DEBUG)


if __name__ == '__main__':
    main()
```


Tartiflette
-----------

https://tartiflette.io/

```
tartiflette-aiohttp
```

```python
#!/usr/bin/env python3

'''
https://tartiflette.io/docs/tutorial/write-your-resolvers
https://tartiflette.io/docs/api/resolver
'''

from aiohttp import web
from asyncio import sleep
import os
import sys
from tartiflette_aiohttp import register_graphql_handlers
from tartiflette import Engine, Resolver
from textwrap import dedent
from typing import Any, Dict, List, Optional


# Dictionnary which contains the ingredients based on the recipe ID as key.
INGREDIENTS = {
    1: [
        {"name": "potato", "quantity": 10, "unitMeasurement": "UNIT"},
        {"name": "onion", "quantity": 2, "unitMeasurement": "UNIT"},
        {"name": "bacon", "quantity": 100, "unitMeasurement": "GRAM"},
        {"name": "white wine", "quantity": 0.05, "unitMeasurement": "LITER"},
        {"name": "reblochon AOP", "quantity": 1, "unitMeasurement": "UNIT"},
    ],
    2: [
        {"name": "potato", "quantity": 1000, "unitMeasurement": "GRAM"},
        {"name": "bacon", "quantity": 200, "unitMeasurement": "GRAM"},
        {"name": "onion", "quantity": 200, "unitMeasurement": "GRAM"},
        {"name": "reblochon AOP", "quantity": 1, "unitMeasurement": "UNIT"},
        {
            "name": "tablespoon of oil",
            "quantity": 2,
            "unitMeasurement": "UNIT",
        },
        {"name": "clove of garlic", "quantity": 1, "unitMeasurement": "UNIT"},
    ],
    3: [
        {"name": "potato", "quantity": 1000, "unitMeasurement": "GRAM"},
        {"name": "smoked bacon", "quantity": 200, "unitMeasurement": "GRAM"},
        {"name": "onion", "quantity": 2, "unitMeasurement": "UNIT"},
        {"name": "reblochon AOP", "quantity": 1, "unitMeasurement": "UNIT"},
        {"name": "fresh cream", "quantity": 0.100, "unitMeasurement": "LITER"},
        {"name": "butter", "quantity": 40, "unitMeasurement": "GRAM"},
        {
            "name": "tablespoon of pepper",
            "quantity": 1,
            "unitMeasurement": "UNIT",
        },
    ],
}

RECIPES = [
    {"id": 1, "name": "Tartiflette by Eric Guelpa", "cookingTime": 15},
    {"id": 2, "name": "La 'vraie' Tartiflette", "cookingTime": 20},
    {"id": 3, "name": "Tartiflette by Alain Ducasse", "cookingTime": 25},
]

QUERY_SCHEMA = dedent('''
    enum UnitMeasurement {
      GRAM
      LITER
      UNIT
    }

    type Ingredient {
      name: String!
      quantity: Float!
      unitMeasurement: UnitMeasurement!
    }

    type Recipe {
      id: Int!
      name: String!
      ingredients: [Ingredient!]!
      cookingTime: Int!
    }

    type Query {
      hello(name: String): String
      recipes: [Recipe!]
      recipe(id: Int!): Recipe
    }
''')


MUTATION_SCHEMA = dedent('''
    input RecipeInput {
      id: Int!
      name: String
      cookingTime: Int
    }

    type Mutation {
      updateRecipe(input: RecipeInput!): Recipe!
    }
''')


@Resolver("Query.hello")
async def resolver_hello(parent, args, ctx, info):
    await sleep(1)
    return "hello " + args["name"]


@Resolver("Query.recipes")
async def resolve_query_recipes(
    parent: Optional[Any],
    args: Dict[str, Any],
    ctx: Dict[str, Any],
    info: "ResolveInfo",
) -> List[Dict[str, Any]]:
    """
    Resolver in charge of returning all recipes.
    :param parent: initial value filled in to the engine `execute` method
    :param args: computed arguments related to the field
    :param ctx: context filled in at engine initialization
    :param info: information related to the execution and field resolution
    :type parent: Optional[Any]
    :type args: Dict[str, Any]
    :type ctx: Dict[str, Any]
    :type info: ResolveInfo
    :return: the list of all recipes
    :rtype: List[Dict[str, Any]]
    """
    return RECIPES


@Resolver("Query.recipe")
async def resolve_query_recipe(
    parent: Optional[Any],
    args: Dict[str, Any],
    ctx: Dict[str, Any],
    info: "ResolveInfo",
) -> Optional[Dict[str, Any]]:
    """
    Resolver in charge of returning a recipe depending on the filled in `id`.
    :param parent: initial value filled in to the engine `execute` method
    :param args: computed arguments related to the field
    :param ctx: context filled in at engine initialization
    :param info: information related to the execution and field resolution
    :type parent: Optional[Any]
    :type args: Dict[str, Any]
    :type ctx: Dict[str, Any]
    :type info: ResolveInfo
    :return: a recipe
    :rtype: Optional[Dict[str, Any]]
    """
    for recipe in RECIPES:
        if recipe["id"] == args["id"]:
            return recipe
    return None


@Resolver("Recipe.ingredients")
async def resolve_recipe_ingredients(
    parent: Optional[Dict[str, Any]],
    args: Dict[str, Any],
    ctx: Dict[str, Any],
    info: "ResolveInfo",
) -> Optional[List[Dict[str, Any]]]:
    """
    Resolver in charge of returning the ingredient list of a recipe.
    :param parent: the recipe for which to return the ingredients
    :param args: computed arguments related to the field
    :param ctx: context filled in at engine initialization
    :param info: information related to the execution and field resolution
    :type parent: Optional[Dict[str, Any]]
    :type args: Dict[str, Any]
    :type ctx: Dict[str, Any]
    :type info: ResolveInfo
    :return: the ingredient list of a recipe
    :rtype: Optional[List[Dict[str, Any]]]
    """
    if parent and parent["id"] in INGREDIENTS:
        return INGREDIENTS[parent["id"]]
    return None


@Resolver("Mutation.updateRecipe")
async def resolve_mutation_update_recipe(
    parent: Optional[Any],
    args: Dict[str, Any],
    ctx: Dict[str, Any],
    info: "ResolveInfo",
) -> Dict[str, Any]:
    """
    Resolver in charge of the mutation of a recipe.
    :param parent: initial value filled in to the engine `execute` method
    :param args: computed arguments related to the mutation
    :param ctx: context filled in at engine initialization
    :param info: information related to the execution and field resolution
    :type parent: Optional[Any]
    :type args: Dict[str, Any]
    :type ctx: Dict[str, Any]
    :type info: ResolveInfo
    :return: the mutated recipe
    :rtype: Dict[str, Any]
    :raises Exception: if the recipe id doesn't exist
    """
    recipe_id = args["input"]["id"]
    name = args["input"].get("name")
    cooking_time = args["input"].get("cookingTime")

    if not (name or cooking_time):
        raise Exception(
            "You should provide at least one value for either name or "
            "cookingTime."
        )

    for index, recipe in enumerate(RECIPES):
        if recipe["id"] == recipe_id:
            if name:
                RECIPES[index]["name"] = name
            if cooking_time:
                RECIPES[index]["cookingTime"] = cooking_time
            return RECIPES[index]

    raise Exception(f"The recipe < {recipe_id} > does not exist.")


def main():
    """
    Entry point of the application.
    """
    app = web.Application()
    register_graphql_handlers(
        app=app,
        #engine_sdl=os.path.dirname(os.path.abspath(__file__)) + "/sdl",
        engine_sdl=QUERY_SCHEMA + MUTATION_SCHEMA,
        #engine_modules=[
        #    "recipes_manager.query_resolvers",
        #    "recipes_manager.mutation_resolvers",
        #    "recipes_manager.subscription_resolvers",
        #    "recipes_manager.directives.rate_limiting",
        #    "recipes_manager.directives.auth",
        #],
        executor_http_endpoint="/graphql",
        executor_http_methods=["POST"],
        graphiql_enabled=True,
    )
    web.run_app(app)


if __name__ == '__main__':
    sys.exit(main())
```


Ariadne
-------

https://ariadnegraphql.org/

```
pip install ariadne

# for the example:
pip install uvicorn
```

Example of GraphQL endpoint implemented using Ariadne:

```python
'''
https://blog.graphqleditor.com/top-3-python-libraries-for-graphql/
'''

from ariadne import ObjectType, QueryType, gql, make_executable_schema
from ariadne.asgi import GraphQL

# Define types using Schema Definition Language (https://graphql.org/learn/schema/)
# Wrapping string in gql function provides validation and better error traceback
type_defs = gql("""
    type Query {
        people: [Person!]!
    }

    type Person {
        firstName: String
        lastName: String
        age: Int
        fullName: String
    }
""")

# Map resolver functions to Query fields using QueryType
query = QueryType()

# Resolvers are simple python functions
@query.field("people")
def resolve_people(*_):
    return [
        {"firstName": "John", "lastName": "Doe", "age": 21},
        {"firstName": "Bob", "lastName": "Boberson", "age": 24},
    ]


# Map resolver functions to custom type fields using ObjectType
person = ObjectType("Person")

@person.field("fullName")
def resolve_person_fullname(person, *_):
    return "%s %s" % (person["firstName"], person["lastName"])

# Create executable GraphQL schema
schema = make_executable_schema(type_defs, [query, person])

# Create an ASGI app using the schema, running in debug mode
app = GraphQL(schema, debug=True)
```


Strawberry
----------

https://github.com/strawberry-graphql/strawberry




