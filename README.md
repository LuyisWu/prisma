# Prisma 2.0

[Quickstart](https://www.prisma.io/docs/getting-started/quickstart) • [Website](https://www.prisma.io) • [Docs](https://www.prisma.io/docs/) • [Examples](https://github.com/prisma/prisma-examples/) • [Blog](https://www.prisma.io/blog) • [Slack](https://slack.prisma.io/) • [Twitter](https://twitter.com/prisma) • [Demo videos](https://www.youtube.com/watch?v=t0rYb6pGLKQ&list=PLn2e1F9Rfr6k9PnR_figWOcSHgc_erDr5&index=2&t=0s)

## What is Prisma?

Prisma is a **database toolkit** that consists of these tools:

- [**Prisma Client**](https://github.com/prisma/prisma-client-js): Type-safe and auto-generated query builder for Node.js and TypeScript
- [**Prisma Migrate**](https://github.com/prisma/migrate): Declarative data modeling and migrations
- [**Studio**](https://github.com/prisma/studio): Admin UI to support various database workflows

## Getting started

The fastest way to get started with Prisma is by following the [**Quickstart (5 min)**](https://www.prisma.io/docs/getting-started/quickstart).

The Quicksart is based on SQLite. You can also get started with your own database (PostgreSQL and MySQL) by following one of these guides:

- [Add Prisma to an existing project](https://www.prisma.io/docs/getting-started/setup-prisma/add-to-existing-project)
- [Setup a new project with Prisma from scratch (SQL migrations + introspection)](https://www.prisma.io/docs/getting-started/setup-prisma/add-to-existing-project)
- [Setup a new project with Prisma from scratch (Prisma Migrate)](https://www.prisma.io/docs/getting-started/setup-prisma/add-to-existing-project)

## How does Prisma work?

### The Prisma schema

Every project that use a tool from the Prisma toolkit starts with a [Prisma schema file](https://www.prisma.io/docs/reference/tools-and-interfaces/prisma-schema/prisma-schema-file). The Prisma schema allows developers to define their _application models_ in an intuitive data modeling language. It also contains the connection to a database and defines a _generator_:

```prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

model Post {
  id        Int     @id @default(autoincrement())
  title     String
  content   String?
  published Boolean @default(false)
  author    User?   @relation(fields:  [authorId], references: [id])
  authorId  Int?
}

model User {
  id    Int     @id @default(autoincrement())
  email String  @unique
  name  String?
  posts Post[]
}
```

In this schema, you configure three things:

- **Data source**: Specifies your database connection (via an environment variable)
- **Generator**: Indicates that you want to genenerate Prisma Client
- **Data model**: Defines your application models

### The Prisma data model

On this page, the focus is on the data model. You can learn more about [Data sources](https://www.prisma.io/docs/reference/tools-and-interfaces/prisma-schema/data-sources) and [Generators](https://www.prisma.io/docs/reference/tools-and-interfaces/prisma-schema/generators) on the respective docs pages.

#### Functions of Prisma models

The data model is a collection of [models](https://www.prisma.io/docs/reference/tools-and-interfaces/prisma-schema/models). A model has two major functions:

- Represent a table in the underlying database
- Provide the foundation for the queries in the Prisma Client API

#### Getting a data model

There are two major workflows for "getting" a data model into your Prisma schema:

- Generate the data model from [introspecting](https://www.prisma.io/docs/reference/tools-and-interfaces/introspection) a database
- Manually writing the data model and mapping it to the database with [Prisma Migrate](https://www.prisma.io/docs/reference/tools-and-interfaces/prisma-migrate)

Once the data model is defined, you can [generate Prisma Client](https://www.prisma.io/docs/reference/tools-and-interfaces/prisma-client/generating-prisma-client) which will expose CRUD and more queries for the defined models. If you're using TypeScript, you'll get full type-safety for all queries (even when only retrieving the subsets of a model's fields). 

### Accessing your database with Prisma Client

#### Generating Prisma Client

The first step when using Prisma Client is installing its npm package:

```
npm install @prisma/client
```

Note that the installation of this package invokes the `prisma generate` command which reads your Prisma schema and _generates_ the Prisma Client code. The code will be located in `node_modules/@prisma/client`. 

After you change your data model, you'll need to manually re-generate Prisma Client to ensure the code inside `node_modules/@prisma/client` get updated:

```
prisma generate
```

Note that cecause the Prisma Client node module contains specific context about _your_ Prisma schema, it's sometimes referred to as a ["smart node module"](https://www.prisma.io/docs/reference/tools-and-interfaces/prisma-client/generating-prisma-client#how-prisma-client-compares-to-conventional-node-modules).

#### Using Prisma Client to send queries to your database 

Once Prisma Client was generated, you can import in your code and send queries to your database. This is what the setup code looks like.

##### Import and instantiate Prisma Client

You can import and instantiate Prisma Client as follows:

```ts
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()
```

or

```js
const { PrismaClient } = require('@prisma/client')

const prisma = new PrismaClient()
```

Now you can start sending queries via the generated Prisma Client API, here are a few sample queries. Note that all Prisma Client queries return _plain old JavaScript objects_.

Learn more about the available operations in the [Prisma Client API reference](https://www.prisma.io/docs/reference/tools-and-interfaces/prisma-client/api) or watch this [demo video](https://www.youtube.com/watch?v=Il6-smKc77w&list=PLn2e1F9Rfr6k9PnR_figWOcSHgc_erDr5&index=4) (2 min).

##### Retrieve all `User` records from the database

```ts
// Run inside `async` function
const allUsers = await prisma.user.findMany()
```

##### Include the `posts` relation on each returned `User` object

```ts
// Run inside `async` function
const allUsers = await prisma.user.findMany({
  include: { posts: true },
})
```

##### Filter all `Post` records that contain `"prisma"`

```ts
// Run inside `async` function
const filteredPosts = await prisma.post.findMany({
  where: {
    OR: [
      { title: { contains: "prisma" } },
      { content: { contains: "prisma" } },
    ],
  },
})
```

##### Create a new `User` and a new `Post` record in the same query

```ts
// Run inside `async` function
const user = await prisma.user.create({
  data: {
    name: "Alice",
    email: "alice@prisma.io",
    posts: {
      create: { title: "Join us for Prisma Day 2020" },
    },
  },
})
```

##### Update an existing `Post` record

```ts
// Run inside `async` function
const post = await prisma.post.update({
  where: { id: 42 },
  data: { published: true },
})
```

#### Usage with TypeScript

Note that when using TypeScript, the result of this query will be _statically typed_ so that you can't accidentally access a property that doesn't exist (and any typos are caught at compile-time). Learn more about leveraging Prisma Client's generated types on the [Advanced usage of generated types](https://www.prisma.io/docs/reference/tools-and-interfaces/prisma-client/advanced-usage-of-generated-types) page in the docs.

## Community

Prisma has a large and supportive [community](https://www.prisma.io/community) of enthusiastic application developers. You can join us on [Slack](https://slack.prisma.io) and here on [GitHub](https://github.com/prisma/prisma2/discussions).

## Support

### Ask a question about Prisma

You can ask questions and initiate [discussions](https://github.com/prisma/prisma2/discussions/) about Prisma-related topics in the `prisma` repository on GitHub.

[**Ask a question**](https://github.com/prisma/prisma2/discussions/new)

### Create a bug report for Prisma

If you see an error message or run into an issue, please make sure to create a bug report! You can find [best practices for creating bug reports](./more/creating-bug-reports) (like including additional debugging output) in these docs.

[**Create bug report**](https://github.com/prisma/prisma/issues/new?assignees=&labels=&template=bug_report.md&title=)


### Submit a feature request

If Prisma currently doesn't have a certain, be sure to check out the [roadmap](./more/roadmap) to see if this is already planned for the future.

If the feature on the roadmap is linked to a GitHub issue, please make sure to leave a +1 on the issue and ideally a comment with your thoughts about the feature!

[**Submit feature request**](https://github.com/prisma/prisma2/issues/new?assignees=&labels=&template=feature_request.md&title=)

## Contributing

Read more about how to contribute to Prisma [here](https://github.com/prisma/prisma2/blob/master/CONTRIBUTING.md)

## Build Status

[![Build status](https://badge.buildkite.com/590e1981074b70961362481ad8319a831b44a38c5d468d6408.svg)](https://buildkite.com/prisma/prisma2-test)

 [![Actions Status](https://github.com/prisma/prisma2-e2e-tests/workflows/test/badge.svg)](https://github.com/prisma/prisma2-e2e-tests/actions)
