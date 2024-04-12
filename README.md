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
DELETE src
DELETE test
CREATE apps/gateway/tsconfig.app.json (231 bytes)
CREATE apps/gateway/src/app.controller.spec.ts (639 bytes)
CREATE apps/gateway/src/app.controller.ts (286 bytes)
CREATE apps/gateway/src/app.module.ts (259 bytes)
CREATE apps/gateway/src/app.service.ts (150 bytes)
CREATE apps/gateway/src/main.ts (216 bytes)
CREATE apps/gateway/test/app.e2e-spec.ts (654 bytes)
CREATE apps/gateway/test/jest-e2e.json (192 bytes)
CREATE apps/users/tsconfig.app.json (229 bytes)
CREATE apps/users/src/main.ts (222 bytes)
CREATE apps/users/src/users.controller.ts (298 bytes)
CREATE apps/users/src/users.controller.spec.ts (665 bytes)
CREATE apps/users/src/users.module.ts (273 bytes)
CREATE apps/users/src/users.service.ts (152 bytes)
CREATE apps/users/test/jest-e2e.json (192 bytes)
CREATE apps/users/test/app.e2e-spec.ts (662 bytes)
UPDATE tsconfig.json (562 bytes)
UPDATE package.json (2015 bytes)
UPDATE nest-cli.json (800 bytes)
```
- Move to the main folder

- `nest g app`
```bash
? What name would you like to use for the app? post
CREATE apps/post/tsconfig.app.json (228 bytes)
CREATE apps/post/src/main.ts (219 bytes)
CREATE apps/post/src/post.controller.ts (292 bytes)
CREATE apps/post/src/post.controller.spec.ts (652 bytes)
CREATE apps/post/src/post.module.ts (266 bytes)
CREATE apps/post/src/post.service.ts (151 bytes)
CREATE apps/post/test/jest-e2e.json (192 bytes)
CREATE apps/post/test/app.e2e-spec.ts (658 bytes)
UPDATE tsconfig.json (562 bytes)
UPDATE package.json (2013 bytes)
UPDATE nest-cli.json (1030 bytes)
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
CREATE apps/users/src/users/users.module.ts (233 bytes)
CREATE apps/users/src/users/users.resolver.ts (1144 bytes)
CREATE apps/users/src/users/users.resolver.spec.ts (544 bytes)
CREATE apps/users/src/users/users.service.ts (651 bytes)
CREATE apps/users/src/users/users.service.spec.ts (471 bytes)
CREATE apps/users/src/users/dto/create-user.input.ts (203 bytes)
CREATE apps/users/src/users/dto/update-user.input.ts (251 bytes)
CREATE apps/users/src/users/entities/user.entity.ts (194 bytes)
UPDATE apps/users/src/users.module.ts (336 bytes)
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
CREATE apps/post/src/posts/posts.module.ts (233 bytes)
CREATE apps/post/src/posts/posts.resolver.ts (1144 bytes)
CREATE apps/post/src/posts/posts.resolver.spec.ts (544 bytes)
CREATE apps/post/src/posts/posts.service.ts (651 bytes)
CREATE apps/post/src/posts/posts.service.spec.ts (471 bytes)
CREATE apps/post/src/posts/dto/create-post.input.ts (203 bytes)
CREATE apps/post/src/posts/dto/update-post.input.ts (251 bytes)
CREATE apps/post/src/posts/entities/post.entity.ts (194 bytes)
UPDATE apps/post/src/post.module.ts (329 bytes)
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
