# Items Creation and Prisma Yoga Flow - Mutation
## Create Feature Branch
`$ git checkout -b feature-branch`

* Replace `feature-branch` with the name of your feature branch

# Don't know
`datamodel.prisma`

```
type User {
  id: ID! @unique
  name: String!
  email: String!
}
```

* This is our schema for prisma and for the db that lives underneath prisma
* We have our user in here

## Add a new type called `Item`
* DateTime is specific to Prisma
    - If you are not using Prisma you will need to create your own DataTime
* **note** Anytime you make a change to your schema you need to deploy to Prisma because we need to update the db to know and expect both of these items and we also need to pull down our new Prisma schema

`datamodel.prisma`

```
type User {
  id: ID! @unique
  name: String!
  email: String!
}

type Item {
  id: ID! @unique
  title: String!
  description: String!
  image: String
  largeImage: String
  price: Int!
  createAt: DateTime!
  updatedAt: DateTime!
  # user: User!
}
```

* Go to backend

`$ cd backend && npm run deploy`

* That will run: `prisma deploy --env-file .env`
    - That will run `prisma deploy` and point to our `.env` file
* You will see a new item was created with a bunch of new fields

![new fields created for Item](https://i.imgur.com/xL7HHzq.png)

## Open `prisma.graphql`
* This is a generated file based on the data model/schema that we gave it, it goes and creates an enormous GraphQL file that has all of the relationships and all of the types/queries/mutations that are required

`src/generated/prisma.graphql`

```
// MORE CODE

type Item implements Node {
  id: ID!
  title: String!
  description: String!
  image: String
  largeImage: String
  price: Int!
  createAt: DateTime!
  updatedAt: DateTime!
}

// MORE CODE
```

* Above is very similar to how we `extend` Component in React
* Prisma creates a base Node (but it is not in this file)
    - It just has a base (ID and createdAt value)

```
createItem(data: ItemCreateInput!): Item!
```

* And here is this:

```
input ItemCreateInput {
  title: String!
  description: String!
  image: String
  largeImage: String
  price: Int!
  createAt: DateTime!
}
```

* The file gets large becuase of everything that was created
    - Because it generates all the different filters that we want
        + price_gte: Int
        + createAt_not: DateTime
        + createAt_in: [DateTime!]
* **note** GraphQL is not really a query language
    - It doesn't come with anything for filtering or fuzzy matching
    - That's what Prisma does under the hood, it creates a gazillion filters that you'll likely use as you build out your site
    - Deploying will always push it up to Prisma and update the db and that post deploy hook will also pull down an updated version of `prisma.graphql` (the generated GraphQL schema)

## We have our data model, we have our type, let's write our first Mutation
### What are the steps to creating a mutation?
1. Open up the `schema.graphql`
    * Why are there so many GraphQL files?
        - `datamodel.graphql`
            + That's for Prisma (That's for our backend) 
        - `prisma.graphql`
            + That's autogenerated and based off `datamodel.graphql` (This is also Prisma)
        - `schema.graphql`
            + This is our public facing API
                - Prisma has access to everything but our `schema.graphql` is our public facing API and that is what we will be interfacing with, with our JavaScript

### Clean up our `schema.graphql`
`scheam.graphql`

```
type Mutation {
  createItem(title: String, description: String, Price: Int, image: String, largeImage: String): Item!
}

type Query {

}
```

* We return an Item (We haven't created it, and it is **required**)
* If you have too many inputs, you can take them out and make just one input, called `data` and create custom inputs (fyi - that is what Prisma does on the backend)

`prisma.graphql`

```
// MORE CODE

createItem(data: ItemCreateInput!): Item!

// MORE CODE

input ItemCreateInput {
  title: String!
  description: String!
  image: String
  largeImage: String
  price: Int!
  createAt: DateTime!
}

// MORE CODE
```

* This helps us keep it clean
* Enables us to pass in an object as an argument instead of multiple arguments in a function

## We could do this:
`schema.graphql`

```
type Item {
  id: ID! @unique
  title: String!
  description: String!
  image: String
  largeImage: String
  price: Int!
  createAt: DateTime!
  updatedAt: DateTime!
  # user: User!
}

type Mutation {
  createItem(title: String, description: String, Price: Int, image: String, largeImage: String): Item!
}

// MORE CODE
```

* But this is bad because it isn't DRY
* We are duplicating code inside `datamodel.prisma`

### A better way
* If the fields are the say you can just import them
* **note** This is not a standard in GraphQL, there are no standards
* Prisma using something called `GraphQL import` and it does it via a comment (a bit hacky)
    - This will import all our generated GraphQL and make them available to us if we need them
    - And if we don't use them it won't get imported

`schema.graphql`

```
# import * from '/generated/prisma.graphql'

type Mutation {
  createItem(title: String, description: String, Price: Int, image: String, largeImage: String): Item!
}

type Query {

}
```

* Now that will import Item from the generated file `prisma.graphql` and we don't have to define it and just return it `Item!`
    - It will see we are trying to return an `Item` type but since we don't have it, it will know it has to import `Item` from our generated, imported filed `prisma.graphql`
    - This will be great in ruducing lines of duplicated code

## Take it for a test drive
* In Playground
* Make sure you are running your server
* Be in the `backend` folder and run:

`$ npm run dev`

### Houston we have a problem
* Our Query is empty in `schema.graphql` and this is causing the error

```
// MORE CODE

type Query {

}
```

* We fix this before with just `hi` but now let's temporarily have `items` that will return and array of Items

```
// MORE CODE

type Query {
  items: [Item]!
}
```

### Houston we have another problem
* We need to delete our `createCar` mutation in `resolvers`

`resolvers/Mutations.js`

```
const Mutations = {};

module.exports = Mutations;
```

### Houston we have another problem
* We also need to delete the cars Query

`resolvers/Query.js`

```
const Query = {};

module.exports = Query;
```

### All these errors are a good thing
* Because GraphQL is strongly typed, it doesn't let anything slide and this helps us find bugs faster
* Refresh Playground and we now see our documentation is updated correctly
    - QUERIES
        + items:[Item]!
    - MUTATIONS
        + createItem(...): Item!

### Now we need to jump into our Mutation and Query and write our resolvers
* Now we can tap into the great power of Prisma
    - Search `prisma.graphql` for **Mutations** and you will see all the mutations we have access to:

`prisma.graphql`

* All of these methods are now available to us on our backend because that is exactly what we are interfacing with

```
// MORE CODE

type Mutation {
  createUser(data: UserCreateInput!): User!
  createItem(data: ItemCreateInput!): Item!
  updateUser(data: UserUpdateInput!, where: UserWhereUniqueInput!): User
  updateItem(data: ItemUpdateInput!, where: ItemWhereUniqueInput!): Item
  deleteUser(where: UserWhereUniqueInput!): User
  deleteItem(where: ItemWhereUniqueInput!): Item
  upsertUser(where: UserWhereUniqueInput!, create: UserCreateInput!, update: UserUpdateInput!): User!
  upsertItem(where: ItemWhereUniqueInput!, create: ItemCreateInput!, update: ItemUpdateInput!): Item!
  updateManyUsers(data: UserUpdateInput!, where: UserWhereInput): BatchPayload!
  updateManyItems(data: ItemUpdateInput!, where: ItemWhereInput): BatchPayload!
  deleteManyUsers(where: UserWhereInput): BatchPayload!
  deleteManyItems(where: ItemWhereInput): BatchPayload!
}

// MORE CODE
```

### How do we access our DB?
`ctx.db`

* We put our DB on **context** here:

`createServer.js`

```
const { GraphQLServer } = require('graphql-yoga');
const Mutation = require('./resolvers/Mutation');
const Query = require('./resolvers/Query');
const db = require('./db');

// Create the GraphQL Yoga Server

function createServer() {
  return new GraphQLServer({
    typeDefs: 'src/schema.graphql',
    resolvers: {
      Mutation,
      Query,
    },
    resolverValidationOptions: {
      requireResolversForResolveType: false,
    },
    context: req => ({ ...req, db }),
  });
}

module.exports = createServer;
```

* This is how we attached our db to `context`

```
// MORE CODE

context: req => ({ ...req, db }),

// MORE CODE
```

### createItem()
`Mutation.js`

```
const Mutations = {
  createItem(parent, args, ctx, info) {
    // TODO: Check if they are logged in

    const item = ctx.db;
  },
};

module.exports = Mutations;
```

* Another way we could have attached the db is by exporting the db and just importing it into `Mutations.js`, but it is great having access to the db inside of our `context`
    - Then we can call `ctx.db.query` or `ctx.db.mutation` and then we have access to all the different methods inside our generated `prisma.graphql`

`Mutation.js`

```
// MORE CODE

const Mutations = {
  createItem(parent, args, ctx, info) {
    // TODO: Check if they are logged in

    const item = ctx.db.mutation.createItem({
      data: {
        title: args.title,
        description: args.desc,
        price: args.price,
        image: args.image,
        largeImage: args.largeImage,
      },
    });
  },
};

module.exports = Mutations;

// MORE CODE
```

### spread saves typing
* If we use the spread operator we can just pull them all in

```
const Mutations = {
  createItem(parent, args, ctx, info) {
    // TODO: Check if they are logged in

    const item = ctx.db.mutation.createItem({
      data: {
        ...args
      },
    });
  },
};

module.exports = Mutations;
```

* Why don't we just do this?

```
// MORE CODE
const item = ctx.db.mutation.createItem({
  data: args,
});

// MORE CODE
```

* Because we soon will also be assigning a user to the items

## info
* `info` has a lot of stuff inside it but it also has the actual `Query` and `ctx.db.mutation` needs to take the query from our frontend and pass it to our backend and that will specify what data gets returned from the db when we create it
    - This means we will have to (almost always for all of these) is pass the `info` variable as a 2nd argument to our `createItem()`
        + And that will make sure the actual item is returned to use from the db when we've created it

```
const Mutations = {
  createItem(parent, args, ctx, info) {
    // TODO: Check if they are logged in

    const item = ctx.db.mutation.createItem({
      data: {
        ...args
      },
    }, info);
  },
};

module.exports = Mutations;
```

* **note** `ctx.db.mutation.createItem()` returns a Promise
    - So if we want the item to go into the `item` variable we will need to use `async ... await`
        + We need to make `createItem` an **async** method
        + And then we simply **await** the creation of the `item`

```
const Mutations = {
  async createItem(parent, args, ctx, info) {
    // TODO: Check if they are logged in

    const item = await ctx.db.mutation.createItem(
      {
        data: {
          ...args,
        },
      },
      info
    );
  },
};

module.exports = Mutations;
```

### Make sure you `return` the item

```
const Mutations = {
  async createItem(parent, args, ctx, info) {
    // TODO: Check if they are logged in

    const item = await ctx.db.mutation.createItem(
      {
        data: {
          ...args,
        },
      },
      info
    );

    return item; // add this line
  },

};

module.exports = Mutations;
```

### Alternative
* You could just return a Promise like this:

```
const Mutations = {
  return createItem(parent, args, ctx, info) {
    // TODO: Check if they are logged in

    const item = ctx.db.mutation.createItem(
      {
        data: {
          ...args,
        },
      },
      info
    );
    
    return item;
  },

};

module.exports = Mutations;
```

* But it is better to put it in a variable and return the item in most cases because if you have issues debugging it you can just console.log(item) as you create it

```
const Mutations = {
  async createItem(parent, args, ctx, info) {
    // TODO: Check if they are logged in

    const item = await ctx.db.mutation.createItem(
      {
        data: {
          ...args,
        },
      },
      info
    );
  },

  console.log(item);

  return item;
};

module.exports = Mutations;
```

* **tip** Always store your prices in `cents` so you never have to deal with fractions or rounding or decimals
    - So is something is $1.00 make it 100 cents
    - We always deal with whole numbers and never worry about decimals

### Test in Playground
* Type this in and click `prettify`

```
mutation {
  createItem(
    title: "test"
    description: "testing desc"
    image: "test.jpg"
    largeImage: "testlarge.jpg"
    price: 1000
  ) {
    id
    title
  }
}
```

## Houston we have a problem
* We get this error: `Field value.createAt of required type DateTime! was not provided.`
* The `createAt` date should automatically be created by Prisma
* Should be spelled `createdAt`

### Comment them out temporarily
`datamodel.prisma`

```
// MORE CODE

type Item {
  id: ID! @unique
  title: String!
  description: String!
  image: String
  largeImage: String
  price: Int!
  # createdAt: DateTime!
  # updatedAt: DateTime!
  # user: User!
}
```

* **remember** We just modified our datamodel so we need to re-deploy it with:

`$ npm run deploy`

* That will update two fields
* Run again

`$ npm run dev`

* Run our Playground createItem mutation one more time
* We get this output

```
{
  "data": {
    "createItem": {
      "id": "cjnebc0w79xv00b94c9k5pdwm",
      "title": "test"
    }
  }
}
```

## Check if our db was updated
* Every time you pressed play in playground you see a record in your terminal

`$ prisma console`

* Prisma console opens in browser
* You will see 3 items

## Public Playground vs Private Playground
* Public facing Playground

`http://localhost:4444`

* Prisma Playground
  - Click on `Playground` link in Prisma Dashboard
  
`https://us1.prisma.sh/pip-5a52b7/sickophant/dev`

* Open that and you will see everything inside it

## Next - We want to pull our Query
* With Mutations we `pushed` them

## GIT 13
1. Check Status
2. Add to staging
3. Commit with useful commit message
4. Push Branch to Origin
5. Create PR on Origin
6. Code Review on Origin
7. Merge to master branch on Origin (or reject and don't merge)
8. Locally check out of feature branch and into master branch
9. Fetch locally
10. Git Diff to see changes
11. Pull Locally
12. Run and test code
13. Delete local branch


