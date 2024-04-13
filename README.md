TOC

1. [Setup](#1-setup)
2. [Resource Setup for user](#2-resource-setup-for-user)
    - [Basic Setup](#basic-setup-)
    - [Test in GraphQL Playground](#test-in-graphql-playground)
3. [Resource Setup for post](#3-resource-setup-for-post)
    - [Basic Setup](#basic-setup)
    - [Test in GraphQL Playground](#test-in-graphql-playground-1)


# 1. Setup

- `nest new gateway`
```bash
⚡  We will scaffold your app in a few seconds..

? Which package manager would you ❤️  to use? pnpm
```
-  `cd gateway`


- `nest g app`
```bash
? What name would you like to use for the app? users
```
- Move to the main folder

- `nest g app`
```bash
? What name would you like to use for the app? post
```

## Dependency

`pnpm i @apollo/gateway @apollo/subgraph @nestjs/apollo @nestjs/graphql apollo-server-express graphql`


# 2. Resource Setup for user


- `nest g resource`

```bash
? Which project would you like to generate to? users
? What name would you like to use for this resource (plural, e.g., "users")? users
? What transport layer do you use? GraphQL (code first)
? Would you like to generate CRUD entry points? Yes
```

- `Users` > `Users`
> - copy all and paste in the src folder and replace all
> - delete users.controller.ts also test file

## Basic Setup 

- user.entity.ts
```ts
import { ObjectType, Field, ID, Directive } from '@nestjs/graphql';

@ObjectType()
@Directive('@key(fields: "id")')
export class User {
  @Field(() => ID)
  id: string;

  @Field()
  email: string;

  @Field()
  password: string;
}
```

- dto -> create-user.input.ts
```ts
import { InputType, Field } from '@nestjs/graphql';

@InputType()
export class CreateUserInput {
  @Field()
  id: string;

  @Field()
  email: string;

  @Field()
  password: string;
}
```


- `user.module.ts`
```ts
imports: [
    GraphQLModule.forRoot<ApolloFederationDriverConfig>({
        driver: ApolloFederationDriver,
        autoSchemaFile: {
            federation: 2,
        },
    }),
]
```


- `User.service.ts`
```ts
@Injectable()
export class UsersService {
  private readonly users: User[] = [];

  create(createUserInput: CreateUserInput) {
    this.users.push(createUserInput);
    return createUserInput;
  }

  findAll() {
    return this.users;
  }

  findOne(id: string) {
    return this.users.find((user) => user.id === id);
  }
}
```
- `User.resolver.ts`
```ts
@Resolver(() => User)
export class UsersResolver {
  constructor(private readonly usersService: UsersService) {}

  @Mutation(() => User)
  createUser(@Args('createUserInput') createUserInput: CreateUserInput) {
    return this.usersService.create(createUserInput);
  }

  @Query(() => [User], { name: 'users' })
  findAll() {
    return this.usersService.findAll();
  }

  @Query(() => User, { name: 'user' })
  findOne(@Args('id') id: string) {
    return this.usersService.findOne(id);
  }

}
```
## Test in GraphQL Playground

Go to - http://localhost:3000/graphql

### Write the mutation query

```graphql
mutation {
    createUser(
        createUserInput: { id: "123", email: "test@example.com", password: "test" }
) {
        id
        email
        password
    }
}
```
- Create a user by running the provided mutation

### Write the query to fetch by id

```graphql
query {
    user(id: "123") {
        id
        email
        password
    }
}

```
- Fetch users by running the provided queries

# 3. Resource Setup for post

- `nest g resource`
```bash
? Which project would you like to generate to? post
? What name would you like to use for this resource (plural, e.g., "users")? posts
? What transport layer do you use? GraphQL (code first)
? Would you like to generate CRUD entry points? Yes
```

- `Posts` > `Posts`
> - copy all and paste in the src folder and replace all
> - delete posts.controller.ts also test file

## Basic Setup

- post.entity.ts
```ts
import { ObjectType, Field, ID } from '@nestjs/graphql';
@ObjectType()
export class Post {
    @Field(() => ID)
    id: string;

    @Field()
    body: string;

    @Field()
    authorId: string;
}
```

- dto -> create-post.input.ts
```ts
import { InputType, Field } from '@nestjs/graphql';

@InputType()
export class CreatePostInput {
    @Field()
    id: string;

    @Field()
    body: string;

    @Field()
    authorId: string;
}
```


- `post.module.ts`
```ts
imports: [
    GraphQLModule.forRoot<ApolloFederationDriverConfig>({
        driver: ApolloFederationDriver,
        autoSchemaFile: {
            federation: 2,
        },
    }),
]
```


- `Post.service.ts`
```ts
@Injectable()
export class PostsService {
    private readonly posts: Post[] = [];

    create(createPostInput: CreatePostInput) {
        this.posts.push(createPostInput);
        return createPostInput;
    }

    findAll() {
        return this.posts;
    }

    findOne(id: string) {
        return this.posts.find((post:Post) => post.id === id);
    }
}
```
- `Post.resolver.ts`
```ts
@Resolver(() => Post)
export class PostsResolver {
    constructor(private readonly postsService: PostsService) {}

    @Mutation(() => Post)
    createPost(@Args('createPostInput') createPostInput: CreatePostInput) {
        return this.postsService.create(createPostInput);
    }

    @Query(() => [Post], { name: 'posts' })
    findAll() {
        return this.postsService.findAll();
    }

    @Query(() => Post, { name: 'post' })
    findOne(@Args('id') id: string) {
        return this.postsService.findOne(id);
    }
}
```

## Test in GraphQL Playground

Go to - http://localhost:3001/graphql

### Write the mutation query

```graphql
mutation {
    createPost(
        createPostInput: { authorId: "123", id: "234", body: "Text of the post" }
    ) {
        id
        authorId
        body
    }
}
```
- Create a user by running the provided mutation

### Write the query to fetch by id

```graphql
query {
    post(id: "234") {
        id
        authorId

        body
    }
}
```

- Fetch users by id running the provided queries

### Write the query to fetch all

```graphql
query {
    posts{
        id
        authorId
        body
    }
}
```

- Fetch users running the provided queries


# 3. Connect Gateway


In your `gateway` -> `app.module.ts`

```ts
imports: [
   GraphQLModule.forRoot<ApolloGatewayDriverConfig>({
      driver: ApolloGatewayDriver,
      gateway: {
         supergraphSdl: new IntrospectAndCompose({
            subgraphs: [
               {
                  name: 'users',
                  url: 'http://localhost:3000/graphql', //this one for user server
               },
               {
                  name: 'posts',
                  url: 'http://localhost:3001/graphql',//this one for post server
               },
            ],
         }),
      },
   }),
]
```

in `main.ts`

```ts
const app = await NestFactory.create(AppModule, { cors: true });
await app.listen(3002);//gateway server
```

## Test in GraphQL Playground

Go to - http://localhost:3000/graphql

Now you can perform query and mutation for both post and user 

Note: Start both user and post-server also 
> `pnpm start:dev post` and `pnpm start:dev user`
