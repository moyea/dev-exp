# 基于Prisma 2创建一个NodeJS GraphQL server

本节我们将创建一个基于Prisma 2的NodeJS GraphQL Server。开始本节前，请确保已经跟着 [Prisma 2 入门指南](Prisma%202%20入门指南.md) 教程中完整走过一遍。本教程将基于上述教程来完成我们的GraphQL server。

## 目录结构

在确保跟着 [Prisma 2 入门指南](Prisma%202%20入门指南.md)  完成过一遍之后，我们应该有以下目录结构：

```bash
.
├── index.js
├── package.json
├── prisma
│   ├── dev.db
│   ├── migrations
│   │   ├── 20200921035211-init
│   │   │   ├── README.md
│   │   │   ├── schema.prisma
│   │   │   └── steps.json
│   │   └── migrate.lock
│   ├── schema.prisma
│   └── seed.js
└── yarn.lock
```

同时在我们的数据库中，有以下数据：

![prisma-user-table](/Users/ericfeng/github/dev-exp/images/prisma-assets/prisma-user-table.png)

![prisma-todo-table](/Users/ericfeng/github/dev-exp/images/prisma-assets/prisma-todo-table.png)

## 安装依赖

安装GraphQL相关依赖

```bash
$> yarn add graphql-tools graphql-yoga
```

## 创建GraphQL server

在根目录创建 `graphql-server.js` 文件:

```bash
$> touch graphql-server.js
```

打开 `graphql-server.js` , 添加以下代码:

```javascript
const { GraphQLServer } = require('graphql-yoga')

const typeDefs = `
  type Query {
    hello(name: String): String!
  }
`

const resolvers = {
  Query: {
    hello: (_, { name }) => `Hello ${name || 'World'}`,
  },
}

// 使用GraphQLServer创建一个server，需要以下参数:
// typeDefs  - GraphQL类型定义(SDL, Schema Definition Language)或类型定义文件,
//             没有指定schema时，此字段必须
// resolvers - 包含typeDefs中指定字段的解析器，没有指定schema时，此字段必须
// schema    - GraphQLSchema的一个实例，没有typeDefs和resolvers时必须指定此参数
// @example
// const server = new GraphQLServer({ typeDefs, resolvers })
// or
// const server = new GraphQLServer({ schema })
const server = new GraphQLServer({
  typeDefs,
  resolvers,
})

const options = {
  port: 8000,
  endpoint: '/graphql',
  subscriptions: '/subscriptions',
  playground: '/playground',
}

server.start(options, ({ port }) =>
  console.log(
    `Server started, listening on port ${port} for incoming requests.`,
  )
)
```

我们可以测试一下我们的GraphQL server，运行以下命令:

```bash
$> node graphql-server.js
```

之后打开浏览器，访问: http://localhost:8000/playground，我们可以看到这样一个界面:

![graph-playground](/Users/ericfeng/github/dev-exp/images/prisma-assets/graph-playground.png)

在左侧输入以下查询:

```graphql
query{
	hello(name: "John")
}
```

点击运行，我们将会收到一下响应：

```json
{
	"data": {
		"hello": "Hello John"
	}
}
```

## 连接Prisma Client到GraphQL server

打开 `graphql-server.js` ，在顶部添加以下代码:

```javascript
const { PrismaClient } = require('@prisma/client')
```

现在创建一个 `PrismaClient` 的实例，并通过context字段传递给GraphQL server的构造器:

```javascript
const prisma = new PrismaClient()

const server = new GraphQLServer({
  typeDefs,
  resolvers,
  context: {
    prisma,
  }
})
```

我们可以使用 `graphql-tools` 包里的 `makeExecutableSchema` 方法创建一个GraphQLSchema 的实例。

下面使用 graphql-tools 改造下我们的代码:

```javascript
const { makeExecutableSchema } = require('@graphql-tools/schema')
// ...

const schema = makeExecutableSchema({
  typeDefs,
  resolvers,
})

// ...
const server = new GraphQLServer({
  schema,
  context: {
    prisma,
  }
})
// ...
```

## 更新 `typeDefs`

现在我们需要根据我们的数据模型来更新先前定义的 `typeDefs`。同时我们需要添加GraphQL server暴露出去的queries 和 mutations。在queries中，我们需要查询users和todos。在mutations中，我们需要创建users和todos。

使用下面的代码来更新我们的 `typeDefs`:

```javascript
const typeDefs = `
  type User {
    id    :ID!
    name  :String!
    todos :[Todo]
  }

  type Todo {
    id        :ID!
    text      :String!
    completed :Boolean
    userId    :String
    user      :User
  }

  type Query {
    users: [User]
    todos: [Todo]
    user(id: ID!): User
    todo(id:ID!): Todo
  }

  type Mutation {
    createUser(name: String): User
    createTodo(
      text: String!
      name: String
    ): Todo
  }
`
```

## 更新 `resolvers`

我们需要通过传递给`resolvers`的context来与Prisma进行交互，现在更新resolvers对象，以支持 `typeDefs` 中定义的queries和mutations:

### 通过以下代码来实现 creatUser 解析器

```javascript
const resolvers = {
	Query: {
    //...
  },
  Mutation: {
    createUser: (parent, args, context, info) => {
      const user = context.prisma.user.create({
        data: {
          name: args.name
        },
      })
      return user
    }
  }
}
```

在playground中，通过以下代码来测试createUser:

```graphql
mutation {
	createUser(name: 'Tom') {
		id
		name
	}
}
```

### 获取用户

在 `typeDefs` , 我们在queries中定义了通过ID查询一个用户及所有用户，现在来实现这部分的resolvers:

```javascript
const resolvers = {
  Query: {
    user: async (parent, args, context) => {
      return context.prisma.user.findOne({
        where: {
          id: args.id
        },
        include: {
          todos: true
        }
      })
    },
    users: async (parent, args, context) => {
      return context.prisma.user.findMany({
        include: {
          todos: true
        }
      })
    }
  },
  Mutation: {
    // ...
  }
}
```

在playground中，通过以下代码测试:

- 获取单个用户

```graphql
query {
	user(id: "ckfc3lunt0000fx5y2atg2zry") {
		id
		name
		todos {
			id
			text
			completed
		}
	}
}
```

- 获取所有用户

```graphql
query {
	users {
		id
		name
		todos {
			id
			text
			completed
		}
	}
}
```

相应的，我们可以完成 `typeDefs` 中其他queries及mutations。

最终，完整的 `graphql-server.js` 如下:

```javascript
const { PrismaClient } = require('@prisma/client')
const { GraphQLServer } = require('graphql-yoga')
const { makeExecutableSchema } = require('@graphql-tools/schema')

const typeDefs = `
  type User {
    id    :ID!
    name  :String!
    todos :[Todo]
  }

  type Todo {
    id        :ID!
    text      :String!
    completed :Boolean
    userId    :String
    user      :User
  }

  type Query {
    users: [User]
    todos: [Todo]
    user(id: ID!): User
    todo(id:ID!): Todo
  }

  type Mutation {
    createUser(name: String): User
    createTodo(
      text: String!
      name: String
    ): Todo
  }
`

const resolvers = {
  Query: {
    user: async (parent, args, context) => {
      const { id } = args
      return context.prisma.user.findOne({
        where: {
          id,
        },
        include: {
          todos: true,
        },
      })
    },
    users: async (parent, args, context) => {
      return context.prisma.user.findMany({
        include: {
          todos: true,
        },
      })
    },
    todos: async (parent, args, context) => {
      return context.prisma.todo.findMany()
    },
    todo: async (parent, args, context) => {
      const { id } = args
      return context.prisma.todo.findOne({
        where: {
          id,
        },
      })
    },
  },
  Mutation: {
    createUser: async (parent, args, context, info) => {
      const newUser = context.prisma.user.create({
        data: {
          name: args.name,
        },
      })
      return newUser
    },
    createTodo: async (parent, args, context, info) => {
      const todo = context.prisma.todo.create({
        data: {
          text: args.text,
          user: {
            create: {
              name: args.name,
            },
          },
        },
      })
      return todo
    },
  },
}

const schema = makeExecutableSchema({
  typeDefs,
  resolvers,
})

const prisma = new PrismaClient()

const server = new GraphQLServer({
  schema,
  context: {
    prisma,
  },
})

const options = {
  port: 8000,
  endpoint: '/graphql',
  subscriptions: '/subscriptions',
  playground: '/playground',
}

server.start(options, ({ port }) => {
  console.log(
    `Server started, listening on port ${port} for incoming requests.`
  )
})

```

