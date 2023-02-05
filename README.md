## 一、配置环境
1.安装Tailwind
https://tailwindcss.com/docs/guides/nextjs
## 二、设计数据模型，引入Prisma
### 1.安装Prisma
a. `pnpm add -D prisma`
b. `pnpx prisma init` 
此时会看到一个 /prisma 文件夹，里面有一个 schema.prisma 文件，这个文件就是用来定义数据模型的。
另外还会有一个 .env 文件，这个文件是用来配置数据库连接的。(https://github.com/motdotla/dotenv 内部用的这个库，平时也可以使用它，很有用)
c.docker创建postgresql数据实例
`docker build -t linkdb .`
`docker run --name linkdb -p 5433:5432 -v $(pwd)/db_storage:/var/lib/postgresql/data -d linkdb`
d.配置.env文件：`DATABASE_URL="postgresql://admin:admin@localhost:5433/linkdb?schema=public"`
e.将模型写入数据库中：`pnpx prisma migrate dev --name init`
The command does the following things:

Generate a new SQL migration called init
Apply the migration to the database
Install Prisma Client if it's not yet installed
Generate Prisma Client based off the current schema
可以看到prisma文件夹里多了一个migrations文件夹，里面有一个.sql文件，这个文件就是数据库的初始化文件(记录)。
### 2.填充数据
a. 将seed文件放到prisma文件夹下，命名为seed.ts
```TS
import { PrismaClient } from '@prisma/client'
import { links } from '../data/links'

const prisma = new PrismaClient()

async function main() {
  await prisma.user.create({
    data: {
      email: 'test@gamil.com',
      role: 'ADMIN'
    }
  })
  await prisma.link.createMany({data: links})
}

main().catch(e => {
  console.error(e)
  process.exit(1)
}).finally(async () => await prisma.$disconnect())

```
b. 安装ts-node,以作为执行上面文件的依赖
c. 配置ts-node为commonjs的形式执行，因为next.js默认的tsconfig里是以es-next的形式执行的，而prisma的seed文件中依赖的其他包是以commonjs的形式执行的，为了避免不一致，需要配置ts-node为commonjs的形式执行
d.package.json中添加如下配置
```JSON
...
"prisma": {
    "seed": "ts-node --transpile-only ./prisma/seed.ts"
}
...
```
b. `pnpx prisma db seed`

## 三、看数据(Prisma Studio)[https://www.prisma.io/studio]
`pnpx prisma studio`

## 四、GraphQL API
### 和restful对比，GraphQL的优势在于：
1.Rest 开发的一个通用流程：1.要确定方法和路径 2.要确定返回的数据结构 3.要确定请求参数的数据结构
增加了学习曲线、使用麻烦，影响开发效率。后端同学也要管理它，并且需要写接口文档。随着项目的迭代和复杂，接口文档也需要不断更新和复杂。
2.restful api 没有type,导致返回的结果可能出现前端报错
3.overfetching & underfetching

### GraphQL's SDL
定义Schema：
```graphql
type User {
  id: ID
  email: String
  image: String
  role: Role
  # ! is non-nullable
  bookmarks: [Link]!
}

enum Role {
  ADMIN
  USER
}
```
查询：
```graphql
type Query {
  links: [Link]!
  link(id: ID!): Link!
  user(id: ID!): User!
  users: [User]!
}
```
增删改：
···graphql
type Mutation {
  createLink(category: String!, description: String!, imageUrl: String!, title: String!, url: String!): Link!
  deleteLink(id: ID!): Link!
  updateLink(category: String, description: String, id: String, imageUrl: String, title: String, url: String): Link!
}
```
### Build GraphQL
