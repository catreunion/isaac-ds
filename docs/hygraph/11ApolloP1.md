---
sidebar_position: 12
---

# Apollo Tutorials P1

Think of each object as a node and each relationship as an edge between two nodes. A schema defines this graph structure in Schema Definition Language (SDL). Schema is the single **source of truth** for your data.

[apollographql.com](https://www.apollographql.com/tutorials/) provides a series of amazing tutorials about GraphQL.

Catstronauts : A learning platform for adventurous cats (developers) who want to explore the universe (GraphQL)! 😺 🚀

## Setup

In the directory of your choice with your preferred terminal, clone one of the [apollographql.com](https://www.apollographql.com/tutorials/) repositories that suits your level.

```bash
git clone https://github.com/apollographql/odyssey-lift-off-part1
# or
git clone https://github.com/apollographql/odyssey-lift-off-part2
# or
git clone https://github.com/apollographql/odyssey-lift-off-part3

cd server
yarn
yarn add apollo-server graphql
yarn start
# navigate to http://localhost:4000 in Firefox or Chrome
```

In a new terminal window, navigate to the repo's client directory.

```bash
cd client
yarn
yarn add graphql @apollo/client
yarn start
# navigate to http://localhost:3000 in any browser
```

## Defining the schema

A schema defines this **graph structure** in Schema Definition Language (SDL).

```js title="server/src/schema.js"
const typeDefs = gql`
  type SpaceCat {
    id: ID!
    name: String!
    age: Int
    missions: [Mission]
  }

  type Mission {
    id: ID!
    codename: String!
    to: String!
    scheduled: Boolean!
    crewMembers: [SpaceCat]!
  }
`
```

`gql` : Called a **tagged template literal**. It wraps GraphQL strings with backticks.

`type SpaceCat {}` : Declare an **object type** called SpaceCat in **PascalCase** with **curly brackets**.

`name: String!` : Declare a **field** called name in **camelCase** with a **colon** and **without commas**.

If a field should never be null, add an **exclamation mark** after its type.

`missions: [Mission]` : Declare a field called missions which is an **array** of missions indicated by **square brackets**.

Schema defines what a GraphQL API can and can't do, and how clients can request or change data, like a contract between the server and the client.

**Descriptions** are strings wrapped with **double quotes**.

## Journey of a GraphQL query

1. Our web app (GraphQL client) sends a query, in HTTP POST or GET requests, to the remote GraphQL server. The query itself is formatted as a string.

2. The server **parses** and **transforms** the string into something it can better manipulate : a tree-structured document called an Abstract Syntax Tree (AST).

3. The server **validates** the query against the **types** and **fields** in our **schema**.

- If anything is off, the server throws an error and sends it right back to the app.

- Queries must use valid GraphQL syntax and must only include schema-defined fields.

- If the query looks good, the server can execute it and fetch the data.

4. For **each field** in the query, the server invokes that field's **resolver function** to populate data from a correct source.

5. As all of the query's fields are resolved, the **data is assembled / stitched into a nicely ordered JSON object** with the **exact same shape as the query**.

6. The server assigns the object to the HTTP response body's **data key**, and it's time for the return trip, back to our app.

7. Our web app (GraphQL client) receives the response with exactly the data it needs, passes that data to the right components to render them.

An amazing illustration by [apollographql.com](https://www.apollographql.com/tutorials/)

![An amazing illustration by apollographql.com](https://res.cloudinary.com/apollographql/image/upload/e_sharpen:50,c_scale,q_90,w_1440,fl_progressive/v1617351987/odyssey/lift-off-part2/lop2-1-06_enfbis.jpg)

## Fetching live data from a REST API in Node.js environment

One query is often composed of a mix of different fields and types, coming from different REST API endpoints, with different cache policies.

[The Catstronauts REST API](https://odyssey-lift-off-rest-api.herokuapp.com/)

```text title='contains 6 endpoints'
GET   /tracks
GET   /track/:id
PATCH /track/:id
GET   /track/:id/modules
GET   /author/:id
GET   /module/:id
```

`axios` / `node-fetch` : Libraries providing easy access to HTTP methods and nice async behavior.

```js title='fetch data from the /tracks endpoint'
fetch("apiUrl/tracks").then(function (response) {
  // do something
})
```

```json title='response from the /tracks endpoint'
[
  {
    "id": "c_0",
    "thumbnail": "https://res.cloudinary.com/dety84pbu/image/upload/v1598465568/nebula_cat_djkt9r.jpg",
    "topic": "Cat-stronomy",
    "authorId": "cat-1",
    "title": "Cat-stronomy, an introduction",
    "description": "Curious to learn what Cat-stronomy is all about? Explore the planetary and celestial alignments and how they have affected our space missions.",
    "numberOfViews": 0,
    "createdAt": "2018-09-10T07:13:53.020Z",
    "length": 2377,
    "modulesCount": 10,
    "modules": ["l_0", "l_1", "l_2", "l_3", "l_4", "l_5", "l_6", "l_7", "l_8", "l_9"]
  },
  {...},
]
```

Compare with the **Track object type** in our GraphQL schema

```text title='GraphQL schema'
type Track {
  id: ID!
  title: String!
  author: Author!
  thumbnail: String
  length: Int
  modulesCount: Int
}
```

`author` is missing.

```js title='fetch data from the /author/:id endpoint'
fetch(`apiUrl/author/${authorId}`).then(function (response) {
  // do something
})
```

```json title='response from the /author/:id endpoint'
{
  "id": "cat-1",
  "name": "Henri, le Chat Noir",
  "photo": "https://images.unsplash.com/photo-1442291928580-fb5d0856a8f1?ixlib=rb-1.2.1&q=80&fm=jpg&crop=entropy&cs=tinysrgb&w=1080&fit=max&ixid=eyJhcHBfaWQiOjExNzA0OH0"
}
```

Compare with the **Author object type** in our GraphQL schema

```text title='GraphQL schema'
type Author {
  id: ID!
  name: String!
  photo: String
}
```

## The classic N+1 problem

1 : the call to fetch the top-level tracks field

N : the number of subsequent calls to fetch the author subfield for each track

Making N calls to the exact same endpoint to fetch the exact same data is very inefficient, especially if N is a large number.

```text title='The classic N+1 problem'
{
  tracks {
    title
    author {
      name
    }
  }
}
```

## Implementing `RESTDataSource`

Define methods that will be used when fetching live data from a REST API.

Provide **helper methods** that make API calls efficient.

Resource caching prevents unneccessary API calls for data that doesn't change frequently.

```js title='server/src/datasources/spacecats-api.js'
const { RESTDataSource } = require("apollo-datasource-rest")

class SpaceCatsAPI extends RESTDataSource {
  constructor() {
    super()
    this.baseURL = "https://fake-spacecats-rest-api.cat/"
  }

  getSpaceCats() {
    return this.get("spacecats")
  }
  getMissions(catId) {
    return this.get(`spacecats/${catId}/missions`)
  }
}

module.exports = SpaceCatsAPI
```

[The Catstronauts REST API](https://odyssey-lift-off-rest-api.herokuapp.com/)

```text title='contains 6 endpoints'
GET   /tracks
GET   /author/:id
GET   /track/:id
GET   /track/:id/modules
PATCH /track/:id
GET   /module/:id
```

## 4 optional parameters of a resolver function <-- signature

`parent`

- the returned value of the resolver for this field's parent

- will be useful when dealing with resolver chains.

`args`

- an object that contains all GraphQL arguments that were provided for the field by the GraphQL operation. When querying for a specific item (such as a specific track instead of all tracks), in client-land we'll make a query with an id argument that will be accessible via this args parameter in server-land. We'll cover this further in Lift-off III.

`context`

- an object shared across all resolvers that are executing for a particular operation

- used to share state, e.g. authentication info, database connection or in our case the RESTDataSource

`info`

- contains information about the operation's execution state, including the field name, the path to the field from the root, and more

- used in more advanced actions like setting cache policies at the resolver level

## Implementing Resolvers

- Data resolvers in a GraphQL server can work with any number of data sources.

- Their **keys** correspond to the **object types** in schema.

- A resolver is a function. It has the **same name as the field** that it populates the data for.

- Resolver functions filter the data properties to match only what the query asks for.

The `Query` type contains the **entry points** to our schema. There are two other possible entry points: **Mutation** and **Subscription**

Apollo Sandbox Explorer : A special mode of Apollo Studio that lets you interactively build and test queries against the local GraphQL server.

```js
const resolvers = {
  Query: {
    spaceCats: (_, __, { dataSources }) => {
      return dataSources.spaceCatsAPI.getSpaceCats()
    }
  }
  SpaceCat: {
    missions: ({catId}, _, {dataSources}) => {
      return dataSources.spaceCatsAPI.getMissions(catId)
    }
  }
}
```

## Connecting schema, resolvers and data sources

```js
const server = new ApolloServer({
  typeDefs,
  resolvers,
  dataSources: () => {
    return {
      spaceCatsAPI: new SpaceCatsAPI()
    }
  }
})
```

## Testing with Apollo Explorer

```text title='Lift-off I'
query GetTracks {
  tracksForHome {
    id
    title
    thumbnail
    length
    modulesCount
    author {
      id
      name
      photo
    }
  }
}
```

```
const lastTrackEntry = {
  //paste the last track entry here
}
```

```js title='server/src/schema.js'
const typeDefs = gql`
  type Query {
    tracksForHome: [Track!]!
    tracksForHomeFetch: [Track!]!
  }

  # ...
`
```

```js title='server/src/resolvers.js'
const resolvers = {
  Query: {
    tracksForHome: () => {
      // ...
    }
    tracksForHomeFetch: async () => {
      const baseUrl = "https://odyssey-lift-off-rest-api.herokuapp.com";
      const res = await fetch(`${baseUrl}/tracks`);
      return res.json();
    },
  },
  Track: {
    // using fetch instead of dataSources
    author: async ({ authorId }, _, { dataSources }) => {
      const baseUrl = "https://odyssey-lift-off-rest-api.herokuapp.com";
      const res = await fetch(`${baseUrl}/author/${authorId}`);
      return res.json();

      // return dataSources.trackAPI.getAuthor(authorId);
    },
  },
}
```

## Error returned due to submitting a query with an invalid field

Things never go smoothly in the real world.

underlined by a red squiggly line

help to narrow down what caused the issues.

There are more than a few ways things can go south.