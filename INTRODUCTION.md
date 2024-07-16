# NestJS

## Introduction

- Nest (NestJS) is a framework for building efficient, scalable Node.js server-side applications. It uses progressive
  JavaScript, is built with and fully supports TypeScript (yet still enables developers to code in pure JavaScript) and
  combines elements of OOP (Object-Oriented Programming), FP (Functional Programming), and FRP (Functional Reactive
  Programming).
- Under the hood, Nest makes use of robust HTTP Server frameworks like Express (the default) and optionally can be
  configured to use Fastify as well!
- However, while plenty of superb libraries, helpers, and tools exist for Node (and server-side JavaScript), none of
  them effectively solve the main problem of - **Architecture**.
- Nest provides an out-of-the-box application architecture which allows developers and teams to create highly testable,
  scalable, loosely coupled, and easily maintainable applications. The architecture is heavily inspired by _Angular_.

## First steps

### Prerequisites

- Please make sure that Node.js (version >= 16) is installed on your operating system.

### Setup

- Setting up a new project is quite simple with the Nest CLI. With npm installed, you can create a new Nest project with
  the following commands in your OS terminal:

```shell
$ npm i -g @nestjs/cli
$ nest new project-name
```

- The project-name directory will be created, node modules and a few other boilerplate files will be installed, and a
  src/ directory will be created and populated with several core files.
- The ``main.ts`` includes an ``async`` function, which will bootstrap our application:

```typescript

import {NestFactory} from '@nestjs/core';
import {AppModule} from './app.module';

async function bootstrap() {
    const app = await NestFactory.create(AppModule);
    await app.listen(3000);
}

bootstrap();

```

- To create a Nest application instance, we use the core NestFactory class. ``NestFactory`` exposes a few static methods
  that allow creating an application instance. The ``create()`` method returns an application object, which fulfills the
  ``INestApplication`` interface.
- Technically, Nest is able to work with any Node HTTP framework once an adapter is created. There are two HTTP
  platforms supported out-of-the-box: ``express`` and ``fastify``. You can choose the one that best suits your needs.
- Whichever platform is used, it exposes its own application interface. These are seen respectively as
  ``NestExpressApplication`` and ``NestFastifyApplication``.
- When you pass a type to the ``NestFactory.create()`` method, as in the example below, the app object will have methods
  available exclusively for that specific platform. Note, however, you don't need to specify a type unless you actually
  want to access the underlying platform API.

```typescript

const app = await NestFactory.create<NestExpressApplication>(AppModule);

```

### Linting and formatting

- CLI provides the best effort to scaffold a reliable development workflow at scale. Thus, a generated Nest project
  comes with both a code linter and formatter preinstalled (respectively eslint and prettier).

### Next Section

[Controllers](CONTROLLERS.md)