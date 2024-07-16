## Controllers

- Controllers are responsible for handling incoming requests and returning responses to the client.
- The **routing** mechanism controls which controller receives which requests. Frequently, each controller has more than
  one route, and different routes can perform different actions.
- In order to create a basic controller, we use classes and decorators. **Decorators** associate classes with required
  metadata and enable Nest to create a routing map (tie requests to the corresponding controllers).

### Routing

- In the following example we'll use the ``@Controller()`` decorator, which is required to define a basic controller.
  We'll specify an optional route path prefix of ``cats``.

```typescript

import {Controller, Get} from '@nestjs/common';

@Controller('cats')
export class CatsController {
    @Get()
    findAll(): string {
        return 'This action returns all cats';
    }
}

```

- **What is the route path?** The route path for a handler is determined by concatenating the (optional) prefix declared
  for the controller, and any path specified in the method's decorator. Since we've declared a prefix for every route (
  cats), and haven't added any path information in the decorator, Nest will map GET ``/cats`` requests to this handler.
- This method will return a ``200`` status code and the associated response, which in this case is just a string. Why
  does
  that happen? To explain, we'll first introduce the concept that Nest employs two different options for manipulating
  responses:

| Option           | Details                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
|------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Standard         | Using this built-in method, when a request handler returns a JavaScript object or array, it will automatically be serialized to JSON. When it returns a JavaScript primitive type (e.g., string, number, boolean), however, Nest will send just the value without attempting to serialize it. This makes response handling simple: just return the value, and Nest takes care of the rest. Furthermore, the response's status code is always ``200`` by default, except for ``POST`` requests which use ``201``. We can easily change this behavior by adding the`` @HttpCode(...)`` decorator at a handler-level (see Status codes). |
| Library-specific | We can use the library-specific (e.g., Express) response object, which can be injected using the ``@Res()`` decorator in the method handler signature (e.g., ``findAll(@Res() response)``). With this approach, you have the ability to use the native response handling methods exposed by that object. For example, with Express, you can construct responses using code like ``response.status(200).send()``.                                                                                                                                                                                                                      |

#### Resources

- It's that simple. Nest provides decorators for all the standard HTTP
  methods: ``@Get()``, ``@Post()``, `` @Put()``, ``@Delete()``, ``@Patch()``, ``@Options()``, and ``@Head()``. In
  addition, ``@All()`` defines an endpoint that handles all of them.

#### Route wildcards

- Pattern based routes are supported as well. For instance, the asterisk is used as a wildcard, and will match any
  combination of characters.

```typescript

import {Controller, Get} from '@nestjs/common';

@Controller('ab*cd')
export class CatsController {
    @Get()
    findAll(): string {
        return 'This action returns all cats';
    }
}

```

- The 'ab*cd' route path will match abcd, ab_cd, abecd, and so on. The characters ?, +, *, and () may be used in a route
  path, and are subsets of their regular expression counterparts. The hyphen ( -) and the dot (.) are interpreted
  literally by string-based paths.

### Req Object

- We can access the request object by instructing Nest to inject it by adding the ``@Req()`` decorator to the handler's
  signature.

```typescript

import {Controller, Get, Req} from '@nestjs/common';
import {Request} from 'express';

@Controller('cats')
export class CatsController {
    @Get()
    findAll(@Req() request: Request): string {
        return 'This action returns all cats';
    }
}

```

- Below is a list of the provided decorators and the plain platform-specific objects they represent.

| Decorator               | Platform-specific object(s)     |
|-------------------------|---------------------------------|
| @Request(), @Req()      | req                             |
| @Response(), @Res()     | res                             |
| @Next()                 | next                            |
| @Session()              | req.session                     |
| @Param(key?: string)    | req.params / req.params[key]    |
| @Body(key?: string)     | req.body / req.body[key]        |
| @Query(key?: string)    | req.query / req.query[key]      |
| @Headers(name?: string) | req.headers / req.headers[name] |
| @Ip()                   | req.ip                          |
| @HostParam()            | req.hosts                       |

- Note that when you inject either ``@Res()`` or ``@Response()`` in a method handler, you put Nest into *
  *Library-specific mode** for that handler, and you become responsible for managing the response. When doing so, you
  must issue some kind of response by making a call on the response object (e.g., ``res.json(...)``
  or ``res.send(...)``), or the HTTP server will hang.

### Status code

- As mentioned, the response status code is always ``200`` by default, except for POST requests which are ``201``.
  We can easily change this behavior by adding the ``@HttpCode(...)`` decorator at a handler level.

```typescript
@Controller('cats')
export class CatsController {
    @Post()
    @HttpCode(204)
    create() {
        return 'This action adds a new cat';
    }
}

// Import HttpCode from the @nestjs/common package. 
```

- Often, your status code isn't static but depends on various factors. In that case, you can use a library-specific
  response (inject using ``@Res()``) object (or, in case of an error, throw an exception).

### Headers

- To specify a custom response header, you can either use a ``@Header()`` decorator or a library-specific response
  object (and call ``res.header()`` directly).

```typescript
@Controller('cats')
export class CatsController {
    @Post()
    @Header('Cache-Control', 'none')
    create() {
        return 'This action adds a new cat';
    }
}

//Import Header from the @nestjs/common package.
```

### Redirection

- To redirect a response to a specific URL, you can either use a ``@Redirect()`` decorator or a library-specific
  response object (and call ``res.redirect()`` directly).
- ``@Redirect()`` takes two arguments, ``url`` and ``statusCode``, both are optional. The default value of statusCode
  is ``302``(Found) if omitted.

```typescript
@Controller('cats')
export class CatsController {
    @Get()
    @Redirect('https://nestjs.com', 301)
    findAll() {
        return 'This action returns all cats';
    }
}
```

- Sometimes you may want to determine the HTTP status code or the redirect URL dynamically. Do this by returning an
  object following the ``HttpRedirectResponse`` interface (from ``@nestjs/common``).
- Returned values will override any arguments passed to the ``@Redirect()`` decorator. For example:

```typescript
@Controller('docs')
export class CatsController {
    @Get()
    @Redirect('https://docs.nestjs.com', 302)
    getDocs(@Query('version') version) {
        if (version && version === '5') {
            return {url: 'https://docs.nestjs.com/v5/'};
        }
    }

}
```

### Route parameters

- In order to define routes with parameters, we can add route parameter tokens in the path of the route to capture the
  dynamic value at that position in the request URL.
- Route parameters declared in this way can be accessed using the ``@Param()`` decorator, which should be added to the
  method signature.
- Routes with parameters should be declared after any static paths. This prevents the parameterized paths from
  intercepting traffic destined for the static paths.

```typescript
@Controller('docs')
export class CatsController {
    @Get(':id')
    findOne(@Param() params: any): string {
        console.log(params.id);
        return `This action returns a #${params.id} doc`;
    }
}
```

- You can also pass in a particular parameter token to the decorator, and then reference the route parameter directly by
  name in the method body.

```typescript
@Controller('docs')
export class CatsController {
    @Get(':id')
    findOne(@Param('id') id: string): string {
        return `This action returns a #${id} doc`;
    }
}
```

### Sub-Domain Routing

- The ``@Controller`` decorator can take a host option to require that the HTTP host of the incoming requests matches
  some specific value.

```typescript

@Controller({host: 'admin.example.com'})
export class AdminController {
    @Get()
    index(): string {
        return 'Admin page';
    }
}
```

- Similar to a route path, the hosts option can use tokens to capture the dynamic value at that position in the host
  name.

```typescript

@Controller({host: ':account.example.com'})
export class AccountController {
    @Get()
    getInfo(@HostParam('account') account: string) {
        return account;
    }
}
```

### Scopes

- For people coming from different programming language backgrounds, it might be unexpected to learn that in Nest,
  almost everything is shared across incoming requests. We have a connection pool to the database, singleton
  services with global state, etc. Remember that Node.js doesn't follow the request/response Multi-Threaded Stateless
  Model in which every request is processed by a separate thread. Hence, using singleton instances is fully safe for our
  applications.

### Asynchronicity

- Data extraction is mostly asynchronous. That's why Nest supports and works well with ``async`` functions.
- Every ``async`` function has to return a ``Promise``. This means that you can return a deferred value that Nest will
  be able to resolve by itself. Let's see an example of this:

```typescript
@Controller('cats')
export class CatsController {
    @Get()
    async findAll(): Promise<any>[] {
        return [];
    }
}
```

- Furthermore, Nest route handlers are even more powerful by being able to return ``RxJS`` observable streams. Nest will
  automatically subscribe to the source underneath and take the last emitted value (once the stream is completed).

```typescript
@Controller('cats')
export class CatsController {
    @Get()
    async findAll(): Observable<any>[] {
        return of([]);
    }
}
```
