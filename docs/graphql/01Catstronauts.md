---
sidebar_position: 1
---

# Catstronauts

A learning platform for adventurous cats who want to explore the universe! 😺 🚀 Created by [Apollo](https://www.apollographql.com/tutorials/), a series of amazing tutorials about GraphQL with super helpful videos and interactive code challenges along the way. Thank you the productioin team!! 🙏🏻

## Project setup 🔧

In the directory of your choice with your preferred terminal, clone one of the repositories createdy by [Apollo](https://www.apollographql.com/tutorials/voyage-part1/intro-to-federation).

```bash title="repos from Apollo Odyssey tutorials"
# clone one of them that suits you
git clone https://github.com/apollographql/odyssey-lift-off-part1
git clone https://github.com/apollographql/odyssey-lift-off-part2
git clone https://github.com/apollographql/odyssey-lift-off-part3
git clone https://github.com/apollographql/odyssey-lift-off-part4
git clone https://github.com/apollographql/odyssey-lift-off-part5-server.git
git clone https://github.com/apollographql/odyssey-lift-off-part5-client.git

# navigate to the server directory
cd odyssey-lift-off-part1/server

# install the default dependencies
yarn

# install additional dependencies
yarn add apollo-server apollo-datasource-rest graphql

# start the backend dev server
yarn start

# navigate to localhost:4000 in a web browser
```

```bash title="in a new terminal window"
# navigate to the client directory
cd odyssey-lift-off-part1/client

# install the default dependencies
yarn

# install additional dependencies
yarn add graphql @apollo/client

# start the frontend dev server
yarn start

# navigate to localhost:3000 in a web browser
```

## What data do we need?

Think of an app's data as a collection of **objects** and **relationships** between objects. Think of the entire data model as a **graph** of nodes and edges.

An illustration by [Apollo](https://www.apollographql.com/tutorials/lift-off-part1/feature-data-requirements)

![A collection of objects and the relationships among them](https://res.cloudinary.com/apollographql/image/upload/e_sharpen:50,c_scale,q_90,w_1440,fl_progressive/v1612409160/odyssey/lift-off-part1/LO_02_v2.00_04_53_09.Still002_g8xow6_bbgabz.jpg)

A **schema** defines a collection of **object types** and the **relationships** between object types. Like a contract, it defines what a GraphQL server can and can't do, and how clients can request or change data.

Schema Definition Language (SDL)

Most of the types defined in a schema are object types. An object type contains a collection of **fields**. Each field **returns** data of the type specified. A field's return type can be a scalar, object, input, enum, union or interface. **Schema does not store data**, database does.

> scalar = primitive, always resolve to a single value

> `Int` : Signed 32‐bit integer

> `Float` : Signed double-precision floating-point value

> `String` : UTF‐8 character sequence

> `Boolean` : true or false

> `ID` : Unique identifier. Serialized as a String.

The following object types include each other as fields.

```js title="server/src/schema.js"
// declare a constant called typeDefs (short for "type definitions")
// wrap GraphQL strings with the gql tag (template literal)
// convert/parse GraphQL strings into the format that Apollo libraries understand
// use backticks (`), don't confused with single quotes (')

const typeDefs = gql`
  # A Book has an associated author.
  type Book {
    title: String!
    author: Author
  }

  # An Author can have a list of books.
  type Author {
    name: String!
    books: [Book!]!
  }
`
```

Declare an object type in PascalCase with curly brackets.

Declare a field in camelCase with a colon and without commas.

If a field should never be null (non-nullable), add an exclamation mark after its type.

Structure a schema as intuitively as possible. Each object type you define should support the actions that your clients will take.

Every object type automatically has a field named `__typename`. It returns the typename of the object (e.g., Book or Author).

## Query type

The Query type is a **special object type** that defines the **top-level entry points** where clients can fetch data against the schema. **Each field** defines the **name** and **return type** of an entry point.

In the following example, each field returns a list of the corresponding type.

```graphql title="server/src/schema.js"
type Query {
  books: [Book]
  authors: [Author]
}
```

Say there is a REST-based API with `/api/books` and `/api/authors` entry points.

```graphql title="fetching data from multiple entry points in a single query"
query GetBooks {
  books {
    title
    author {
      name
    }
  }
}
```

```json title="response from GraphQL server"
{
  "data": {
    "books": [
      {
        "title": "City of Glass",
        "author": {
          "name": "Paul Auster"
        }
      },
      ...
    ]
  }
}
```

## Mutation type

The Mutation type is a special object type that enables clients to modify data / execute. The **fields** of the Mutation type are **also entry points** into the GraphQL API. Each field defines the signature and **return type** of an entry point.

Example 1 : too add a book

Need two arguments.

Return a newly created Book object.

```graphql title="server/src/schema.js"
type Mutation {
  addBook(title: String, author: String): Book
}
```

```graphql title="executing the AddBook operation"
mutation AddBook {
  addBook(title: "Fox in Socks", author: "Dr. Seuss") {
    title
    author {
      name
    }
  }
}
```

Example 2 : to "like" a blog post

Increment the likes count of the post.

Update the likedPosts list of the user.

A single mutation can modify multiple types, or multiple instances of the same type.

Example 3 : An illustration by [Apollo](https://www.apollographql.com/tutorials/lift-off-part1/feature-data-requirements) showing **SpaceCat** and **Mission** object types.

![howing SpaceCat and Mission object types](https://res.cloudinary.com/apollographql/image/upload/e_sharpen:50,c_scale,q_90,w_1440,fl_progressive/v1624650767/odyssey/lift-off-part4/doodle_schema_ea1ivm.png)

A spacecat has a list of missions assigned to.

A mission can have more than one spacecat assigned to it.

> Assign one spacecat to a particular mission.

> Update that spacecat's list of missions.

> Update the mission lists of other crew members.

## Mutation Response

A mutation response is an object type created for **storing the return type of a mutation**. It contains 3 properties, as well as additional fields for each object that the mutation updates.

> `code` : Int! - Status of the mutation, similar to an HTTP status code.

> `success` : Boolean! - Whether the mutation was successful.

> `message` : String! - A human-readable message for the UI

Example 1 :

An illustration by [Apollo](https://www.apollographql.com/tutorials/lift-off-part1/feature-data-requirements) showing the `AssignMissionResponse` return type.

![The `AssignMissionResponse` return type](https://res.cloudinary.com/apollographql/image/upload/e_sharpen:50,c_scale,q_90,w_1440,fl_progressive/v1624651213/odyssey/lift-off-part4/doodle_mutation_return_type_jklp3a.png)

```graphql title="server/src/schema.js"
type AssignSpaceshipResponse {
  code: Int!
  success: Boolean!
  message: String!
  spaceship: Spaceship
  mission: Mission
}

type Mutation {
  assignSpaceship(spaceshipId: ID!, missionId: ID!): AssignSpaceshipResponse!
}
```

Example 2 :

Say there is a mutation defined for updating a user's email address. The object returned should be structured in something like this.

```json title="response from GraphQL server"
{
  "data": {
    "updateUserEmail": {
      "code": "200",
      "success": true,
      "message": "User email was successfully updated",
      "user": {
        "id": "1",
        "name": "Jane Doe",
        "email": "jane@example.com"
      }
    }
  }
}
```

Example 3 :

Say there is a mutation defined for liking a blog post. The object returned should be structured in something like this.

```json title="response from GraphQL server"
{
  "data": {
    "likePost": {
      "code": "200",
      "success": true,
      "message": "Thanks!",
      "post": {
        "id": "123",
        "likes": 5040
      },
      "user": {
        "likedPosts": ["123"]
      }
    }
  }
}
```

## Using in Apollo Sandbox

[Apollo Studio](https://studio.apollographql.com/) is a powerful web IDE for exploring a GraphQL schema and building queries against it.

[Apollo Sandbox](https://studio.apollographql.com/sandbox) is a special mode of Apollo Studio. It allows using the Studio features without an Apollo account.

```graphql title="for Catstronauts homepage"
query GetTracksForHome {
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

## Journey of a query

The web app (GraphQL client) sends a GraphQL query operation, formatted as a string in HTTP POST or GET request, to the GraphQL server.

The server **parses** and **transforms** the string into a tree-structured document called an Abstract Syntax Tree (AST).

The server **validates** the query operation against the **types** and **fields** defined in **schema**.

For **each field** in the query, the server invokes that field's **resolver function** to populate data from the correct source.

As all of the query's fields are resolved, the data is **assembled / stitched into a JSON object** with the **exact same shape as the query**.

The server assigns the JSON object to the **data key** of the HTTP response body, and send it back to the web app.

The web app (GraphQL client) receives the response and passes the data to the right components for rendering.

An illustration by [Apollo](https://www.apollographql.com/tutorials/lift-off-part2/journey-of-a-graphql-query) showing the journey of a GraphQL query operation.

![The journey of a GraphQL query operation](https://res.cloudinary.com/apollographql/image/upload/e_sharpen:50,c_scale,q_90,w_1440,fl_progressive/v1617351987/odyssey/lift-off-part2/lop2-1-06_enfbis.jpg)

## Where is our raw data?

[the Catstronauts REST API](https://odyssey-lift-off-rest-api.herokuapp.com/)

```text title="6 endpoints"
GET   /tracks
GET   /track/:id
GET   /author/:id
GET   /track/:id/modules
PATCH /track/:id
GET   /module/:id
```

An illustration by [Apollo](https://www.apollographql.com/tutorials/lift-off-part2/exploring-our-data) showing the data sources that GraphQL server supports.

![The data sources that GraphQL server supports](https://res.cloudinary.com/apollographql/image/upload/e_sharpen:50,c_scale,q_90,w_1440,fl_progressive/v1612408870/odyssey/lift-off-part2/lop2-2-01_actpy7.jpg)

## How is the raw data structured?

A GraphQL query operation is often composed of a mix of fields and types, coming from **different endpoints**, with different cache policies.

```graphql title="for Catstronauts homepage"
query getTracksForHome {
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

```json title="raw data from /tracks endpoint""
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

The raw data has the `authorId` field. It's value can be used in the `author/:id` endpoint.

The raw data contains an array of modules. Their values can be used in the `track/:id/modules` endpoint.

## The n+1 problem

One call to fetch tracks but n subsequent calls to fetch the author subfield for each track.

Making n calls to the exact same endpoint to fetch the exact same data is very inefficient, especially if n is a large number.

```graphql title="the n+1 problem"
{
  tracks {
    title
    author {
      # n calls for n tracks
      name
    }
  }
}
```

## The `RESTDataSource` class

Methods are defined here for making API calls.

```js title="server/src/datasources/spacecats-api.js"
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
```

## Resolvers & resolver ⛓️

A resolver is a function that's responsible for populating live data for a **single field** from a data source. Whenever a client queries for a particular field, the resolver for that field fetches the requested data from the appropriate data source. **A resolver's name must be the same as the field** that it populates the data for.

```js title="4 parameters of a resolver"
fieldName: (parent, args, context, info) => data
```

`parent` : Contain the **data** returned from the **previous** function in a **resolver chain**.

`args` : Contain all the **arguments** provided for querying a specific object.

`context` : Contain authentication info and all the **methods** defined in `RESTDataSource`.

`info` : Contain informational properties about the **execution state**.

Keep each resolver lightweight and responsible for one piece of data. --> Easy to understand and more resilient to future changes

```js title="server/src/resolvers.js"
const resolvers = {
  Query: {
    spaceCats: (_, __, { dataSources }) => {
      return dataSources.spaceCatsAPI.getSpaceCats()
    },
    spaceCat: (_, { id }, { dataSources }) => {
      return dataSources.spaceCatsAPI.getSpaceCat(id)
    }
  }
  SpaceCat: {
    missions: ({ id }, _, { dataSources }) => {
      return dataSources.spaceCatsAPI.getMissions(id)
    }
  }
}
```

## Query with arguments

An argument is a value provided for a particular field in a query. It is used to retrieve specific objects, filter through a set of objects, or even transform the field's returned value.

An illustration by [Apollo](https://www.apollographql.com/tutorials/lift-off-part1/feature-data-requirements) showing the syntax breakdown of arguments definition.

![The syntax breakdown of arguments definition](https://res.cloudinary.com/apollographql/image/upload/e_sharpen:50,c_scale,q_90,w_1440,fl_progressive/v1623354512/odyssey/lift-off-part3/args-syntax_t0wseq.jpg)

Add parentheses after the field name.

Write the name of the argument followed by a colon, then the type of that argument.

Separate multiple arguments with commas.

An illustration by [Apollo](https://www.apollographql.com/tutorials/lift-off-part1/feature-data-requirements) - A resolver function retrieving a specific object using an argument.

![A resolver function retrieving a specific object from data-land using an argument](https://res.cloudinary.com/apollographql/image/upload/e_sharpen:50,c_scale,q_90,w_1440,fl_progressive/v1623354512/odyssey/lift-off-part3/args-find_xpn4en.jpg)

## A closer look to ⛓️

The `Query.track` resolver and the `Track.author` resolver are linked to form a **resolver chain**.

An illustration by [Apollo](https://www.apollographql.com/tutorials/lift-off-part1/feature-data-requirements) showing the `Query.track` resolver passes data to the `Track.author` resolver. The data is known as **parent parameter**.

![Showing the `Query.track` resolver passes data to the `Track.author` resolver](https://res.cloudinary.com/apollographql/image/upload/e_sharpen:50,c_scale,q_90,w_1440,fl_progressive/v1623355358/odyssey/lift-off-part3/resolver-parent_kne6hn.jpg)

## Sandbox & variables

```bash title="opening Apollo Sandbox"
# navigate to the server directory
cd server/

# start up the server
yarn start

# navigate to http://localhost:4000 in Firefox or Chrome
```

```graphql title="getting a track's details"
query GetTrack($trackId: ID!) {
  track(id: $trackId) {
    id
    title
    author {
      id
      name
      photo
    }
    thumbnail
    length
    modulesCount
    numberOfViews
    modules {
      id
      title
      length
    }
    description
  }
}
```

The $ symbol indicates a variable in GraphQL. The name after the $ symbol is the variable name. After the colon is the variable's type, which must match the type specified in the **schema**.

```bash title="passing an argument to the variable"
{
  "trackId": "c_0"
}
```

```graphql title="if prefer hardcoding a value"
query track(id: ‘c_0') {
  title
  author {
    name
  }
}
```

```graphql title=""
query GetMission($isScheduled: Boolean) {
  mission(scheduled: $isScheduled) {
    id
    codename
  }
}
```

## Isaac's outdoor blog

```bash title="connecting to Hygraph"
# on the existing Sandbox, change the address to
# https://api-us-east-1.hygraph.com/v2/clbq4ju4z13gl01uuf7xi0ulm/master
```

```graphql title="getting activities (brief)"
query GetActivitiesBrief {
  activities {
    id
    activityDate
    desc {
      text
    }
  }
}
```

```graphql title="getting an activity with id"
query GetAnActivity($where: ActivityWhereInput) {
  activities(where: $where) {
    id
    activityDate
    title
    avgPace
    calories
    totalAscent
    totalDescent
    avgHr
    maxHr
    hours
    mins
    secs
    desc {
      text
    }
  }
}
```

```bash title="passing an argument to the variable"
{
  "where": {
    "id": "clbq5ubpk02u70binbk2el2jd"
  }
}
```

```graphql title="if prefer hardcoding a slug"
query GetAnActivitySlug {
  activities(where: { slug: "2022-dec-15-kowloon-peak-running" }) {
    id
    activityDate
    title
    avgPace
    calories
    totalAscent
    totalDescent
    avgHr
    maxHr
    hours
    mins
    secs
    desc {
      text
    }
  }
}
```

## Putting all together

```js title="server/src/schema.js"
const typeDefs = gql`
  type Track {
    id: ID!
    title: String!
    author: Author!
    thumbnail: String
    length: Int
    modulesCount: Int
    description: String
    numberOfViews: Int
    modules: [Module!]!
  }

  type Author {
    id: ID!
    name: String!
    photo: String
  }

  type Module {
    id: ID!
    title: String!
    length: Int
    content: String
    videoUrl: String
  }

  type Query {
    tracksForHome: [Track!]!
    "Query for a specific track provided with the track's ID."
    track(id: ID!): Track!
    "Query for a specific module provided with the module's ID"
    module(id: ID!): Module!
  }
`
```

```js title="server/src/datasources/track-api.js"
class TrackAPI extends RESTDataSource {
  constructor() {
    super()
    this.baseURL = "https://odyssey-lift-off-rest-api.herokuapp.com/"
  }
  getTracksForHome() {
    return this.get("tracks")
  }
  getTrack(trackId) {
    return this.get(`track/${trackId}`)
  }
  getAuthor(authorId) {
    return this.get(`author/${authorId}`)
  }
  getTrackModules(trackId) {
    return this.get(`track/${trackId}/modules`)
  }
  incrementTrackViews(trackId) {
    return this.patch(`track/${trackId}/numberOfViews`)
  }
}
```

```js title="server/src/resolvers.js"
const resolvers = {
  Query: {
    tracksForHome: (_, __, { dataSources }) => {
      return dataSources.trackAPI.getTracksForHome()
    },
    track: (_, { id }, { dataSources }) => {
      return dataSources.trackAPI.getTrack(id)
    }
  },
  Track: {
    author: ({ authorId }, _, { dataSources }) => {
      return dataSources.trackAPI.getAuthor(authorId)
    },
    modules: ({ id }, _, { dataSources }) => {
      return dataSources.trackAPI.getTrackModules(id)
    }
  }
}
```

```js title="server/src/index.js"
const server = new ApolloServer({
  typeDefs,
  resolvers,
  dataSources: () => {
    return {
      trackAPI: new TrackAPI()
    }
  }
})
```

## The `useQuery` hook

The `useQuery` hook returns **an object** with **three properties**.

`loading` indicates whether the query has completed and results have been returned.

`error` is an object that contains any errors that the operation has thrown.

`data` contains the results of the query after it has completed.

To set **variables** in a query, declare them in the **second** parameter of the `useQuery` hook.

```js
const GET_SPACECAT = gql`
  query GetSpacecat($spaceCatId: ID!) {
    spacecat(id: $spaceCatId) {
      name
    }
  }
`

const spaceCatId = "kitty-1"

const Cat = ({ spaceCatId }) => {
  const { loading, error, data } = useQuery(GET_SPACECAT, {
    variables: { spaceCatId }
  })

  return (
    <Layout>
      <QueryResult loading={loading} error={error} data={data}>
        <TrackDetail track={data?.track} />
      </QueryResult>
    </Layout>
  )
}

export default Cat
```

[Lift-off III: Arguments | Apollo Odyssey](https://www.apollographql.com/tutorials/lift-off-part3/the-usequery-hook---with-variables)

## Testing with Sandbox

```bash title="opening Apollo Sandbox"
# navigate to the server directory
cd server/

# start up the server
yarn start

# navigate to http://localhost:4000 in Firefox or Chrome
```

```graphql title="exe mutation operation"
mutation IncrementTrackViews($incrementTrackViewsId: ID!) {
  incrementTrackViews(id: $incrementTrackViewsId) {
    code
    success
    message
    track {
      id
      numberOfViews
    }
  }
}
```

```bash title="passing an argument to the variable"
{
  "incrementTrackViewsId": "c_0"
}
```

```js title="server/src/resolvers.js"
Mutation: {
  assignSpaceship: async (_, { spaceshipId, missionId }, { dataSources }) => {
    try {
      const { spaceship, mission } = await dataSources.spaceAPI.assignSpaceshipToMission(spaceshipId, missionId)
      return {
        code: 200,
        success: true,
        message: `Successfully assigned spaceship ${spaceshipId} to mission ${missionId}`,
        spaceship: spaceship,
        mission: mission
      }
    } catch (err) {
      return {
        code: err.extensions.response.status,
        success: false,
        message: err.extensions.response.body,
        spaceship: null,
        mission: null
      }
    }
  }
}
```

## Deploy the backend

The server and client apps need to be split into two separate repos.

The backend will be deployed to Apollo schema registry.

The frontend will be deployed to x.

Schema evolution

evolve over time as we add new features and improve our codebase.

the schema registry is a version control system for our schema. It stores our schema's change history, tracking the types and fields that were added, modified, and removed. The registry powers almost every Apollo feature.

can track variants of the same graph that are deployed in different environments, such as staging and production. We can run schema checks to detect when a potential change might break one of our clients.

As our graph grows, multiple teams might even want to break parts of it into separate subgraphs that they manage independently. The schema registry can take care of tracking all those subgraphs, enabling Apollo to surface potential conflicts between them, and even powering schema composition with Apollo managed federation!

storing schema change history for a single graph and field usage.

Register our schema with Apollo

[Apollo Studio](https://studio.apollographql.com)

Sandbox is a **development** environment where we can explore and test a GraphQL schema. To register a **production** schema and track our changes over time, we can create a **deployed graph** in Apollo Studio, which is visible to **the whole organization**.

Select Monolith as the graph architecture

Apollo Server uses a protocol called schema reporting, where our GraphQL server automatically registers its latest schema every time it starts up! To enable this, we need to set up the three environment variables shown on the page:

```env title=""
APOLLO_KEY=service:xxxxx
APOLLO_GRAPH_REF=xxxxxx
APOLLO_SCHEMA_REPORTING=true
```

the structure of the graph reference variable is really <APOLLO_GRAPH_ID>@<APOLLO_GRAPH_VARIANT>.

Railway automatically set the NODE_ENV environment variable to production, which alerts our **Apollo Server** to switch itself to production as well, **automatically disabling introspection**. Tools like Apollo Sandbox use introspection to build and run queries. Introspection enables us to query a GraphQL server for data like types, fields, and field-level descriptions.

An illustration by [Apollo](https://www.apollographql.com/tutorials/lift-off-part5/deploying-apollo-server) showing introspection in development environments.

![introspection in development environments](https://res.cloudinary.com/apollographql/image/upload/e_sharpen:50,c_scale,q_90,w_1440,fl_progressive/v1629226863/odyssey/lift-off-part5/e4-introspect-dev_gdfpnj.jpg)

Having introspection in production can be a major security issue. It exposes all of our graph's structure and what we can do with it to the whole world.

An illustration by [Apollo](https://www.apollographql.com/tutorials/lift-off-part5/deploying-apollo-server) showing introspection in production environments, where introspection is turned off.

![introspection in production environments, where introspection is turned off](https://res.cloudinary.com/apollographql/image/upload/e_sharpen:50,c_scale,q_90,w_1440,fl_progressive/v1629226863/odyssey/lift-off-part5/e4-introspect-prod_co4itc.jpg)

Use [Apollo Studio](http://studio.apollographql.com) instead. Query our server at a public URL on the internet.

```graphql title="on Apollo Studio Explorer"
query GetTracks {
  tracksForHome {
    id
    title
    thumbnail
    length
    modulesCount
    author {
      name
      photo
    }
  }
}
```

The Explorer will prompt you for the **Production URL**. This is the domain URL we generated in Railway.

Railway needs our server to run on a port specified by the PORT environment variable, which is set behind the scenes.

```js title="server/src/index.js"
// update the port number by specifying an options object in the listen method.
await server.listen({ port: process.env.PORT || 4000 })
```

```js title="server/src/index.js"
async function startApolloServer(server) {
  // listen to a port specified by process.env.PORT, or if that doesn't exist, a hard-coded port number 4000
  const { url } = await server.listen({ port: process.env.PORT || 4000 })
  console.log(`🚀  Server ready at ${url}`)
}

startApolloServer(server)
```

On the Railway dashboard, click New. Select the GitHub Repo option. Connect the backend repository. Input environment variables. Click Generate Domain.

The backend is live in production.

## Deploy the frontend

initialized our Apollo Client with properties such as the uri and cache

Change the value of `uri` to the backend live address

```js title="src/index.js"
const client = new ApolloClient({
  uri: "https://odyssey-lift-off-part5-server-production.up.railway.app/",
  cache: new InMemoryCache()
})
```

Commit and push changes to GitHub.

On the Railway dashboard, click New. Select the GitHub Repo option. Connect the client app repository. Click Generate Domain.

The frontend is up and running in Interent.

## wording

Things never go smoothly in the real world.

underlined by a red squiggly line

help to narrow down what caused the issues.

There are more than a few ways things can go south.

red squiggly

The key to happiness must be found somewhere else.

```js title="if using fetch"
const resolvers = {
  Query: {
    tracksForHomeFetch: async () => {
      const baseUrl = "https://odyssey-lift-off-rest-api.herokuapp.com"
      const res = await fetch(`${baseUrl}/tracks`)
      return res.json()
    }
  },
  Track: {
    // using fetch instead of dataSources
    author: async ({ authorId }, _, { dataSources }) => {
      const baseUrl = "https://odyssey-lift-off-rest-api.herokuapp.com"
      const res = await fetch(`${baseUrl}/author/${authorId}`)
      return res.json()

      // return dataSources.trackAPI.getAuthor(authorId);
    }
  }
}
```
