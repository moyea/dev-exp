# Prisma 2 入门指南

## Prisma 2 简介

Prisma 2是一个开源的数据工具，代替了传统的ORM，提供一种数据层抽象，允许我们使用Javascript方法和对象编写数据库查询，简化了数据库操作。

Prisma 2将以我们选择的语言编写相应的查询映射到相应的数据库中。当前，仅支持MySQL, SQLite和PostgreSQL。

简言之，**Prisma 2 是一个 NodeJS 的数据库框架**。

## 什么是Prisma 2?

Prisma 2由以下三部分组成:

1. Prisma Client JS: 类型安全且自动生成的查询构建器(Query Builder)

2. Prisma Migrate: 一种可以更改数据库架构的工具，例如创建新表或向已有的表中添加列。

3. Prisma Studio: 数据库的可视化编辑器。

### 环境需求

为了成功运行本指南，需要确保以下环境：

- Node.js
- 本机支持 SQlite

### 初始化项目

1. 创建项目目录并进入该目录:

```bash
$> mkdir hello-prisma && cd hello-prisma
```

2. 初始化Node.js项目，并将Prisma CLI加入到`devDependencies`中

```bash
$> npm init -y
$> yarn add -D @prisma/cli
```

3. 现在通过 `npx prisma` 调用 Prisma CLI 来创建 Prisma Schema 文件

```bash
$> npx prisma init
```

该命令会创建一个 `prisma` 目录，包含以下两个文件:

- `schema.prisma`: 具有数据库连接的Prisma Schema和Prisma Client生成器
- `.env`: 用于定义环境变量的dotenv文件(用于数据库连接)

### 目录结构

```
.
├── package.json
├── prisma
│   ├── .env
│   └── schema.prisma
└── yarn.lock
```

### 连接数据库

要连接数据库，需要将schema.prisma中 `datasource` 块中 `url` 设置为数据库连接URL:

`prisma/schema.prisma`

```prisma
// 这是你的Prisma schema文件，
// 通过 https://www.prisma.io/docs/reference/tools-and-interfaces/prisma-schema 了解更多

// 通过`generator`块, 你明确指定了你想创建Prisma's DB client.
// 通过运行`prisma generate`命令将会在`node_modules/@prisma`目录创建Client，
// 且可以被类似 import { Prisma Client } from '@prisma/client' 代码导入
generator client {
	provider = "prisma-client-js"
}

// `database` 块用于指定数据库连接, 通过
// https://www.prisma.io/docs/reference/tools-and-interfaces/prisma-schema/data-sources
// 了解更多
datasource db {
	// `provider`项来指定你的数据库是`postgreSQL`, `MySQL` 或`SQLite`
	provider = "sqlite"
	// `url`用于指定你数据库连接, 这里是使用`prisma/.env`中的环境变量
	url      = env("DATABASE_URL")
}
```

这种情况下，`url` 是通过在`prisma/.env`中环境变量设置的

`prisma/.env`

```
# 指定数据源
DATABASE_URL="sqlite:./dev.db"
# 指定其他数据源的例子:
# PostgreSQL:
# DATABASE_URL="postgresql://johndoe:johndoe@localhost:5432/mydb?schema=public"
# MySQL:
# DATABASE_URL="mysql://johndoe:johndoe@localhost:3306/mydb"
```

### 创建数据模型

打开`prisma/schema.prisma` 文件，添加下面的内容:

```prisma
model User {
	id   String @default(cuid()) @id
	name String 
	todos Todo[]
}

model Todo {
	id 	 			String @default(cuid()) @id
	text 			String
	completed Boolean @default(false)
	userId    String?
	user			User? @relation(fields:[userId], references: [id])
}
```

这样我们就定义了User和Todo两个模型，下一步就是将模型写入到数据库。

### 创建SQLite数据库

将模型映射到数据库，需要借助**Prisma Migrate**工具:

```
$> prisma migrate save --experimental
$> prisma migrate up --experimental
```

`prisma migrate save --experimental` 保存一个新的migration到根目录下`prisma/migrations`目录中并更新数据库中`_Migration`表。每次运行该命令完成都会生成一个带有详细信息的`README.md`的新的migration。

首次运行该命令时，将会创建一个数据库，当你选择了`Yes`之后，你需要为你的migration添加一个名字，同时你会注意到一个migrations目录被创建。

之后，我们需要运行 `prisma migrate up --experimental` 来真正执行创建的migrations。

### 创建Prisma Client

到目前为止，我们已经可以开始创建我们的Prisma Client。通过以下任意一个命令:

```bash
$> yarn add @prisma/client


$> prisma generate
```

注意：`yarn add @prisma/client` 同样是运行 `prisma generate` 来构建 Prisma Client到`node_modules/@prisma/client` 目录。

### 创建具有初始值的数据库

在`prisma`目录下创建一个名为`seed.js`的文件:

```bash
$> touch prisma/seed.js
```

之后，代开`seed.js` 文件，并导入 Prisma Client:

```javascript
const { PrismaClient } = require('@prisma/client')

// 创建一个PrismaClient实例
const prisma = new PrismaClient()

const main = async () => {
	// ... 此处可以写Prisma Client的查询代码
}

main()
  .catch(console.error)
  .finally(async () => {
  	// 关闭prisma的链接
    await prisma.$disconnect()
  })

```

现在，我们可以再`main`函数中，添加以下代码：

```javascript
// 创建名为Sasha的用户
const sasha = await prisma.user.create({
  data: {
    name: 'Sasha',
  },
})

console.log(sasha)

// 创建名为Johnny的用户，以及两条todo数据
const johnny = await prisma.user.create({
  data: {
    name: 'Johnny',
    todos: {
      create: [{ text: 'Do dishes' }, { text: 'Walk the dog' }],
    },
  },
})

console.log(johnny)

// 创建一条TODO 
const run = await prisma.todo.create({
  data: {
    text: 'Run a full marathon',
  },
})

console.log(run)

// 创建一条TODO并创建关联的用户Amelia
const grocery = await prisma.todo.create({
  data: {
    text: 'Buy groceries for the work',
    User: {
      create: {
        name: 'Amelia',
      },
    },
  },
})

console.log(grocery)
```

最后，执行以下命令：

```bash
$> node prisma/seed.js
```

这样，我们就可以在数据库中插入了一些默认数据。

## 通过Prisma Studio查看数据

Prisma Studio允许通过一个管理员UI界面来查看我们的数据，同时也可以对我们的数据执行CRUD(create, read, update, delete)操作。

启动Prisma Studio，通过运行以下命令:

```bash
$> prisma studio
```

现在打开: http://localhost:5555 就可以查看到一个管理员UI界面了。

## 执行一些查询操作

在根目录下创建一个`index.js` :

```bash
$> touch index.js
```

插入以下代码：

```javascript
const { PrismaClient } = require("@prisma/client")

const prisma = new PrismaClient()

const main = async () => {
  // 查找所有用户
  const users = await prisma.users.findMany()
  
  console.log(users)
  
  // 查找所有todos有数据的用户
  const usersWithTodos = await prisma.user.findMany({
    where: {
      todos: {
        some: {
          NOT: undefined,
        },
      },
    },
  })

  // 格式化输出有todo数据的用户
  console.log(JSON.stringify(usersWithTodos, null, 2))
}

main()
  .catch(e => console.error(e))
  .finally(async () => {
    await prisma.disconnect()
  })
```

