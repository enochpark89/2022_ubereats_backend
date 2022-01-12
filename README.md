# UberEat Backend

# Development Progress

1. Delete controllers and providers in app.module.ts 
```js
@Module({
  imports: [],
  controllers: [],
  providers: [],
})
export class AppModule {}
```

- To make a simple start, delete below:
        - app.controller.spec.ts
        - app.controller.ts
        - app.service.ts
- As a result, the only left files in a src folder should be main.ts and app.module.ts

*App module is the only module that gets imported to main.ts. Therefore, Everything has to be imported to app module.*

2. Import GraphQL Module into app.module.ts > @Modules({})

*Error: UnhandledPromiseRejectionWarning: Error: Apollo Server requires either an existing schema, modules, or typeDefs*
- Apollo Server requires either schema, modules, or typeDefs.

End Result:
src/app.module.ts
```js
import { Module } from '@nestjs/common';
import { GraphQLModule } from '@nestjs/graphql';

@Module({
  imports: [GraphQLModule.forRoot()],
  controllers: [],
  providers: [],
})
export class AppModule {}
```

# 3. Install typeDefs and resolvers

- typeDefs and resolvers are required to create an apollo server. 

*Please refer to the documentation*
https://www.apollographql.com/docs/

*What is typeDefs?*
- typeDefs: Document(s) that represent your server's GraphQL schema.

*What is resolvers?*
- A map of functions that populate data from individual schema fields.

Two Options in approaches: 
    1. Code first
    2. Schema first

## 1. Code first

- You can create a module and a resolver.

- Resolver can make query inside as below
example 1)
restaurant.resolver.ts
```js
import { Query, Resolver } from '@nestjs/graphql';

@Resolver()
export class RestaurantResolver {
    // Query will return a boolean
  @Query(returns => Boolean)
  isPizzaGood(): Boolean {
    return true;
  }
}
```
- With this, schema gets generated automatically in src/schema.gql.

Reference:

restaurant.module.ts
```js
import { Module } from '@nestjs/common';
import { RestaurantResolver } from './restaurants.resolver';

@Module({
  providers: [RestaurantResolver],
})
export class RestaurantsModule {}
```

app.module.ts
```js
import { Module } from '@nestjs/common';
import { GraphQLModule } from '@nestjs/graphql';
import { join } from 'path';
import { RestaurantsModule } from './restaurants/restaurants.module';

@Module({
  imports: [
    GraphQLModule.forRoot({
      autoSchemaFile: true,
    }),
    RestaurantsModule,
  ],
  controllers: [],
  providers: [],
})
export class AppModule {}
```

Nutshell:

app.module.ts import GraphQLModule > GraphQLModule set up forRoot and turn on autoSchemaFile > RestaurantModule gets imported in GraphQLModule > Restaurant module import Restaurant resolver > Resolver creates a Query and Schema > Schema file gets automatically generated as a result.

# 4. Entities

- Entities is like a model of the database.

1. Create Entities
src/restaurants/entities/restaurant.entity.ts
```js
import { Field, ObjectType } from '@nestjs/graphql';

@ObjectType()
export class Restaurant {
  @Field(type => String)
  name: string;

  @Field(type => Boolean, { nullable: true })
  isGood?: boolean;
}
```

*What are entities?*
- Entity is essentially an object containing fields of data.

2. Import an entity from a resolver. 
```js
import { Query, Resolver } from '@nestjs/graphql';
import { Restaurant } from './entities/restaurant.entity';

@Resolver()
export class RestaurantResolver {
  @Query(returns => Restaurant)
  myRestaurant() {
    return true;
  }
}
```

**How do you check your schema and model?**
localhost(port number:)/graphql - creates a graphql sandbox page that shows a model and a schema.

# 5. Arguments

- NestJS: If you need it you have to ask for it.
- If you want to add arguments in the query, you ask for it.
src/restaurants/restaurants.resolver.ts:
```js
import { Args, Query, Resolver } from '@nestjs/graphql';
import { Restaurant } from './entities/restaurant.entity';

@Resolver(of => Restaurant)
export class RestaurantResolver {
  @Query(returns => [Restaurant])
  restaurants(@Args('veganOnly') veganOnly: boolean): Restaurant[] {
    return [];
  }
}
```

# 6. Mutation

- After importing mutation, you can call it in a resolver.
*What is mutation?*
- Mutation allows us to mutate or to update fields.

DTO: Data Transfer Object

Steps:

1. Create DTOs
src/restaurants/dtos/create-restaurant.dto.ts:
```js
import { ArgsType, Field } from '@nestjs/graphql';

@ArgsType()
export class CreateRestaurantDto {
  @Field(type => String)
  name: string;
  @Field(type => Boolean)
  isVegan: boolean;
  @Field(type => String)
  address: string;
  @Field(type => String)
  ownersName: string;
}
```
2. Create a mutation in a resolver
```js
import { Args, Mutation, Query, Resolver } from '@nestjs/graphql';
import { CreateRestaurantDto } from './dtos/create-restaurant.dto';
import { Restaurant } from './entities/restaurant.entity';

@Resolver(of => Restaurant)
  restaurants(@Args('veganOnly') veganOnly: boolean): Restaurant[] {
    return [];
  }
  @Mutation(returns => Boolean)
  createRestaurant(@Args() createRestaurantDto: CreateRestaurantDto): boolean {
    console.log(createRestaurantDto);
    return true;
  }
}
```
InputType - one object that you pass down.
ARgumentType - allows you to pass seperate values without having to put everything in the object.

# 7. Validating ArgsTypes

Steps:
1. install class-validator and a class-transformer as below
```bash
npm i class-validator
npm i class-transformer
```

2. Create a validation pipeline in main.ts as below
```js
import { ValidationPipe } from '@nestjs/common';
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(new ValidationPipe());
  await app.listen(3000);
}
bootstrap();
```

3. Add restriction on ArgsType()
```js
import { ArgsType, Field } from '@nestjs/graphql';
import { IsBoolean, IsString, Length } from 'class-validator';

@ArgsType()
export class CreateRestaurantDto {
  @Field(type => String)
  @IsString()
  @Length(5, 10)
  name: string;

  @Field(type => Boolean)
  @IsBoolean()
  isVegan: boolean;

  @Field(type => String)
  @IsString()
  address: string;

  @Field(type => String)
  @IsString()
  ownersName: string;
}
```

# 8. TypeORM

- To communicate DB with NestJS, it is good to use TypeORM. You can write your own SQL code, but this will help us mix typescript to NestJS and to have interactivity.

- If you want to set up DB in the front end, you can ddo that. 

- You can also use many of the popular DB such as mysql, postgres, cockroachdb, mariadb, sqlite etc.

- Before you start, 
Widnows:
1. Install PostgreSQL
2. Install pgAdmin

# 9. Window PostSQL and pgAdmin installation

Steps:
1. Make sure PostgreSQL is running
2. Go to pgAdmin and create a new DB called uber-eats


# 10 TypeORM setup

- @nestjs/typeorm - natively made.

- @nestjs/seaulize - good rating!
*3.15 million downloads*

*How do you install TypeORM?*

```js
npm install --save @nestjs/typeorm typeorm pg
```

- Connect from app.module.ts
```js
import { Module } from '@nestjs/common';
import { GraphQLModule } from '@nestjs/graphql';
import { TypeOrmModule } from '@nestjs/typeorm';
import { RestaurantsModule } from './restaurants/restaurants.module';
// If you import TypeOrmModule as below you can connect to the db.
@Module({
  imports: [
    RestaurantsModule,
    TypeOrmModule.forRoot({
      type: 'postgres',
      host: 'localhost',
      port: 5432,
      username: 'username',
      password: 'passwords123'
      database: 'uber-eats',
      synchronize: true,
      logging: true,
    }),
    GraphQLModule.forRoot({
      autoSchemaFile: true,
    }),
  ],
  controllers: [],
  providers: [],
})
export class AppModule {}
```

# 11. Putting sensitive information to .env.

- you can use .dotenv to hide the sensitive information. 
- NestJS way also have configuration module.

*How do you use a configuration module?*
```bash
npm i --save @nestjs/config
```

Steps:

1. Import ConfigModule from the app.module.ts

- Make this global so that this can be access everywhere. 

src/app.module.ts
```js
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import { GraphQLModule } from '@nestjs/graphql';
import { TypeOrmModule } from '@nestjs/typeorm';
import { RestaurantsModule } from './restaurants/restaurants.module';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      envFilePath: process.env.NODE_ENV === 'dev' ? '.env.dev' : '.env.test',
    }),
    TypeOrmModule.forRoot({
      type: 'postgres',
      host: 'localhost',
    GraphQLModule.forRoot({
      autoSchemaFile: true,
    }),
    RestaurantsModule,
  ],
  controllers: [],
  providers: [],
})
export class AppModule {}
```

2. Create three environment variables 
- dev
- env.dev
- .env.test

3. Install cross-env
- With cross-env package, you can alternate between different .env files.
*How do you install cross-env?*
```bash
npm i cross-env
```

4. Add the 'cross-env ENV=dev ' to the package.json script
```js
    "start:dev": "cross-env ENV=dev nest start --watch",
```