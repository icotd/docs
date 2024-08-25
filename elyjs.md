# 01-at-glance/at-glance.md

# At glance
Elysia is an ergonomic web framework for building backend servers with Bun.

Designed with simplicity and type safety in mind with familiar API with extensive support for TypeScript, optimized for Bun.

Here's a simple hello world in Elysia.

```typescript twoslash
import { Elysia } from 'elysia'

new Elysia()
    .get('/', () => 'Hello Elysia')
    .get('/user/:id', ({ params: { id }}) => id)
    .post('/form', ({ body }) => body)
    .listen(3000)
```

Navigate to [localhost:3000](http://localhost:3000/) and it should show 'Hello Elysia' as a result.

<Playground 
    :elysia="demo1"
    :alias="{
        '/user/:id': '/user/1'
    }"
    :mock="{
        '/user/:id': {
            GET: '1'
        },
        '/form': {
            POST: JSON.stringify({
                hello: 'Elysia'
            })
        }
    }" 
/>
 
## TypeScript

Elysia is designed to help you write less TypeScript.

Elysia's Type System is fine-tuned to infer your code into type automatically without needing to write explicit TypeScript while providing type-safety for both runtime and compile time to provide you with the most ergonomic developer experience.

Take a look at this example:

```typescript twoslash
import { Elysia } from 'elysia'

new Elysia()
    .get('/user/:id', ({ params: { id } }) => id)
                        // ^?
    .listen(3000)
```

The above code create a path parameter "id", the value that replaces `:id` will be passed to `params.id` both in runtime and type without manual type declaration.

<Playground 
    :elysia="demo2"
    :alias="{
        '/user/:id': '/user/123'
    }"
    :mock="{
        '/user/:id': {
            GET: '123'
        },
    }" 
/>

Elysia's goal is to help you write less TypeScript and focus more on Business logic. Let the complex types be handled by the framework.

TypeScript is not needed to use Elysia, but it's recommended to use Elysia with TypeScript.

## Type Integrity

To take a step further, Elysia provide **Elysia.t**, a schema builder to validate type and value in both runtime and compile-time to create a single source of truth for your data-type.

Let's modify the previous code to accept only a numeric value instead of a string.

```typescript twoslash
import { Elysia, t } from 'elysia'

new Elysia()
    .get('/user/:id', ({ params: { id } }) => id, {
                                // ^?
        params: t.Object({
            id: t.Numeric()
        })
    })
    .listen(3000)
```

This code ensures that our path parameter **id**, will always be a numeric string and then transforms it into a number automatically in both runtime and compile-time (type-level).

::: tip
Hover over "id" in the above code snippet to see a type definition.
:::

With Elysia schema builder, we can ensure type safety like a strong-typed language with a single source of truth.

## Standard

Elysia adopts many standards by default, like OpenAPI, and WinterCG compliance, allowing you to integrate with most of the industry standard tools or at least easily integrate with tools you are familiar with.

For instance, as Elysia adopts OpenAPI by default, generating a documentation with Swagger is as easy as adding a one-liner:

```typescript twoslash
import { Elysia, t } from 'elysia'
import { swagger } from '@elysiajs/swagger'

new Elysia()
    .use(swagger())
    .get('/user/:id', ({ params: { id } }) => id, {
        params: t.Object({
            id: t.Numeric()
        })
    })
    .listen(3000)
```

With the Swagger plugin, you can seamlessly generate a Swagger page without additional code or specific config and share it with your team effortlessly.

## End-to-end Type Safety

With Elysia, type safety is not limited to server-side only.

With Elysia, you can synchronize your types with your frontend team automatically like tRPC, with Elysia's client library, "Eden".

```typescript twoslash
import { Elysia, t } from 'elysia'
import { swagger } from '@elysiajs/swagger'

const app = new Elysia()
    .use(swagger())
    .get('/user/:id', ({ params: { id } }) => id, {
        params: t.Object({
            id: t.Numeric()
        })
    })
    .listen(3000)

export type App = typeof app
```

And on your client-side:

```typescript twoslash
// @filename: server.ts
import { Elysia, t } from 'elysia'

const app = new Elysia()
    .get('/user/:id', ({ params: { id } }) => id, {
        params: t.Object({
            id: t.Numeric()
        })
    })
    .listen(3000)

export type App = typeof app

// @filename: client.ts
// ---cut---
// client.ts
import { treaty } from '@elysiajs/eden'
import type { App } from './server'

const app = treaty<App>('localhost:3000')

// Get data from /user/617
const { data } = await app.user({ id: 617 }).get()
      // ^?

console.log(data)
```

With Eden, you can use the existing Elysia types to query Elysia server **without code generation** and synchronize types for both frontend and backend automatically.

Elysia is not only about helping you create a confident backend but for all that is beautiful in this world.


---

# api/constructor.md


# Constructor
You can customize Elysia behavior by:

1. using constructor 
2. using `listen`

## Constructor
Constructor will change some behavior of Elysia.

```typescript
new Elysia({
    serve: {
        hostname: '0.0.0.0'
    }
})
```

## Listen
`.listen` will configure any value for starting the server.

By default `listen` will either accept `number` or `Object`.

For Object, `listen` accepts the same value as `Bun.serve`, you can provide any custom one except `serve`.

```typescript
// ✅ This is fine
new Elysia()
    .listen(3000)

// ✅ This is fine
new Elysia()
    .listen({
        port: 3000,
        hostname: '0.0.0.0'
    })
```

::: tip
For providing WebSocket, please use [`WebSocket`](/patterns/websocket)
:::

## Custom Port
You can provide a custom port from ENV by using `process.env`
```typescript
new Elysia()
    .listen(process.env.PORT ?? )
```

## Retrieve Port
You can get underlying `Server` instance from either using:
`.server` property.
Using callback in `.listen`

```typescript
const app = new Elysia()
    .listen(, ({ hostname, port }) => {
        console.log(`Running at http://${hostname}:${port}`)
    })

// `server` will be null if listen isn't called
console.log(`Running at http://${app.server!.hostname}:${app.server!.port}`)
```


---

# eden/fetch.md


# Eden Fetch
A fetch-like alternative to Eden Treaty .

With Eden Fetch can interact with Elysia server in a type-safe manner using Fetch API.



---

# eden/installation.md


# Eden Installation
Start by installing Eden on your frontend:
```bash
bun add @elysiajs/eden
bun add -d elysia
```

::: tip
Eden needs Elysia to infer utilities type.

Make sure to install Elysia with the version matching on the server.
:::

First, export your existing Elysia server type:
```typescript twoslash
// server.ts
import { Elysia, t } from 'elysia'

const app = new Elysia()
    .get('/', () => 'Hi Elysia')
    .get('/id/:id', ({ params: { id } }) => id)
    .post('/mirror', ({ body }) => body, {
        body: t.Object({
            id: t.Number(),
            name: t.String()
        })
    })
    .listen(3000)

export type App = typeof app // [!code ++]
```

Then consume the Elysia API on client side:
```typescript twoslash
// @filename: server.ts
import { Elysia, t } from 'elysia'

const app = new Elysia()
    .get('/', 'Hi Elysia')
    .get('/id/:id', ({ params: { id } }) => id)
    .post('/mirror', ({ body }) => body, {
        body: t.Object({
            id: t.Number(),
            name: t.String()
        })
    })
    .listen(3000)

export type App = typeof app // [!code ++]

// @filename: index.ts
// ---cut---
// client.ts
import { treaty } from '@elysiajs/eden'
import type { App } from './server' // [!code ++]

const client = treaty<App>('localhost:3000') // [!code ++]

// response: Hi Elysia
const { data: index } = await client.index.get()

// response: 1895
const { data: id } = await client.id({ id: 1895 }).get()

// response: { id: 1895, name: 'Skadi' }
const { data: nendoroid } = await client.mirror.post({
    id: 1895,
    name: 'Skadi'
})

// @noErrors
client.
//     ^|
```

## Gotcha
Sometimes Eden may not infer type from Elysia correctly, the following are the most common workaround to fix Eden type inference.

### Type Strict
Make sure to enable strict mode in **tsconfig.json**
```json
{
  "compilerOptions": {
    "strict": true // [!code ++]
  }
}
```

### Unmatch Elysia version
Eden depends Elysia class to import Elysia instance and infers type correctly.

Make sure that both client and server have a matching Elysia version.

### TypeScript version
Elysia uses newer features and syntax of TypeScript to infer types in a the most performant way. Features like Const Generic and Template Literal are heavily used.

Make sure your client has a **minimum TypeScript version if >= 5.0**

### Method Chaining
To make Eden works, Elysia must be using **method chaining**

Elysia's type system is complex, methods usually introduce a new type to the instance.

Using method chaining will help save that new type reference.

For example:
```typescript twoslash
import { Elysia } from 'elysia'

new Elysia()
    .state('build', 1)
    // Store is strictly typed // [!code ++]
    .get('/', ({ store: { build } }) => build)
    .listen(3000)
```
Using this, **state** now returns a new **ElysiaInstance** type, introducing **build** into store and replace the current one.

Without using method chaining, Elysia doesn't save the new type when introduced, leading to no type inference.
```typescript twoslash
// @errors: 2339
import { Elysia } from 'elysia'

const app = new Elysia()

app.state('build', 1)

app.get('/', ({ store: { build } }) => build)

app.listen(3000)
```

We recommend to **always use method chaining** to provide an accurate type inference.


---

# eden/overview.md


# End-to-End Type Safety
Imagine you have a toy train set. 

Each piece of the train track has to fit perfectly with the next one, like puzzle pieces. 

End-to-end type safety is like making sure all the pieces of the track match up correctly so the train doesn't fall off or get stuck. 

For a framework to have end-to-end type safety means you can connect client and server in a type-safe manner.

Elysia provide end-to-end type safety **without code generation** out of the box with RPC-like connector, **Eden**

<video mute controls>
  <source src="/eden/eden-treaty.mp4" type="video/mp4" />
  Something went wrong trying to load video
</video>

Others framework that support e2e type safety:
- tRPC
- Remix
- SvelteKit
- Nuxt
- TS-Rest

<!-- <iframe
    id="embedded-editor"
    src="https://codesandbox.io/p/sandbox/bun-elysia-rdxljp?embed=1&codemirror=1&hidenavigation=1&hidedevtools=1&file=eden.ts"
    allow="accelerometer"
    sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
    loading="lazy"
/>

::: tip
Hover over variable and function to see type definition.
::: -->

Elysia allows you to change the type on the server and it will be instantly reflected on the client, helping with auto-completion and type-enforcement.

## Eden
Eden is a RPC-like client to connect Elysia  **end-to-end type safety** using only TypeScript's type inference instead of code generation.

Allowing you to sync client and server types effortlessly, weighing less than 2KB.

Eden is consists of 2 modules:
1. Eden Treaty **(recommended)**: an improved version RFC version of Eden Treaty
2. Eden Fetch: Fetch-like client with type safety.

Below is an overview, use-case and comparison for each module.

## Eden Treaty (Recommended)
Eden Treaty is an object-like representation of an Elysia server providing end-to-end type safety and a significantly improved developer experience.

With Eden Treaty we can connect interact Elysia server with full-type support and auto-completion, error handling with type narrowing, and creating type safe unit test.

Example usage of Eden Treaty:
```typescript twoslash
// @filename: server.ts
import { Elysia, t } from 'elysia'

const app = new Elysia()
    .get('/', 'hi')
    .get('/users', () => 'Skadi')
    .put('/nendoroid/:id', ({ body }) => body, {
        body: t.Object({
            name: t.String(),
            from: t.String()
        })
    })
    .get('/nendoroid/:id/name', () => 'Skadi')
    .listen(3000)

export type App = typeof app

// @filename: index.ts
// ---cut---
import { treaty } from '@elysiajs/eden'
import type { App } from './server'

const app = treaty<App>('localhost:3000')

// @noErrors
app.
//  ^|




// Call [GET] at '/'
const { data } = await app.index.get()

// Call [POST] at '/nendoroid/:id'
const { data: nendoroid, error } = await app.nendoroid({ id: 1895 }).post({
    name: 'Skadi',
    from: 'Arknights'
})
```

## Eden Fetch
A fetch-like alternative to Eden Treaty for developers that prefers fetch syntax.
```typescript twoslash
// @filename: server.ts
import { Elysia, t } from 'elysia'

const app = new Elysia()
    .get('/', 'hi')
    .post('/name/:name', ({ body }) => body, {
        body: t.Object({
            branch: t.String(),
            type: t.String()
        })
    })
    .listen(3000)

export type App = typeof app

// @filename: index.ts
// ---cut---
import { edenFetch } from '@elysiajs/eden'
import type { App } from './server'

const fetch = edenFetch<App>('http://localhost:3000')

const { data } = await fetch('/name/:name', {
    method: 'POST',
    params: {
        name: 'Saori'
    },
    body: {
        branch: 'Arius',
        type: 'Striker'
    }
})
```

::: tip NOTE
Unlike Eden Treaty, Eden Fetch doesn't provide Web Socket implementation for Elysia server
:::


---

# eden/test.md


# Eden Test
Using Eden, we can create an integration test with end-to-end type safety and auto-completion.

<video mute controls>
  <source src="/eden/eden-test.mp4" type="video/mp4" />
  Something went wrong trying to load video
</video>

> Using Eden Treaty to create tests by [irvilerodrigues on Twitter](https://twitter.com/irvilerodrigues/status/1724836632300265926)

## Setup
We can use [Bun test](https://bun.sh/guides/test/watch-mode) to create tests.

Create **test/index.test.ts** in the root of project directory with the following:

```typescript
// test/index.test.ts
import { describe, expect, it } from 'bun:test'

import { edenTreaty } from '@elysiajs/eden'

const app = new Elysia()
    .get('/', () => 'hi')
    .listen(3000)

const api = edenTreaty<typeof app>('http://localhost:3000')

describe('Elysia', () => {
    it('return a response', async () => {
        const { data } = await api.get()

        expect(data).toBe('hi')
    })
})
```

Then we can perform tests by running **bun test**

```bash
bun test
```

This allows us to perform integration tests programmatically instead of manual fetch while supporting type checking automatically.


---

# essential/context.md



<script setup>
import Playground from '../../components/nearl/playground.vue'
import { Elysia } from 'elysia'

const demo1 = new Elysia()
    .state('version', 1)
    .get('/a', ({ store: { version } }) => version)
    .get('/b', ({ store }) => store)
    .get('/c', () => 'still ok')

const demo2 = new Elysia()
    // @ts-expect-error
    .get('/error', ({ store }) => store.counter)
    .state('version', 1)
    .get('/', ({ store: { version } }) => version)

const demo3 = new Elysia()
    .derive(({ headers }) => {
        const auth = headers['authorization']

        return {
            bearer: auth?.startsWith('Bearer ') ? auth.slice(7) : null
        }
    })
    .get('/', ({ bearer }) => bearer ?? '12345')

const demo4 = new Elysia()
    .state('counter', 0)
    .state('version', 1)
    .state(({ version, ...store }) => ({
        ...store,
        elysiaVersion: 1
    }))
    // ✅ Create from state remap
    .get('/elysia-version', ({ store }) => store.elysiaVersion)
    // ❌ Excluded from state remap
    .get('/version', ({ store }) => store.version)

const setup = new Elysia({ name: 'setup' })
    .decorate({
        argon: 'a',
        boron: 'b',
        carbon: 'c'
    })

const demo5 = new Elysia()
    .use(
        setup
            .prefix('decorator', 'setup')
    )
    .get('/', ({ setupCarbon }) => setupCarbon)

const demo6 = new Elysia()
    .use(setup.prefix('all', 'setup'))
    .get('/', ({ setupCarbon }) => setupCarbon)

const demo7 = new Elysia()
    .state('counter', 0)
    // ✅ Using reference, value is shared
    .get('/', ({ store }) => store.counter++)
    // ❌ Creating a new variable on primitive value, the link is lost
    .get('/error', ({ store: { counter } }) => counter)

</script>

# Context

Context is a request information passed to a [route handler](/essential/handler).

Context is unique for each request, and is not shared except for `store` which is a global mutable state.

Elysia context consists of:

-   **path** - Pathname of the request
-   **body** - [HTTP message](https://developer.mozilla.org/en-US/docs/Web/HTTP/Messages), form or file upload.
-   **query** - [Query String](https://en.wikipedia.org/wiki/Query_string), include additional parameters for search query as JavaScript Object. (Query is extracted from a value after pathname starting from '?' question mark sign)
-   **params** - Elysia's path parameters parsed as JavaScript object
-   **headers** - [HTTP Header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers), additional information about the request like User-Agent, Content-Type, Cache Hint.
-   **request** - [Web Standard Request](https://developer.mozilla.org/en-US/docs/Web/API/Request)
-   **redirect** - A function to redirect a response
-   **store** - A global mutable store for Elysia instance
-   **cookie** - A global mutable signal store for interacting with Cookie (including get/set)
-   **set** - Property to apply to Response:
    -   **status** - [HTTP status](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status), defaults to 200 if not set.
    -   **headers** - Response headers
    -   **redirect** - Response as a path to redirect to
-   **error** - A function to return custom status code

## Extending context

As Elysia only provides essential information, we can customize Context for our specific need for instance:
- extracting user ID as variable
- inject a common pattern repository
- add a database connection



---

# essential/handler.md


<script setup>
import Playground from '../../components/nearl/playground.vue'
import { Elysia } from 'elysia'

const demo1 = new Elysia()
    .get('/', ({ path }) => path)

const demo2 = new Elysia()
    .get('/', ({ error }) => error(418, "Kirifuji Nagisa"))
</script>

# Handler

After a resource is located, a function that respond is refers as **handler**

```typescript twoslash
import { Elysia } from 'elysia'

new Elysia()
    // the function `() => 'hello world'` is a handler
    .get('/', () => 'hello world')
    .listen(3000)
```

Handler maybe a literal value, and can be inlined.

```typescript twoslash
import { Elysia } from 'elysia'

new Elysia()
    .get('/', 'Hello Elysia')
    .get('/video', Bun.file('kyuukurarin.mp4'))
    .listen(3000)
```

Using an inline value always returns the same value which is useful to optimize performance for static resource like file.

This allows Elysia to compile the response ahead of time to optimize performance.

::: tip
Providing an inline value is not a cache.

Static Resource value, headers and status can be mutate dynamically using lifecycle.
:::


## Context

Context is an request's information sent to server.

```typescript twoslash
import { Elysia } from 'elysia'

new Elysia()
    .get('/', ({ path }) => path)
    .listen(3000)
```

<Playground :elysia="demo1" />

We will be covering context property in the next page [context](/essential/context), for now lets see what handler is capable of.

## Set

**set** is a mutable property that form a response accessible via `Context.set`.

- **set.status** - Set custom status code
- **set.headers** - Append custom headers
- **set.redirect** - Append redirect


## Status
We can return a custom status code by using either:

- **error** function (recommended)
- **set.status**

## error
A dedicated `error` function for returning status code with response.

```typescript twoslash
import { Elysia } from 'elysia'

new Elysia()
    .get('/', ({ error }) => error(418, "Kirifuji Nagisa"))
    .listen(3000)
```

<Playground :elysia="demo2" />

It's recommend to use `error` inside main handler as it has better inference:

- allows TypeScript to check if a return value is correctly type to response schema
- autocompletion for type narrowing base on status code
- type narrowing for error handling using End-to-end type safety (Eden)

## set.status
Set a default status code if not provided.

It's recommended to use this in a plugin that only needs to return a specific status code while allowing the user to return a custom value. For example, HTTP 201/206 or 403/405, etc.

```typescript twoslash
import { Elysia } from 'elysia'

new Elysia()
    .onBeforeHandle(({ set }) => {
        set.status = 418

        return 'Kirifuji Nagisa'
    })
    .get('/', () => 'hi')
    .listen(3000)
```

::: tip
HTTP Status indicates the type of response. If the route handler is executed successfully without error, Elysia will return the status code 200.
:::

You can also set a status code using the common name of the status code instead of using a number.

```typescript twoslash
// @errors 2322
import { Elysia } from 'elysia'

new Elysia()
    .get('/', ({ set }) => {
        set.status
          // ^?

        return 'Kirifuji Nagisa'
    })
    .listen(3000)
```

## set.headers
Allowing us to append or delete a response headers represent as Object.

```typescript twoslash
import { Elysia } from 'elysia'

new Elysia()
    .get('/', ({ set }) => {
        set.headers['x-powered-by'] = 'Elysia'

        return 'a mimir'
    })
    .listen(3000)
```

::: warning
The names of headers should be lowercase to force case-sensitivity consistency for HTTP headers and auto-completion, eg. use `set-cookie` rather than `Set-Cookie`.
:::

## redirect
Redirect a request to another resource.

```typescript twoslash
import { Elysia } from 'elysia'

new Elysia()
    .get('/', ({ redirect }) => {
        return redirect('https://youtu.be/whpVWVWBW4U?&t=8')
    })
    .get('/custom-status', ({ redirect }) => {
        // You can also set custom status to redirect
        return redirect('https://youtu.be/whpVWVWBW4U?&t=8', 302)
    })
    .listen(3000)
```

When using redirect, returned value is not required and will be ignored. As response will be from another resource.

## Response

Elysia is built on top of Web Standard Request/Response.

To comply with the Web Standard, a value returned from route handler will be mapped into a [Response](https://developer.mozilla.org/en-US/docs/Web/API/Response) by Elysia.

Letting you focus on business logic rather than boilerplate code.

```typescript twoslash
import { Elysia } from 'elysia'

new Elysia()
    // Equivalent to "new Response('hi')"
    .get('/', () => 'hi')
    .listen(3000)
```

If you prefer an explicit Response class, Elysia also handles that automatically.

```typescript twoslash
import { Elysia } from 'elysia'

new Elysia()
    .get('/', () => new Response('hi'))
    .listen(3000)
```

::: tip
Using a primitive value or `Response` has near identical performance (+- 0.1%), so pick the one you prefer, regardless of performance.
:::


---

# essential/life-cycle.md


# Life Cycle

Also known as middleware in Express or Hook in Fastify.

Imagine we want to return some HTML.

We need to set **"Content-Type"** headers as **"text/html"** for the browser to render HTML.

Explicitly specifying that the response is HTML could be repetitive if there are a lot of handlers, say ~200 endpoints.

This can lead to a lot of duplicated code just to specify the **"text/html"** **"Content-Type"**

But what if after we send a response, we could detect that the response is an HTML string then append the header automatically?

That's when the concept of Life Cycle comes into play.



---

# essential/path.md


<script setup>
import Playground from '../../components/nearl/playground.vue'

import { Elysia } from 'elysia'

const demo1 = new Elysia()
    .get('/id/:id', ({ params: { id } }) => id)
    .get('/id/123', '123')
    .get('/id/anything', 'anything')
    .get('/id', ({ error }) => error(404))
    .get('/id/anything/test', ({ error }) => error(404))

const demo2 = new Elysia()
    .get('/id/:id', ({ params: { id } }) => id)
    .get('/id/123', '123')
    .get('/id/anything', 'anything')
    .get('/id', ({ error }) => error(404))
    .get('/id/:id/:name', ({ params: { id, name } }) => id + ' ' + name)

const demo3 = new Elysia()
	.get('/id', () => `id: undefined`)
    .get('/id/:id', ({ params: { id } }) => `id: ${id}`)

const demo4 = new Elysia()
    .get('/id/:id', ({ params: { id } }) => id)
    .get('/id/123', '123')
    .get('/id/anything', 'anything')
    .get('/id', ({ error }) => error(404))
    .get('/id/:id/:name', ({ params: { id, name } }) => id + '/' + name)

const demo5 = new Elysia()
    .get('/id/1', () => 'static path')
    .get('/id/:id', () => 'dynamic path')
    .get('/id/*', () => 'wildcard path')
</script>

# Path

A path or pathname is an identifier to locate resources of a server.

```bash
http://localhost:/path/page
```

Elysia uses the path and method to look up the correct resource.

<div class="bg-white rounded-lg">
    <img src="/essential/url-object.svg" alt="URL Representation" />
</div>

A path starts after the origin. Prefix with **/** and ends before search query **(?)**

We can categorize the URL and path as follows:

| URL                             | Path         |
| ------------------------------- | ------------ |
| http://site.com/                | /            |
| http://site.com/hello           | /hello       |
| http://site.com/hello/world     | /hello/world |
| http://site.com/hello?name=salt | /hello       |
| http://site.com/hello#title     | /hello       |

::: tip
If the path is not specified, the browser and web server will treat the path as '/' as a default value.
:::

Elysia will look up each request for [route](/essential/route) and response using [handler](/essential/handler) function.

## Dynamic path

URLs can be both static and dynamic.

Static paths are hardcoded strings that can be used to locate resources of the server, while dynamic paths match some part and captures the value to extract extra information.

For instance, we can extract the user ID from the pathname. For example:

```typescript twoslash
import { Elysia } from 'elysia'

new Elysia()
    .get('/id/:id', ({ params: { id } }) => id)
                      // ^?
    .listen(3000)
```

<br>

Here dynamic path is created with `/id/:id` which tells Elysia to match any path up until `/id`. What comes after that is then stored as **params** object.

<Playground
  :elysia="demo1"
  :alias="{
    '/id/:id': '/id/1'
  }"
  :mock="{
    '/id/:id': {
      GET: '1'
    }
  }"
/>

When requested, the server should return the response as follows:

| Path                   | Response  |
| ---------------------- | --------- |
| /id/1                  | 1         |
| /id/123                | 123       |
| /id/anything           | anything  |
| /id/anything?name=salt | anything  |
| /id                    | Not Found |
| /id/anything/rest      | Not Found |

Dynamic paths are great to include things like IDs, which then can be used later.

We refer to the named variable path as **path parameter** or **params** for short.

## Segment

URL segments are each path that is composed into a full path.

Segments are separated by `/`.
![Representation of URL segments](/essential/url-segment.webp)

Path parameters in Elysia are represented by prefixing a segment with ':' followed by a name.
![Representation of path parameter](/essential/path-parameter.webp)

Path parameters allow Elysia to capture a specific segment of a URL.

The named path parameter will then be stored in `Context.params`.

| Route     | Path   | Params  |
| --------- | ------ | ------- |
| /id/:id   | /id/1  | id=1    |
| /id/:id   | /id/hi | id=hi   |
| /id/:name | /id/hi | name=hi |

## Multiple path parameters

You can have as many path parameters as you like, which will then be stored into a `params` object.

```typescript twoslash
import { Elysia } from 'elysia'

new Elysia()
    .get('/id/:id', ({ params: { id } }) => id)
    .get('/id/:id/:name', ({ params: { id, name } }) => id + ' ' + name)
                             // ^?
    .listen(3000)
```

<br>
<br>

<Playground
  :elysia="demo2"
  :alias="{
    '/id/:id': '/id/1',
    '/id/:id/:name': '/id/anything/rest'
  }"
  :mock="{
    '/id/:id': {
      GET: '1'
    },
    '/id/:id/:name': {
      GET: 'anything rest'
    }
  }"
/>

The server will respond as follows:

| Path                   | Response      |
| ---------------------- | ------------- |
| /id/1                  | 1             |
| /id/123                | 123           |
| /id/anything           | anything      |
| /id/anything?name=salt | anything      |
| /id                    | Not Found     |
| /id/anything/rest      | anything rest |

## Optional path parameters
Sometime we might want a static and dynamic path to resolve the same handler.

We can make a path parameter optional by adding a question mark `?` after the parameter name.

```typescript twoslash
import { Elysia } from 'elysia'

new Elysia()
    .get('/id/:id?', ({ params: { id } }) => `id ${id}`)
                          // ^?
    .listen(3000)
```

<br>

<Playground
  :elysia="demo3"
  :alias="{
    '/id/:id': '/id/1'
  }"
  :mock="{
    '/id/:id': {
      GET: 'id 1'
    },
  }"
/>

The server will respond as follows:

| Path                   | Response      |
| ---------------------- | ------------- |
| /id                    | id undefined  |
| /id/1                  | id 1          |

## Wildcards

Dynamic paths allow capturing certain segments of the URL.

However, when you need a value of the path to be more dynamic and want to capture the rest of the URL segment, a wildcard can be used.

Wildcards can capture the value after segment regardless of amount by using "\*".

```typescript twoslash
import { Elysia } from 'elysia'

new Elysia()
    .get('/id/*', ({ params }) => params['*'])
                    // ^?
    .listen(3000)
```

<br>

<Playground
  :elysia="demo4"
  :alias="{
    '/id/:id': '/id/1',
    '/id/:id/:name': '/id/anything/rest'
  }"
  :mock="{
    '/id/:id': {
      GET: '1'
    },
    '/id/:id/:name': {
      GET: 'anything/rest'
    }
  }"
/>

In this case the server will respond as follows:

| Path                   | Response      |
| ---------------------- | ------------- |
| /id/1                  | 1             |
| /id/123                | 123           |
| /id/anything           | anything      |
| /id/anything?name=salt | anything      |
| /id                    | Not Found     |
| /id/anything/rest      | anything/rest |

Wildcards are useful for capturing a path until a specific point.

::: tip
You can use a wildcard with a path parameter.
:::

## Summary

To summarize, the path in Elysia can be grouped into 3 types:

-   **static paths** - static string to locate the resource
-   **dynamic paths** - segment can be any value
-   **wildcards** - path until a specific point can be anything

You can use all of the path types together to compose a behavior for your web server.

The priorities are as follows:

1. static paths
2. dynamic paths
3. wildcards

If the path is resolved as the static wild dynamic path is presented, Elysia will resolve the static path rather than the dynamic path

```typescript twoslash
import { Elysia } from 'elysia'

new Elysia()
    .get('/id/1', () => 'static path')
    .get('/id/:id', () => 'dynamic path')
    .get('/id/*', () => 'wildcard path')
    .listen(3000)
```

<Playground
  :elysia="demo5"
    :alias="{
    '/id/:id': '/id/2',
    '/id/*': '/id/2/a'
  }"
  :mock="{
    '/id/*': {
      GET: 'wildcard path'
    }
  }"
/>

Here the server will respond as follows:

| Path    | Response      |
| ------- | ------------- |
| /id/1   | static path   |
| /id/2   | dynamic path  |
| /id/2/a | wildcard path |


---

# essential/plugin.md


<script setup>
import Playground from '../../components/nearl/playground.vue'
import { Elysia } from 'elysia'

const plugin = new Elysia()
    .decorate('plugin', 'hi')
    .get('/plugin', ({ plugin }) => plugin)

const demo1 = new Elysia()
    .get('/', ({ plugin }) => plugin)
    .use(plugin)

const plugin2 = (app) => {
    if ('counter' in app.store) return app

    return app
        .state('counter', 0)
        .get('/plugin', () => 'Hi')
}

const demo2 = new Elysia()
    .use(plugin2)
    .get('/counter', ({ store: { counter } }) => counter)

const version = (version = 1) => new Elysia()
        .get('/version', version)

const demo3 = new Elysia()
    .use(version(1))

const setup = new Elysia({ name: 'setup' })
    .decorate('a', 'a')

const plugin3 = (config) => new Elysia({
        name: 'my-plugin', 
        seed: config, 
    })
    .get(`${config.prefix}/hi`, () => 'Hi')

const demo4 = new Elysia()
    .use(
        plugin3({
            prefix: '/v2'
        })
    )

// child.ts
const child = new Elysia()
    .use(setup)
    .get('/', ({ a }) => a)

// index.ts
const demo5 = new Elysia()
    .use(child)
</script>

# Plugin

Plugin is a pattern that decouples functionality into smaller parts. Creating reusable components for our web server.

Defining a plugin is to define a separate instance.

```typescript twoslash
import { Elysia } from 'elysia'

const plugin = new Elysia()
    .decorate('plugin', 'hi')
    .get('/plugin', ({ plugin }) => plugin)

const app = new Elysia()
    .use(plugin)
    .get('/', ({ plugin }) => plugin)
               // ^?
    .listen(3000)
```

We can use the plugin by passing an instance to **Elysia.use**.

<Playground :elysia="demo1" />

The plugin will inherit all properties of the plugin instance, including **state**, **decorate**, **derive**, **route**, **lifecycle**, etc.

Elysia will also handle the type inference automatically as well, so you can imagine as if you call all of the other instances on the main one.

::: tip
Notice that the plugin doesn't contain **.listen**, because **.listen** will allocate a port for the usage, and we only want the main instance to allocate the port.
:::

## Separate File

Using a plugin pattern, you decouple your business logic into a separate file.

First, we define an instance in a difference file:
```typescript twoslash
// plugin.ts
import { Elysia } from 'elysia'

export const plugin = new Elysia()
    .get('/plugin', () => 'hi')
```

And then we import the instance into the main file:
```typescript twoslash
// @filename: plugin.ts
import { Elysia } from 'elysia'

export const plugin = new Elysia()
    .get('/plugin', () => 'hi')
// @filename: index.ts
// ---cut---
// main.ts
import { Elysia } from 'elysia'
import { plugin } from './plugin'

const app = new Elysia()
    .use(plugin)
    .listen(3000)
```

## Config

To make the plugin more useful, allowing customization via config is recommended.

You can create a function that accepts parameters that may change the behavior of the plugin to make it more reusable.

```typescript twoslash
import { Elysia } from 'elysia'

const version = (version = 1) => new Elysia()
        .get('/version', version)

const app = new Elysia()
    .use(version(1))
    .listen(3000)
```

## Functional callback

It's recommended to define a new plugin instance instead of using a function callback.

Functional callback allows us to access the existing property of the main instance. For example, checking if specific routes or stores existed.

To define a functional callback, create a function that accepts Elysia as a parameter.

```typescript twoslash
import { Elysia } from 'elysia'

const plugin = (app: Elysia) => app
    .state('counter', 0)
    .get('/plugin', () => 'Hi')

const app = new Elysia()
    .use(plugin)
    .get('/counter', ({ store: { counter } }) => counter)
    .listen(3000)
```

<Playground :elysia="demo3" />

Once passed to `Elysia.use`, functional callback behaves as a normal plugin except the property is assigned directly to

::: tip
You shall not worry about the performance difference between a functional callback and creating an instance.

Elysia can create 10k instances in a matter of milliseconds, the new Elysia instance has even better type inference performance than the functional callback.
:::

## Plugin Deduplication

By default, Elysia will register any plugin and handle type definitions.

Some plugins may be used multiple times to provide type inference, resulting in duplication of setting initial values or routes.

Elysia avoids this by differentiating the instance by using **name** and **optional seeds** to help Elysia identify instance duplication:

```typescript twoslash
import { Elysia } from 'elysia'

const plugin = <T extends string>(config: { prefix: T }) => 
    new Elysia({
        name: 'my-plugin', // [!code ++]
        seed: config, // [!code ++]
    })
    .get(`${config.prefix}/hi`, () => 'Hi')

const app = new Elysia()
    .use(
        plugin({
            prefix: '/v2'
        })
    )
    .listen(3000)
```

<Playground :elysia="demo4" />

Elysia will use **name** and **seed** to create a checksum to identify if the instance has been registered previously or not, if so, Elysia will skip the registration of the plugin.

If seed is not provided, Elysia will only use **name** to differentiate the instance. This means that the plugin is only registered once even if you registered it multiple times.

```typescript twoslash
import { Elysia } from 'elysia'

const plugin = new Elysia({ name: 'plugin' })

const app = new Elysia()
    .use(plugin)
    .use(plugin)
    .use(plugin)
    .use(plugin)
    .listen(3000)
```

This allows Elysia to improve performance by reusing the registered plugins instead of processing the plugin over and over again.

::: tip
Seed could be anything, varying from a string to a complex object or class.

If the provided value is class, Elysia will then try to use the `.toString` method to generate a checksum.
:::

## Service Locator
When you apply multiple state and decorators plugin to an instance, the instance will gain type safety.

However, you may notice that when you are trying to use the decorated value in another instance without decorator, the type is missing.

```typescript twoslash
// @errors: 2339
import { Elysia } from 'elysia'

const child = new Elysia()
    // ❌ 'a' is missing
    .get('/', ({ a }) => a)

const main = new Elysia()
    .decorate('a', 'a')
    .use(child)
```

This is a TypeScript limitation; Elysia can only refer to the current instance.

Elysia introduces the **Service Locator** pattern to counteract this.

To put it simply, Elysia will lookup the plugin checksum and get the value or register a new one. Infer the type from the plugin.

Simply put, we need to provide the plugin reference for Elysia to find the service.

```typescript twoslash
// @errors: 2339
import { Elysia } from 'elysia'

// setup.ts
const setup = new Elysia({ name: 'setup' })
    .decorate('a', 'a')

// index.ts
const error = new Elysia()
    .get('/', ({ a }) => a)

const main = new Elysia()
    .use(setup)
    .get('/', ({ a }) => a)
    //           ^?
```

<Playground :elysia="demo5" />

## Official Plugins

You can find an officially maintained plugin at Elysia's [plugins](/plugins/overview).

Some plugins include:
- GraphQL
- Swagger
- Server Sent Event

And various community plugins.


---

# essential/route.md


<script setup>
import Playground from '../../components/nearl/playground.vue'
import { Elysia } from 'elysia'

const demo1 = new Elysia()
    .get('/', () => 'hello')
    .get('/hi', () => 'hi')

const demo2 = new Elysia()
    .get('/', () => 'hello')
    .post('/hi', () => 'hi')

const demo3 = new Elysia()
    .get('/get', () => 'hello')
    .post('/post', () => 'hi')
    .route('M-SEARCH', '/m-search', () => 'connect') 

const demo4 = new Elysia()
    .get('/', () => 'hi')
    .post('/', () => 'hi')

const demo5 = new Elysia()
    .get('/', () => 'hello')
    .get('/hi', ({ error }) => error(404, 'Route not found :('))
</script>

# Route

Web servers use the request's **path and HTTP method** to look up the correct resource, refers as **"routing"**.

We can define a route by calling a **method named after HTTP verbs**, passing a path and a function to execute when matched.

```typescript twoslash
import { Elysia } from 'elysia'

new Elysia()
    .get('/', () => 'hello')
    .get('/hi', () => 'hi')
    .listen(3000)
```

We can access the web server by going to **http://localhost:3000**

By default, web browsers will send a GET method when visiting a page.

<Playground :elysia="demo1" />

::: tip
Using an interactive browser above, hover on a blue highlight area to see difference result between each path
:::

## HTTP Verb

There are many HTTP methods to use in a different situation, for instance.

### GET

Requests using GET should only retrieve data.

### POST

Submits a payload to the specified resource, often causing state change or side effect.

### PUT

Replaces all current representations of the target resource using the request's payload.

### DELETE

Deletes the specified resource.



---

# essential/schema.md


<script setup>
import Playground from '../../components/nearl/playground.vue'
import { Elysia, t, ValidationError } from 'elysia'

const demo1 = new Elysia()
    .get('/id/1', 1)
	.get('/id/a', () => {
		throw new ValidationError(
			'params',
			t.Object({
				id: t.Numeric()
			}),
			{
				id: 'a'
			}
		)
	})

const demo2 = new Elysia()
    .get('/none', () => 'hi')
    .guard({ 
        query: t.Object({ 
            name: t.String() 
        }) 
    }) 
    .get('/query', ({ query: { name } }) => name)
    .get('/any', ({ query }) => query)
</script>

# Schema

One of the most important areas to create a secure web server is to make sure that requests are in the correct shape.

Elysia handled this by providing a validation tool out of the box to validate incoming requests using **Schema Builder**.

**Elysia.t**, a schema builder based on [TypeBox](https://github.com/sinclairzx81/typebox) to validate the value in both runtime and compile-time, providing type safety like in a strict type language.

## Type

Elysia schema can validate the following:

-   body - HTTP body.
-   query - query string or URL parameters.
-   params - Path parameters.
-   header - Request's headers.
-   cookie - Request's cookie
-   response - Value returned from handler

Schema can be categorized into 2 types:

1. Local Schema: Validate on a specific route
2. Global Schema: Validate on every route

## Local Schema

The local schema is executed on a specific route.

To validate a local schema, you can inline schema into a route handler:

```typescript twoslash
import { Elysia, t } from 'elysia'

new Elysia()
    .get('/id/:id', ({ params: { id } }) => id, {
                               // ^?
        params: t.Object({ // [!code ++]
            id: t.Numeric() // [!code ++]
        }) // [!code ++]
    })
    .listen(3000)
```

<Playground :elysia="demo1" />

This code ensures that our path parameter **id**, will always be a numeric string and then transform to a number automatically in both runtime and compile-time (type-level).

The response should be listed as follows:

| Path  | Response |
| ----- | -------- |
| /id/1 | 1        |
| /id/a | Error    |

## Global Schema

Register hook into **every** handler that came after.

To add a global hook, you can use `.guard` followed by a life cycle event in camelCase:

```typescript twoslash
import { Elysia, t } from 'elysia'

new Elysia()
    .get('/none', () => 'hi')
    .guard({ // [!code ++]
        query: t.Object({ // [!code ++]
            name: t.String() // [!code ++]
        }) // [!code ++]
    }) // [!code ++]
    .get('/query', ({ query: { name } }) => name)
                    // ^?
    .get('/any', ({ query }) => query)
    .listen(3000)
```

This code ensures that the query must have **name** with a string value for every handler after it. The response should be listed as follows:

<Playground
    :elysia="demo2"
    :mock="{
        '/query': {
            GET: 'Elysia'
        },
        '/any': {
            GET: JSON.stringify({ name: 'Elysia', race: 'Elf' })
        },
    }" 
/>

The response should be listed as follows:

| Path          | Response |
| ------------- | -------- |
| /none         | hi       |
| /none?name=a  | hi       |
| /query        | error    |
| /query?name=a | a        |

If multiple global schemas are defined for same property, the latest one will have the preference. If both local and global schemas are defined, the local one will have the preference.


---

# essential/scope.md


# Scope

<script setup>
import Playground from '../../components/nearl/playground.vue'
import Elysia from 'elysia'

const demo1 = new Elysia()
    .post('/student', 'Rikuhachima Aru')

const plugin2 = new Elysia()
    .onBeforeHandle({ as: 'global' }, () => {
        return 'hi'
    })
    .get('/child', () => 'child')

const demo2 = new Elysia()
    .use(plugin2)
    .get('/parent', () => 'parent')

const mock2 = {
    '/child': {
        'GET': 'hi'
    },
    '/parent': {
        'GET': 'hi'
    }
}

const plugin3 = new Elysia()
    .onBeforeHandle({ as: 'global' }, () => {
        return 'overwrite'
    })

const demo3 = new Elysia()
    .guard(app => app
        .use(plugin3)
        .get('/inner', () => 'inner')
    )
    .get('/outer', () => 'outer')

const mock3 = {
    '/inner': {
        'GET': 'overwrite'
    },
    '/outer': {
        'GET': 'outer'
    }
}
</script>

By default, hook and schema will apply to **current instance only**.

Elysia has an encapsulation scope for to prevent unintentional side effects.

## Scope
Scope type is to specify the scope of hook whether is should be encapsulated or global.

```typescript twoslash
// @errors: 2339
import { Elysia } from 'elysia'

const plugin = new Elysia()
    .derive(() => {
        return { hi: 'ok' }
    })
    .get('/child', ({ hi }) => hi)

const main = new Elysia()
    .use(plugin)
    // ⚠️ Hi is missing
    .get('/parent', ({ hi }) => hi)
```

From the above code, we can see that `hi` is missing from the parent instance because the scope is local by default if not specified, and will not apply to parent.

To apply the hook to the parent instance, we can use the `as` to specify scope of the hook.

```typescript twoslash
// @errors: 2339
import { Elysia } from 'elysia'

const plugin = new Elysia()
    .derive({ as: 'scoped' }, () => { // [!code ++]
        return { hi: 'ok' }
    })
    .get('/child', ({ hi }) => hi)

const main = new Elysia()
    .use(plugin)
    // ✅ Hi is now available
    .get('/parent', ({ hi }) => hi)
```

## Scope level
Elysia has 3 levels of scope as the following:
Scope type are as the following:
- **local** (default) - apply to only current instance and descendant only
- **scoped** - apply to parent, current instance and descendants
- **global** - apply to all instance that apply the plugin (all parents, current, and descendants)

Let's review what each scope type does by using the following example:
```typescript twoslash
import { Elysia } from 'elysia'

// ? Value base on table value provided below
const type = 'local'

const child = new Elysia()
    .get('/child', () => 'hi')

const current = new Elysia()
    .onBeforeHandle({ as: type }, () => { // [!code ++]
        console.log('hi')
    })
    .use(child)
    .get('/current', () => 'hi')

const parent = new Elysia()
    .use(current)
    .get('/parent', () => 'hi')

const main = new Elysia()
    .use(parent)
    .get('/main', () => 'hi')
```

By changing the `type` value, the result should be as follows:

| type       | child | current | parent | main |
| ---------- | ----- | ------- | ------ | ---- |
| 'local'    | ✅    | ✅       | ❌     | ❌   |
| 'scoped'    | ✅    | ✅       | ✅     | ❌   |
| 'global'   | ✅    | ✅       | ✅     | ✅   |

## Guard

Guard allows us to apply hook and schema into multiple routes all at once.

```typescript twoslash
const signUp = <T>(a: T) => a
const signIn = <T>(a: T) => a
const isUserExists = <T>(a: T) => a
// ---cut---
import { Elysia, t } from 'elysia'

new Elysia()
    .guard(
        { // [!code ++]
            body: t.Object({ // [!code ++]
                username: t.String(), // [!code ++]
                password: t.String() // [!code ++]
            }) // [!code ++]
        }, // [!code ++]
        (app) => // [!code ++]
            app
                .post('/sign-up', ({ body }) => signUp(body))
                .post('/sign-in', ({ body }) => signIn(body), {
                                                     // ^?
                    beforeHandle: isUserExists
                })
    )
    .get('/', () => 'hi')
    .listen(3000)
```

This code applies validation for `body` to both '/sign-in' and '/sign-up' instead of inlining the schema one by one but applies not to '/'.

We can summarize the route validation as the following:
| Path | Has validation |
| ------- | ------------- |
| /sign-up | ✅ |
| /sign-in | ✅ |
| / | ❌ |

Guard accepts the same parameter as inline hook, the only difference is that you can apply hook to multiple routes in the scope.

This means that the code above is translated into:

```typescript twoslash
const signUp = <T>(a: T) => a
const signIn = <T>(a: T) => a
const isUserExists = (a: any) => a
// ---cut---
import { Elysia, t } from 'elysia'

new Elysia()
    .post('/sign-up', ({ body }) => signUp(body), {
        body: t.Object({
            username: t.String(),
            password: t.String()
        })
    })
    .post('/sign-in', ({ body }) => body, {
        beforeHandle: isUserExists,
        body: t.Object({
            username: t.String(),
            password: t.String()
        })
    })
    .get('/', () => 'hi')
    .listen(3000)
```

## Grouped Guard

We can use a group with prefixes by providing 3 parameters to the group.

1. Prefix - Route prefix
2. Guard - Schema
3. Scope - Elysia app callback

With the same API as guard apply to the 2nd parameter, instead of nesting group and guard together.

Consider the following example:
```typescript twoslash
import { Elysia, t } from 'elysia'

new Elysia()
    .group('/v1', (app) =>
        app.guard(
            {
                body: t.Literal('Rikuhachima Aru')
            },
            (app) => app.post('/student', ({ body }) => body)
                                            // ^?
        )
    )
    .listen(3000)
```


From nested groupped guard, we may merge group and guard together by providing guard scope to 2nd parameter of group:
```typescript twoslash
import { Elysia, t } from 'elysia'

new Elysia()
    .group(
        '/v1',
        (app) => app.guard( // [!code --]
        {
            body: t.Literal('Rikuhachima Aru')
        },
        (app) => app.post('/student', ({ body }) => body)
        ) // [!code --]
    )
    .listen(3000)
```

Which results in the follows syntax:
```typescript twoslash
import { Elysia, t } from 'elysia'

new Elysia()
    .group(
        '/v1',
        {
            body: t.Literal('Rikuhachima Aru')
        },
        (app) => app.post('/student', ({ body }) => body)
                                       // ^?
    )
    .listen(3000)
```

<Playground :elysia="demo1" />

## Scope cast
To apply hook to parent may use one of the following:
1. `inline as` apply only to a single hook
2. `guard as` apply to all hook in a guard
3. `instance as` apply to all hook in an instance

### 1. Inline as
Every event listener will accept `as` parameter to specify the scope of the hook.

```typescript twoslash
import { Elysia } from 'elysia'

const plugin = new Elysia()
    .derive({ as: 'scoped' }, () => { // [!code ++]
        return { hi: 'ok' }
    })
    .get('/child', ({ hi }) => hi)

const main = new Elysia()
    .use(plugin)
    // ✅ Hi is now available
    .get('/parent', ({ hi }) => hi)
```

However, this method is apply to only a single hook, and may not be suitable for multiple hooks.

### 2. Guard as
Every event listener will accept `as` parameter to specify the scope of the hook.

```typescript twoslash
import { Elysia, t } from 'elysia'

const plugin = new Elysia()
	.guard({
		as: 'scoped', // [!code ++]
		response: t.String(),
		beforeHandle() {
			console.log('ok')
		}
	})
    .get('/child', () => 'ok')

const main = new Elysia()
    .use(plugin)
    .get('/parent', () => 'hello')
```

Guard alllowing us to apply `schema` and `hook` to multiple routes all at once while specifying the scope.

However, it doesn't support `derive` and `resolve` method.

### 3. Instance as
`as` will read all hooks and schema scope of the current instance, modify.

```typescript twoslash
import { Elysia } from 'elysia'

const plugin = new Elysia()
    .derive(() => { // [!code ++]
        return { hi: 'ok' }
    })
    .get('/child', ({ hi }) => hi)
    .as('plugin')

const main = new Elysia()
    .use(plugin)
    // ✅ Hi is now available
    .get('/parent', ({ hi }) => hi)
```

Sometimes we want to reapply plugin to parent instance as well but as it's limited by `scoped` mechanism, it's limited to 1 parent only.

To apply to the parent instance, we need to **"lift the scope up** to the parent instance, and `as` is the perfect method to do so.

Which means if you have `local` scope, and want to apply it to the parent instance, you can use `as('plugin')` to lift it up.
```typescript twoslash
// @errors: 2304 2345
import { Elysia, t } from 'elysia'

const plugin = new Elysia()
	.guard({
		response: t.String()
	})
	.onBeforeHandle(() => { console.log('called') })
	.get('/ok', () => 'ok')
	.get('/not-ok', () => 1)
	.as('plugin') // [!code ++]

const instance = new Elysia()
	.use(plugin)
	.get('/no-ok-parent', () => 2)
	.as('plugin') // [!code ++]

const parent = new Elysia()
	.use(instance)
	// This now error because `scoped` is lifted up to parent
	.get('/ok', () => 3)
```

## Plugin

By default plugin will only **apply hook to itself and descendants** only.

If the hook is registered in a plugin, instances that inherit the plugin will **NOT** inherit hooks and schema.

```typescript twoslash
import { Elysia } from 'elysia'

const plugin = new Elysia()
    .onBeforeHandle(() => {
        console.log('hi')
    })
    .get('/child', () => 'log hi')

const main = new Elysia()
    .use(plugin)
    .get('/parent', () => 'not log hi')
```

To apply hook to globally, we need to specify hook as global.
```typescript twoslash
import { Elysia } from 'elysia'

const plugin = new Elysia()
    .onBeforeHandle(() => {
        return 'hi'
    })
    .get('/child', () => 'child')
    .as('plugin')

const main = new Elysia()
    .use(plugin)
    .get('/parent', () => 'parent')
```

<Playground :elysia="demo2" :mock="mock2" />


---

# essential/what-next.md


# What's next
Congratulation! You have just completed an essential chapter for developing with Elysia.

Essential chapter cover a basic building block, however there are some several useful concepts you might want to read, each take less ~15 minutes to complete.

Here's a recommended chapters we recommended in order (Feels free to jump to the chapter you are interested first).

<script setup>
    import Card from '../../components/nearl/card.vue'
    import Deck from '../../components/nearl/card-deck.vue'
</script>

<Deck>
    <Card title="Validation" href="/validation/overview">
        Schema to enforce data type
    </Card>
    <Card title="Life Cycle" href="/life-cycle/overview">
        Intercept correct order for each request
    </Card>
    <Card title="Plugin" href="/plugins/overview">
        Checkout plugins and ecosystem
    </Card>
    <Card title="Eden" href="/eden/overview">
        Integrate your frontend with E2E type safety
    </Card>
    <Card title="MVC model" href="/patterns/mvc">
        Using MVC model with Elysia
    </Card>
    <Card title="Cheat sheet" href="/integrations/cheat-sheet">
        A quick overview of Elysia
    </Card>
</Deck>

## If you are stuck

Feels free to ask our community on GitHub Discussions, Discord, and Twitter, if you have any further question.

<Deck>
    <Card title="Discord" href="https://discord.gg/eaFJ2KDJck">
        Official ElysiaJS discord community server
    </Card>
    <Card title="Twitter" href="https://twitter.com/elysiajs">
        Track update and status of Elysia
    </Card>
    <Card title="GitHub" href="https://github.com/elysiajs">
        Source code and development
    </Card>
</Deck>

We wish you happy on your journey with Elysia ❤️


---

# integrations/astro.md


# Integration with Astro

With [Astro Endpoint](https://docs.astro.build/en/core-concepts/endpoints/), we can run Elysia on Astro directly.

1. Set **output** to **server** in **astro.config.mjs**

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config'

// https://astro.build/config
export default defineConfig({
    output: 'server' // [!code ++]
})
```

2. Create **pages/[...slugs].ts**
3. Create or import an existing Elysia server in **[...slugs].ts**
4. Export the handler with the name of method you want to expose

```typescript twoslash
// pages/[...slugs].ts
import { Elysia, t } from 'elysia'

const app = new Elysia()
    .get('/api', () => 'hi')
    .post('/api', ({ body }) => body, {
        body: t.Object({
            name: t.String()
        })
    })

const handle = ({ request }: { request: Request }) => app.handle(request) // [!code ++]

export const GET = handle // [!code ++]
export const POST = handle // [!code ++]
```

Elysia will work normally as expected because of WinterCG compliance.

We recommended running [Astro on Bun](https://docs.astro.build/en/recipes/bun) as Elysia is designed to be run on Bun

::: tip
You can run Elysia server without running Astro on Bun thanks to WinterCG support.

However some plugins like **Elysia Static** may not work if you are running Astro on Node.
:::

With this approach, you can have co-location of both frontend and backend in a single repository and have End-to-end type-safety with Eden.

Please refer to [Astro Endpoint](https://docs.astro.build/en/core-concepts/endpoints/) for more information.

## Prefix

If you place an Elysia server not in the root directory of the app router, you need to annotate the prefix to the Elysia server.

For example, if you place Elysia server in **pages/api/[...slugs].ts**, you need to annotate prefix as **/api** to Elysia server.

```typescript twoslash
// pages/api/[...slugs].ts
import { Elysia, t } from 'elysia'

const app = new Elysia({ prefix: '/api' }) // [!code ++]
    .get('/', () => 'hi')
    .post('/', ({ body }) => body, {
        body: t.Object({
            name: t.String()
        })
    })

const handle = ({ request }: { request: Request }) => app.handle(request) // [!code ++]

export const GET = handle // [!code ++]
export const POST = handle // [!code ++]
```

This will ensure that Elysia routing will work properly in any location you place it.


---

# integrations/cheat-sheet.md


# Cheat Sheet
Here are a quick overview for a common Elysia patterns

## Hello World
A simple hello world

```typescript twoslash
import { Elysia } from 'elysia'

new Elysia()
    .get('/', () => 'Hello World')
    .listen(3000)
```

## Custom HTTP Method
Define route using custom HTTP methods/verbs

See [Route](/essential/route.html#custom-method)

```typescript twoslash
import { Elysia } from 'elysia'

new Elysia()
    .get('/hi', () => 'Hi')
    .post('/hi', () => 'From Post')
    .put('/hi', () => 'From Put')
    .route('M-SEARCH', '/hi', () => 'Custom Method')
    .listen(3000)
```

## Path Parameter
Using dynamic path parameter

See [Path](/essential/path.html)

```typescript twoslash
import { Elysia } from 'elysia'

new Elysia()
    .get('/id/:id', ({ params: { id } }) => id)
    .get('/rest/*', () => 'Rest')
    .listen(3000)
```

## Return JSON
Elysia convert JSON to response automatically

See [Handler](/essential/handler.html)

```typescript twoslash
import { Elysia } from 'elysia'

new Elysia()
    .get('/json', () => {
        return {
            hello: 'Elysia'
        }
    })
    .listen(3000)
```

## Return a file
A file can be return in as formdata response

The response must 1-level deep object

```typescript twoslash
import { Elysia } from 'elysia'

new Elysia()
    .get('/json', () => {
        return {
            hello: 'Elysia',
            image: Bun.file('public/cat.jpg')
        }
    })
    .listen(3000)
```

## Header and status
Set a custom header and a status code

See [Handler](/essential/handler.html)

```typescript twoslash
import { Elysia } from 'elysia'

new Elysia()
    .get('/', ({ set, error }) => {
        set.headers['x-powered-by'] = 'Elysia'

        return error(418, "I'm teapod")
    })
    .listen(3000)
```

## Group
Define a prefix once for sub routes

See [Group](/patterns/group.html)

```typescript twoslash
import { Elysia } from 'elysia'

new Elysia()
    .get("/", () => "Hi")
    .group("/auth", app => {
        return app
            .get("/", () => "Hi")
            .post("/sign-in", ({ body }) => body)
            .put("/sign-up", ({ body }) => body)
    })
    .listen(3000)
```

## Schema
Enforce a data type of a route

See [Schema](/essential/schema.html)

```typescript twoslash
import { Elysia, t } from 'elysia'

new Elysia()
    .post('/mirror', ({ body: { username } }) => username, {
        body: t.Object({
            username: t.String(),
            password: t.String()
        })
    })
    .listen(3000)
```

## Lifecycle Hook
Intercept an Elysia event in order

See [Lifecycle](/essential/life-cycle.html)

```typescript twoslash
import { Elysia, t } from 'elysia'

new Elysia()
    .onRequest(() => {
        console.log('On request')
    })
    .on('beforeHandle', () => {
        console.log('Before handle')
    })
    .post('/mirror', ({ body }) => body, {
        body: t.Object({
            username: t.String(),
            password: t.String()
        }),
        afterHandle: () => {
            console.log("After handle")
        }
    })
    .listen(3000)
```

## Guard
Enforce a data type of sub routes

See [Scope](/essential/scope.html#guard)

```typescript twoslash
// @errors: 2345
import { Elysia, t } from 'elysia'

new Elysia()
    .guard({
        response: t.String()
    }, (app) => app
        .get('/', () => 'Hi')
        // Invalid: will throws error, and TypeScript will report error
        .get('/invalid', () => 1)
    )
    .listen(3000)
```

## Customize context
Add custom variable to route context

See [Context](/essential/context.html)

```typescript twoslash
import { Elysia } from 'elysia'

new Elysia()
    .state('version', 1)
    .decorate('getDate', () => Date.now())
    .get('/version', ({ 
        getDate, 
        store: { version } 
    }) => `${version} ${getDate()}`)
    .listen(3000)
```

## Redirect
Redirect a response

See [Handler](/essential/handler.html#redirect)

```typescript twoslash
import { Elysia } from 'elysia'

new Elysia()
    .get('/', () => 'hi')
    .get('/redirect', ({ redirect }) => {
        return redirect('/')
    })
    .listen(3000)
```

## Plugin
Create a separate instance

See [Plugin](/essential/plugin)

```typescript twoslash
import { Elysia } from 'elysia'

const plugin = new Elysia()
    .state('plugin-version', 1)
    .get('/hi', () => 'hi')

new Elysia()
    .use(plugin)
    .get('/version', ({ store }) => store['plugin-version'])
    .listen(3000)
```

## Web Socket
Create a realtime connection using Web Socket

See [Web Socket](/patterns/websocket)

```typescript twoslash
import { Elysia } from 'elysia'

new Elysia()
    .ws('/ping', {
        message(ws, message) {
            ws.send('hello ' + message)
        }
    })
    .listen(3000)
```

## OpenAPI documentation
Create a interactive documentation using Scalar (or optionally Swagger)

See [Documentation](/patterns/documentation)

```typescript twoslash
import { Elysia } from 'elysia'
import { swagger } from '@elysiajs/swagger'

const app = new Elysia()
    .use(swagger())
    .listen(3000)

console.log(`View documentation at "${app.server!.url}swagger" in your browser`);
```

## Unit Test
Write a unit test of your Elysia app

See [Unit Test](/patterns/unit-test)

```typescript twoslash
// test/index.test.ts
import { describe, expect, it } from 'bun:test'
import { Elysia } from 'elysia'

describe('Elysia', () => {
    it('return a response', async () => {
        const app = new Elysia().get('/', () => 'hi')

        const response = await app
            .handle(new Request('http://localhost/'))
            .then((res) => res.text())

        expect(response).toBe('hi')
    })
})
```

## Custom body parser
Create a custom logic for parsing body

See [Parse](/life-cycle/parse.html)

```typescript twoslash
import { Elysia } from 'elysia'

new Elysia()
    .onParse(({ request, contentType }) => {
        if (contentType === 'application/custom-type')
            return request.text()
    })
```

## GraphQL
Create a custom GraphQL server using GraphQL Yoga or Apollo

See [GraphQL Yoga](/plugins/graphql-yoga)

```typescript
import { Elysia } from 'elysia'
import { yoga } from '@elysiajs/graphql-yoga'

const app = new Elysia()
    .use(
        yoga({
            typeDefs: /* GraphQL */`
                type Query {
                    hi: String
                }
            `,
            resolvers: {
                Query: {
                    hi: () => 'Hello from Elysia'
                }
            }
        })
    )
    .listen(3000)
```


---

# integrations/custom-404.md


# Custom 404
You can define custom 404 using `onError` hook:
```typescript
import { Elysia } from 'elysia'

new Elysia()
    .onError(({ code, error }) => {
        if (code === 'NOT_FOUND')
            return new Response('Not Found :(', {
                status: 404
            })
    })
    .listen(3000)
```


---

# integrations/docker.md


# Docker
You use Elysia with Docker with the following Dockerfile below:
```docker
FROM oven/bun

WORKDIR /app

COPY package.json .
COPY bun.lockb .

RUN bun install --production

COPY src src
COPY tsconfig.json .
# COPY public public

ENV NODE_ENV production
CMD ["bun", "src/index.ts"]

EXPOSE 3000
```

## Distroless
If you like to use Distroless:
```docker
FROM debian:11.6-slim as builder

WORKDIR /app

RUN apt update
RUN apt install curl unzip -y

RUN curl https://bun.sh/install | bash

COPY package.json .
COPY bun.lockb .

RUN /root/.bun/bin/bun install --production

# ? -------------------------
FROM gcr.io/distroless/base

WORKDIR /app

COPY --from=builder /root/.bun/bin/bun bun
COPY --from=builder /app/node_modules node_modules

COPY src src
COPY tsconfig.json .
# COPY public public

ENV NODE_ENV production
CMD ["./bun", "src/index.ts"]

EXPOSE 3000
```

## Development

To develop with Elysia in Docker, you can use the following minimal docker compose template:

```yaml
# docker-compose.yml
version: '3.9'

services:
  app:
    image: "oven/bun"
    # override default entrypoint allows us to do `bun install` before serving
    entrypoint: []
    # execute bun install before we start the dev server in watch mode
    command: "/bin/sh -c 'bun install && bun run --watch src/index.ts'"
    # expose the right ports
    ports: ["3000:3000"]
    # setup a host mounted volume to sync changes to the container
    volumes: ["./:/home/bun/app"]
```

---

# integrations/drizzle.md


# Drizzle
[Drizzle](https://orm.drizzle.team) is a TypeScript ORM that offers type integrity out of the box.

Allowing us to define and infers Database schema into TypeScript type directly allowing us to perform end-to-end type safety from database to server to client-side.

## Drizzle Typebox
[Elysia.t](/validation/overview) is a fork of TypeBox, allowing us to use any TypeBox type in Elysia directly.

We can convert Drizzle schema into TypeBox schema using ["drizzle-typebox"](https://npmjs.org/package/drizzle-typebox), and use it directly on Elysia's schema validation.

```typescript
import { Elysia, t } from 'elysia'

import { createInsertSchema } from 'drizzle-typebox'
import { sqliteTable, text } from 'drizzle-orm/sqlite-core'
import { createId } from '@paralleldrive/cuid2'

const user = sqliteTable('user', {
    id: text('id').primaryKey().$defaultFn(createId),
    username: text('username').notNull(),
    password: text('password').notNull(),
})

const createUser = createInsertSchema(user)

const auth = new Elysia({ prefix: '/auth' })
    .put(
        '/sign-up',
        ({ body }) => createUser(body),
        {
            body: t.Omit(createUser, ['id'])
        }
    )
```

Or if you want to add a custom field on validation-side, eg. file uploading:
```typescript
import { Elysia, t } from 'elysia'

import { createInsertSchema } from 'drizzle-typebox'
import { sqliteTable, text } from 'drizzle-orm/sqlite-core'
import { createId } from '@paralleldrive/cuid2'

const user = sqliteTable('user', {
    id: text('id').primaryKey().$defaultFn(createId),
    username: text('username').notNull(),
    password: text('password').notNull(),
    image: text('image')
})

const createUser = createInsertSchema(user, {
    image: t.File({ // [!code ++]
        type: 'image', // [!code ++]
        maxSize: '2m' // [!code ++]
    }) // [!code ++]
})

const auth = new Elysia({ prefix: '/auth' })
    .put(
        '/sign-up',
        async ({ body: { image, ...body } }) => {
            const imageURL = await uploadImage(image) // [!code ++]
// [!code ++]
            return createUser({ image: imageURL, ...body }) // [!code ++]
        },
        {
            body: t.Omit(createUser, ['id'])
        }
    )
```


---

# integrations/expo.md


# Integration with Expo

Starting from Expo SDK 50, and App Router v3, Expo allows us to create API route directly in an Expo app.

1. Create an Expo app if not exists with:
```typescript
bun create expo-app --template tabs
```

2. Create **app/[...slugs]+api.ts**
3. In **[...slugs]+api.ts**, create or import an existing Elysia server
4. Export the handler with the name of method you want to expose

```typescript twoslash
// app/[...slugs]+api.ts
import { Elysia, t } from 'elysia'

const app = new Elysia()
    .get('/', () => 'hello Next')
    .post('/', ({ body }) => body, {
        body: t.Object({
            name: t.String()
        })
    })

export const GET = app.handle // [!code ++]
export const POST = app.handle // [!code ++]
```

Elysia will work normally as expected because of WinterCG compliance, however, some plugins like **Elysia Static** may not work if you are running Expo on Node.

You can treat the Elysia server as if normal Expo API route.

With this approach, you can have co-location of both frontend and backend in a single repository and have [End-to-end type safety with Eden](https://elysiajs.com/eden/overview.html) with both client-side and server action

Please refer to [API route](https://docs.expo.dev/router/reference/api-routes/) for more information.

## Prefix
If you place an Elysia server not in the root directory of the app router, you need to annotate the prefix to the Elysia server.

For example, if you place Elysia server in **app/api/[...slugs]+api.ts**, you need to annotate prefix as **/api** to Elysia server.

```typescript twoslash
// app/api/[...slugs]+api.ts
import { Elysia, t } from 'elysia'

const app = new Elysia({ prefix: '/api' }) // ![code ++]
    .get('/', () => 'hi')
    .post('/', ({ body }) => body, {
        body: t.Object({
            name: t.String()
        })
    })

export const GET = app.handle
export const POST = app.handle
```

This will ensure that Elysia routing will works properly in any location you place in.

## Deployment
You can either directly use API route using Elysia and deploy as normal Elysia app normally if need or using [experimental Expo server runtime](https://docs.expo.dev/router/reference/api-routes/#deployment).

If you are using Expo server runtime, you may use `expo export` command to create optimized build for your expo app, this will include an Expo function which is using Elysia at **dist/server/_expo/functions/[...slugs\]+api.js**

::: tip
Please note that Expo Function are treated as Edge function instead of normal server, so running the Edge function directly will not allocate any port.
:::

You may use the Expo function adapter provided by Expo to deploy your Edge Function.

Currently Expo support the following adapter:
- [Express](https://docs.expo.dev/router/reference/api-routes/#express)
- [Netlify](https://docs.expo.dev/router/reference/api-routes/#netlify)
- [Vercel](https://docs.expo.dev/router/reference/api-routes/#vercel)


---

# integrations/nextjs.md


# Integration with Nextjs

With Nextjs App Router, we can run Elysia on Nextjs route.

1. Create **api/[[...slugs]]/route.ts** inside app router
2. In **route.ts**, create or import an existing Elysia server
3. Export the handler with the name of method you want to expose

```typescript twoslash
// app/api/[[...slugs]]/route.ts
import { Elysia, t } from 'elysia'

const app = new Elysia({ prefix: '/api' })
    .get('/', () => 'hello Next')
    .post('/', ({ body }) => body, {
        body: t.Object({
            name: t.String()
        })
    })

export const GET = app.handle // [!code ++]
export const POST = app.handle // [!code ++]
```

Elysia will work normally as expected because of WinterCG compliance, however, some plugins like **Elysia Static** may not work if you are running Nextjs on Node.

You can treat the Elysia server as a normal Nextjs API route.

With this approach, you can have co-location of both frontend and backend in a single repository and have [End-to-end type safety with Eden](https://elysiajs.com/eden/overview.html) with both client-side and server action

Please refer to [Nextjs Route Handlers](https://nextjs.org/docs/app/building-your-application/routing/route-handlers#static-route-handlers) for more information.

## Prefix

Because our Elysia server is not in the root directory of the app router, you need to annotate the prefix to the Elysia server.

For example, if you place Elysia server in **app/user/[[...slugs]]/route.ts**, you need to annotate prefix as **/user** to Elysia server.

```typescript twoslash
// app/api/[[...slugs]]/route.ts
import { Elysia, t } from 'elysia'

const app = new Elysia({ prefix: '/user' }) // [!code ++]
    .get('/', () => 'hi')
    .post('/', ({ body }) => body, {
        body: t.Object({
            name: t.String()
        })
    })

export const GET = app.handle
export const POST = app.handle
```

This will ensure that Elysia routing will work properly in any location you place it.


---

# integrations/sveltekit.md


# Integration with SvelteKit

With SvelteKit, you can run Elysia on server routes.

1. Create **src/routes/[...slugs]/+server.ts**.
2. In **+server.ts**, create or import an existing Elysia server
3. Export the handler with the name of method you want to expose

```typescript twoslash
// src/routes/[...slugs]/+server.ts
import { Elysia, t } from 'elysia';

const app = new Elysia()
    .get('/', () => 'hello SvelteKit')
    .post('/', ({ body }) => body, {
        body: t.Object({
            name: t.String()
        })
    })

type RequestHandler = (v: { request: Request }) => Response | Promise<Response>

export const GET: RequestHandler = ({ request }) => app.handle(request)
export const POST: RequestHandler = ({ request }) => app.handle(request)
```

You can treat the Elysia server as a normal SvelteKit server route.

With this approach, you can have co-location of both frontend and backend in a single repository and have [End-to-end type-safety with Eden](https://elysiajs.com/eden/overview.html) with both client-side and server action

Please refer to [SvelteKit Routing](https://kit.svelte.dev/docs/routing#server) for more information.

## Prefix
If you place an Elysia server not in the root directory of the app router, you need to annotate the prefix to the Elysia server.

For example, if you place Elysia server in **src/routes/api/[...slugs]/+server.ts**, you need to annotate prefix as **/api** to Elysia server.

```typescript twoslash
// src/routes/api/[...slugs]/+server.ts
import { Elysia, t } from 'elysia';

const app = new Elysia({ prefix: '/api' }) // [!code ++]
    .get('/', () => 'hi')
    .post('/', ({ body }) => body, {
        body: t.Object({
            name: t.String()
        })
    })

type RequestHandler = (v: { request: Request }) => Response | Promise<Response>

export const GET: RequestHandler = ({ request }) => app.handle(request)
export const POST: RequestHandler = ({ request }) => app.handle(request)
```

This will ensure that Elysia routing will work properly in any location you place it.


---

# life-cycle/after-handle.md


# After Handle

Execute after the main handler, for mapping a returned value of **before handle** and **route handler** into a proper response.

It's recommended to use After Handle in the following situations:

-   Transform requests into a new value, eg. Compression, Event Stream
-   Add custom headers based on the response value, eg. **Content-Type**

## Example

Below is an example of using the after handle to add HTML content type to response headers.

```typescript twoslash
import { Elysia } from 'elysia'
import { isHtml } from '@elysiajs/html'

new Elysia()
    .get('/', () => '<h1>Hello World</h1>', {
        afterHandle({ response, set }) {
            if (isHtml(response))
                set.headers['content-type'] = 'text/html; charset=utf8'
        }
    })
    .get('/hi', () => '<h1>Hello World</h1>')
    .listen(3000)
```

The response should be listed as follows:

| Path | Content-Type             |
| ---- | ------------------------ |
| /    | text/html; charset=utf8  |
| /hi  | text/plain; charset=utf8 |

## Returned Value

If a value is returned After Handle will use a return value as a new response value unless the value is **undefined**

The above example could be rewritten as the following:

```typescript twoslash
import { Elysia } from 'elysia'
import { isHtml } from '@elysiajs/html'

new Elysia()
    .get('/', () => '<h1>Hello World</h1>', {
        afterHandle({ response, set }) {
            if (isHtml(response)) {
                set.headers['content-type'] = 'text/html; charset=utf8'
                return new Response(response)
            }
        }
    })
    .get('/hi', () => '<h1>Hello World</h1>')
    .listen(3000)
```

Unlike **beforeHandle**, after a value is returned from **afterHandle**, the iteration of afterHandle **will **NOT** be skipped.**

## Context

`onAfterHandle` context extends from `Context` with the additional property of `response`, which is the response to return to the client.

The `onAfterHandle` context is based on the normal context and can be used like the normal context in route handlers.

---

# life-cycle/after-response.md


# After Response
Executed after the response sent to the client.

It's recommended to use **After Response** in the following situations:
- Clean up response
- Logging and analytics

## Example
Below is an example of using the response handle to check for user sign-in.

```typescript twoslash
import { Elysia } from 'elysia'

new Elysia()
	.onAfterResponse(() => {
		console.log('Response', performance.now())
	})
	.listen(3000)
```

Console should log as the following:

```bash
Response 0.0000
Response 0.0001
Response 0.0002
```


---

# life-cycle/before-handle.md


# Before Handle

Execute after validation and before the main route handler.

Designed to provide a custom validation to provide a specific requirement before running the main handler.

If a value is returned, the route handler will be skipped.

It's recommended to use Before Handle in the following situations:

-   Restricted access check: authorization, user sign-in
-   Custom request requirement over data structure

## Example

Below is an example of using the before handle to check for user sign-in.

```typescript twoslash
// @filename: user.ts
export const validateSession = (a?: string): boolean => true

// @filename: index.ts
// ---cut---
import { Elysia } from 'elysia'
import { validateSession } from './user'

new Elysia()
    .get('/', () => 'hi', {
        beforeHandle({ set, cookie: { session } }) {
            if (!validateSession(session.value))
                return (set.status = 'Unauthorized')
        }
    })
    .listen(3000)
```

The response should be listed as follows:

| Is signed in | Response     |
| ------------ | ------------ |
| ❌           | Unauthorized |
| ✅           | Hi           |

## Guard

When we need to apply the same before handle to multiple routes, we can use [guard](#guard) to apply the same before handle to multiple routes.

```typescript twoslash
// @filename: user.ts
export const validateSession = (a?: string): boolean => true
export const isUserExists = (a: unknown): boolean => true
export const signUp = (body: unknown): boolean => true
export const signIn = (body: unknown): boolean => true

// @filename: index.ts
// ---cut---
import { Elysia } from 'elysia'
import {
    signUp,
    signIn,
    validateSession,
    isUserExists
} from './user'

new Elysia()
    .guard(
        {
            beforeHandle({ set, cookie: { session } }) {
                if (!validateSession(session.value))
                    return (set.status = 'Unauthorized')
            }
        },
        (app) =>
            app
                .get('/user/:id', ({ body }) => signUp(body))
                .post('/profile', ({ body }) => signIn(body), {
                    beforeHandle: isUserExists
                })
    )
    .get('/', () => 'hi')
    .listen(3000)
```

## Resolve

A "safe" version of [derive](/life-cycle/before-handle#derive).

Designed to append new value to context after validation process storing in the same stack as **beforeHandle**.

Resolve syntax is identical to [derive](/life-cycle/before-handle#derive), below is an example of retrieving a bearer header from the Authorization plugin.

```typescript twoslash
// @filename: user.ts
export const validateSession = (a: string): boolean => true

// @filename: index.ts
// ---cut---
import { Elysia, t } from 'elysia'

new Elysia()
    .guard(
        {
            headers: t.Object({
                authorization: t.TemplateLiteral('Bearer ${string}')
            })
        },
        (app) =>
            app
                .resolve(({ headers: { authorization } }) => {
                    return {
                        bearer: authorization.split(' ')[1]
                    }
                })
                .get('/', ({ bearer }) => bearer)
    )
    .listen(3000)
```

Using `resolve` and `onBeforeHandle` is stored in the same queue.

```typescript twoslash
import { Elysia } from 'elysia'

new Elysia()
    .onBeforeHandle(() => {
        console.log(1)
    })
    .resolve(() => {
        console.log(2)

        return {}
    })
    .onBeforeHandle(() => {
        console.log(3)
    })
```

The console should log as the following:

```bash
1
2
3
```

Same as **derive**, properties which assigned by **resolve** is unique and not shared with another request.

## Guard resolve

As resolve is not available in local hook, it's recommended to use guard to encapsulate the **resolve** event.

```typescript twoslash
// @filename: user.ts
export const isSignIn = (body: any): boolean | undefined => true
export const findUserById = (id?: string) => id

// @filename: index.ts
// ---cut---
import { Elysia } from 'elysia'
import { isSignIn, findUserById } from './user'

new Elysia()
    .guard(
        {
            beforeHandle: isSignIn
        },
        (app) =>
            app
                .resolve(({ cookie: { session } }) => ({
                    userId: findUserById(session.value)
                }))
                .get('/profile', ({ userId }) => userId)
    )
    .listen(3000)
```


---

# life-cycle/map-response.md


# Map Response

Executed just after **"afterHandle"**, designed to provide custom response mapping.

It's recommended to use transform for the following:

-   Compression
-   Map value into a Web Standard Response

## Example

Below is an example of using mapResponse to provide Response compression.

```typescript twoslash
import { Elysia } from 'elysia'

const encoder = new TextEncoder()

new Elysia()
    .mapResponse(({ response, set }) => {
        const isJson = typeof response === 'object'

        const text = isJson
            ? JSON.stringify(response)
            : response?.toString() ?? ''

        set.headers['Content-Encoding'] = 'gzip'

        return new Response(
            Bun.gzipSync(encoder.encode(text)),
            {
                headers: {
                    'Content-Type': `${
                        isJson ? 'application/json' : 'text/plain'
                    }; charset=utf-8`
                }
            }
        )
    })
    .get('/text', () => 'mapResponse')
    .get('/json', () => ({ map: 'response' }))
    .listen(3000)
```

Like **parse** and **beforeHandle**, after a value is returned, the next iteration of **mapResponse** will be skipped.

Elysia will handle the merging process of **set.headers** from **mapResponse** automatically. We don't need to worry about appending **set.headers** to Response manually.


---

# life-cycle/on-error.md


# Error Handling

**On Error** is the only life-cycle event that is not always executed on each request, but only when an error is thrown in any other life-cycle at least once.

Designed to capture and resolve an unexpected error, its recommended to use on Error in the following situation:

-   To provide custom error message
-   Fail safe or an error handler or retrying a request
-   Logging and analytic

## Example

Elysia catches all the errors thrown in the handler, classifies the error code, and pipes them to `onError` middleware.

```typescript twoslash
import { Elysia } from 'elysia'

new Elysia()
    .onError(({ code, error }) => {
        return new Response(error.toString())
    })
    .get('/', () => {
        throw new Error('Server is during maintenance')

        return 'unreachable'
    })
```

With `onError` we can catch and transform the error into a custom error message.

::: tip
It's important that `onError` must be called before the handler we want to apply it to.
:::

For example, returning custom 404 messages:

```typescript twoslash
import { Elysia, NotFoundError } from 'elysia'

new Elysia()
    .onError(({ code, error, set }) => {
        if (code === 'NOT_FOUND') {
            set.status = 404

            return 'Not Found :('
        }
    })
    .post('/', () => {
        throw new NotFoundError()
    })
    .listen(3000)
```

## Context

`onError` Context is extends from `Context` with additional properties of the following:

-   error: Error object thrown
-   code: Error Code

### Error Code

Elysia error code consists of:

-   NOT_FOUND
-   INTERNAL_SERVER_ERROR
-   VALIDATION
-   PARSE
-   UNKNOWN

By default, the thrown error code is `unknown`.

::: tip
If no error response is returned, the error will be returned using `error.name`.
:::

## Custom Error

Elysia supports custom error both in the type-level and implementation level.

To provide a custom error code, we can use `Elysia.error` to add a custom error code, helping us to easily classify and narrow down the error type for full type safety with auto-complete as the following:

```typescript twoslash
import { Elysia } from 'elysia'

class MyError extends Error {
    constructor(public message: string) {
        super(message)
    }
}

new Elysia()
    .error({
        MyError
    })
    .onError(({ code, error }) => {
        switch (code) {
            // With auto-completion
            case 'MyError':
                // With type narrowing
                // Hover to see error is typed as `CustomError`
                return error
        }
    })
    .get('/', () => {
        throw new MyError('Hello Error')
    })
```

Properties of `error` code is based on the properties of `error`, the said properties will be used to classify the error code.

## Local Error

Same as others life-cycle, we provide an error into an [scope](/essential/scope) using guard:

```typescript twoslash
const isSignIn = (headers: Headers): boolean => true
// ---cut---
import { Elysia } from 'elysia'

new Elysia()
    .get('/', () => 'Hello', {
        beforeHandle({ set, request: { headers } }) {
            if (!isSignIn(headers)) {
                set.status = 401

                throw new Error('Unauthorized')
            }
        },
        error({ error }) {
            return 'Handled'
        }
    })
    .listen(3000)
```


---

# life-cycle/overview.md


<script setup>
    import Card from '../../components/nearl/card.vue'
    import Deck from '../../components/nearl/card-deck.vue'
</script>

# Life Cycle

It's recommended that you have read [Essential life-cycle](/essential/life-cycle) for better understanding of Elysia's Life Cycle.

Life Cycle allows us to intercept an important event at the predefined point allowing us to customize the behavior of our server as needed.

Elysia's Life Cycle event can be illustrated as the following.
![Elysia Life Cycle Graph](/assets/lifecycle.webp)

Below are the request life cycle available in Elysia:

<Deck>
    <Card title="Request" href="request">
        Notify new event is received
    </Card>
    <Card title="Parse" href="parse">
        Parse body into <b>Context.body</b>
    </Card>
    <Card title="Transform" href="transform">
        Modify <b>Context</b> before validation
    </Card>
    <Card title="Before Handle" href="before-handle">
        Custom validation before route handler
    </Card>
    <Card title="After Handle" href="after-handle">
        Transform returned value into a new value
    </Card>
    <Card title="Map Response" href="map-response">
        Map returned value into a response
    </Card>
    <Card title="On Error" href="on-error">
        Capture error when thrown
    </Card>
    <Card title="On Response" href="on-response">
        Executed after response sent to the client
    </Card>
    <Card title="Trace" href="trace">
        Audit and capture timespan of each event
    </Card>
</Deck>



---

# life-cycle/parse.md


# Parse
Parse is an equivalent of **body parser** in Express.

A function to parse body, the return value will be append to `Context.body`, if not, Elysia will continue iterating through additional parser functions assigned by `onParse` until either body is assigned or all parsers have been executed.

By default, Elysia will parse the body with content-type of:
- `text/plain`
- `application/json`
- `multipart/form-data`
- `application/x-www-form-urlencoded`

It's recommended to use the `onParse` event to provide a custom body parser that Elysia doesn't provide.

## Example
Below is an example code to retrieve value based on custom headers.

```typescript twoslash
import { Elysia } from 'elysia'

new Elysia()
    .onParse(({ request, contentType }) => {
        if (contentType === 'application/custom-type')
            return request.text()
    })
```

The returned value will be assigned to Context.body. If not, Elysia will continue iterating through additional parser functions from **onParse** stack until either body is assigned or all parsers have been executed.

## Context
`onParse` Context is extends from `Context` with additional properties of the following:
- contentType: Content-Type header of the request

All of the context is based on normal context and can be used like normal context in route handler.

## Explicit Body

By default, Elysia will try to determine body parsing function ahead of time and pick the most suitable function to speed up the process.

Elysia is able to determine that body function by reading `body`.

Take a look at this example:
```typescript twoslash
import { Elysia, t } from 'elysia'

new Elysia()
    .post('/', ({ body }) => body, {
        body: t.Object({
            username: t.String(),
            password: t.String()
        })
    })
```

Elysia read the body schema and found that, the type is entirely an object, so it's likely that the body will be JSON. Elysia then picks the JSON body parser function ahead of time and tries to parse the body.

Here's a criteria that Elysia uses to pick up type of body parser

- `application/json`: body typed as `t.Object`
- `multipart/form-data`: body typed as `t.Object`, and is 1 level deep with `t.File`
- `application/x-www-form-urlencoded`: body typed as `t.URLEncoded`
- `text/plain`: other primitive type

This allows Elysia to optimize body parser ahead of time, and reduce overhead in compile time.

## Explicit Content Type
However, in some scenario if Elysia fails to pick the correct body parser function, we can explicitly tell Elysia to use a certain function by specifying `type`

```typescript twoslash
import { Elysia } from 'elysia'

new Elysia()
    .post('/', ({ body }) => body, {
        // Short form of application/json
        type: 'json',
    })
```

Allowing us to control Elysia behavior for picking body parser function to fit our needs in a complex scenario.

`type` may be one of the following:
```typescript
type ContentType = |
    // Shorthand for 'text/plain'
    | 'text'
    // Shorthand for 'application/json'
    | 'json'
    // Shorthand for 'multipart/form-data'
    | 'formdata'
    // Shorthand for 'application/x-www-form-urlencoded'\
    | 'urlencoded'
    | 'text/plain'
    | 'application/json'
    | 'multipart/form-data'
    | 'application/x-www-form-urlencoded'
```


---

# life-cycle/request.md


# Request
The first life-cycle event to get executed for every new request is recieved.

As `onRequest` is designed to provide only the most crucial context to reduce overhead, it is recommended to use in the following scenario:
- Caching
- Rate Limiter / IP/Region Lock
- Analytic
- Provide custom header, eg. CORS

## Example
Below is a pseudo code to enforce rate-limit on a certain IP address.
```typescript twoslash
import { Elysia } from 'elysia'

// ---cut-start---
const rateLimiter = new Elysia()
    .decorate({
        ip: '127.0.0.1' as string,
        rateLimiter: {
            check(ip: string) {
                return true as boolean
            }
        }
    })
// ---cut-end---
new Elysia()
    .use(rateLimiter)
    .onRequest(({ rateLimiter, ip, set }) => {
        if(rateLimiter.check(ip)) {
            set.status = 420
            return 'Enhance your calm'
        }
    })
    .get('/', () => 'hi')
    .listen(3000)
```

If a value is returned from `onRequest`, it will be used as the response and the rest of the life-cycle will be skipped.

## Pre Context
Context's onRequest is typed as `PreContext`, a minimal representation of `Context` with the attribute on the following:
request: `Request`
- set: `Set`
- store
- decorators

Context doesn't provide `derived` value because derive is based on `onTransform` event.


---

# life-cycle/trace.md


# Trace

Performance is an important aspect for Elysia.

We don't want to be fast for benchmarking purposes, we want you to have a real fast server in real-world scenario.

There are many factors that can slow down our app - and it's hard to identify them, but **trace** can helps solve that problem by injecting start and stop code to each life-cycle.

Trace allows us to inject code to before and after of each life-cycle event, block and interact with the execution of the function.

## Trace
Trace use a callback listener to ensure that callback function is finished before moving on to the next lifecycle event.

To use `trace`, you need to call `trace` method on the Elysia instance, and pass a callback function that will be executed for each life-cycle event.

You may listen to each lifecycle by adding `on` prefix follows by life-cycle name, for example `onHandle` to listen to `handle` event.

```ts twoslash
import { Elysia } from 'elysia'

const app = new Elysia()
    .trace(async ({ onHandle }) => {
	    onHandle(({ begin, onStop }) => {
			onStop(({ end }) => {
        		console.log('handle took', end - begin, 'ms')
			})
	    })
    })
    .get('/', () => 'Hi')
    .listen(3000)
```

Please refer to [Life Cycle Events](/essential/life-cycle#events) for more information:

![Elysia Life Cycle](/assets/lifecycle.webp)

## Children
Every events except `handle` have a children, which is an array of events that are executed inside for each life-cycle event.

You can use `onEvent` to listen to each child event in order

```ts twoslash
import { Elysia } from 'elysia'

const sleep = (time = 1000) =>
    new Promise((resolve) => setTimeout(resolve, time))

const app = new Elysia()
    .trace(async ({ onBeforeHandle }) => {
        onBeforeHandle(({ total, onEvent }) => {
            console.log('total children:', total)

            onEvent(({ onStop }) => {
                onStop(({ elapsed }) => {
                    console.log('child took', elapsed, 'ms')
                })
            })
        })
    })
    .get('/', () => 'Hi', {
        beforeHandle: [
            function setup() {},
            async function delay() {
                await sleep()
            }
        ]
    })
    .listen(3000)
```

In this example, total children will be `2` because there are 2 children in the `beforeHandle` event.

Then we listen to each child event by using `onEvent` and print the duration of each child event.

## Trace Parameter
When each lifecycle is called

```ts twoslash
import { Elysia } from 'elysia'

const app = new Elysia()
	// This is trace parameter
	// hover to view the type
	.trace((parameter) => {
	})
	.get('/', () => 'Hi')
	.listen(3000)
```

`trace` accept the following parameters:

### id - `number`
Randomly generated unique id for each request

### context - `Context`
Elysia's [Context](/essential/context), eg. `set`, `store`, `query, `params`

### set - `Context.set`
Shortcut for `context.set`, to set a headers or status of the context

### store - `Singleton.store`
Shortcut for `context.store`, to access a data in the context

### time - `number`
Timestamp of when request is called

### on[Event] - `TraceListener`
An event listener for each life-cycle event.

You may listen to the following life-cycle:
-   **onRequest** - get notified of every new request
-   **onParse** - array of functions to parse the body
-   **onTransform** - transform request and context before validation
-   **onBeforeHandle** - custom requirement to check before the main handler, can skip the main handler if response returned.
-   **onHandle** - function assigned to the path
-   **onAfterHandle** - interact with the response before sending it back to the client
-   **onMapResponse** - map returned value into a Web Standard Response
-   **onError** - handle error thrown during processing request
-   **onAfterResponse** - cleanup function after response is sent

## Trace Listener
A listener for each life-cycle event

```ts twoslash
import { Elysia } from 'elysia'

const app = new Elysia()
	.trace(({ onBeforeHandle }) => {
		// This is trace listener
		// hover to view the type
		onBeforeHandle((parameter) => {

		})
	})
	.get('/', () => 'Hi')
	.listen(3000)
```

Each lifecycle listener accept the following

### name - `string`
The name of the function, if the function is anonymous, the name will be `anonymous`

### begin - `number`
The time when the function is started

### end - `Promise<number>`
The time when the function is ended, will be resolved when the function is ended

### error - `Promise<Error | null>`
Error that was thrown in the lifecycle, will be resolved when the function is ended

### onStop - `callback?: (detail: TraceEndDetail) => any`
A callback that will be executed when the lifecycle is ended

```ts twoslash
import { Elysia } from 'elysia'

const app = new Elysia()
	.trace(({ onBeforeHandle, set }) => {
		onBeforeHandle(({ onStop }) => {
			onStop(({ elapsed }) => {
				set.headers['X-Elapsed'] = elapsed.toString()
			})
		})
	})
	.get('/', () => 'Hi')
	.listen(3000)
```

It's recommended to mutate context in this function as there's a lock mechanism to ensure the context is mutate successfully before moving on to the next lifecycle event

## TraceEndDetail
A parameter that passed to `onStop` callback

### end - `number`
The time when the function is ended

### error - `Error | null`
Error that was thrown in the lifecycle

### elapsed - `number`
Elapsed time of the lifecycle or `end - begin`


---

# life-cycle/transform.md


# Transform
Executed just before **Validation** process, designed to mutate context to conform with the validation or appending new value.

It's recommended to use transform for the following:
- Mutate existing context to conform with validation.
- `derive` is based on `onTransform` with support for providing type.

## Example
Below is an example of using transform to mutate params to be numeric values.

```typescript twoslash
import { Elysia, t } from 'elysia'

new Elysia()
    .get('/id/:id', ({ params: { id } }) => id, {
        params: t.Object({
            id: t.Number()
        }),
        transform({ params }) {
            const id = +params.id

            if(!Number.isNaN(id))
                params.id = id
        }
    })
    .listen(3000)
```

## Derive
Designed to append new value to context directly before validation process storing in the same stack as **transform**.

Unlike **state** and **decorate** that assigned value before the server started. **derive** assigns a property when each request happens. Allowing us to extract a piece of information into a property instead.

```typescript twoslash
import { Elysia } from 'elysia'

new Elysia()
    .derive(({ headers }) => {
        const auth = headers['Authorization']

        return {
            bearer: auth?.startsWith('Bearer ') ? auth.slice(7) : null
        }
    })
    .get('/', ({ bearer }) => bearer)
```

Because **derive** is assigned once a new request starts, **derive** can access Request properties like **headers**, **query**, **body** where **store**, and **decorate** can't.

Unlike **state**, and **decorate**. Properties which assigned by **derive** is unique and not shared with another request.

## Queue twoslash
Using `derived` and `transform` is stored in the same queue.

```typescript twoslash
import { Elysia } from 'elysia'

new Elysia()
    .onTransform(() => {
        console.log(1)
    })
    .derive(() => {
        console.log(2)

        return {}
    })
```

The console should log as the following:

```bash
1
2
```

---

# patterns/cookie-signature.md


# Cookie Signature
And lastly, with an introduction of Cookie Schema, and `t.Cookie` type. We are able to create a unified type for handling sign/verify cookie signature automatically.

Cookie signature is a cryptographic hash appended to a cookie's value, generated using a secret key and the content of the cookie to enhance security by adding a signature to the cookie.

This make sure that the cookie value is not modified by malicious actor, helps in verifying the authenticity and integrity of the cookie data.

## Using Cookie Signature
By provide a cookie secret, and `sign` property to indicate which cookie should have a signature verification.
```ts twoslash
import { Elysia, t } from 'elysia'

new Elysia()
    .get('/', ({ cookie: { profile } }) => {
        profile.value = {
            id: 617,
            name: 'Summoning 101'
        }
    }, {
        cookie: t.Cookie({
            profile: t.Object({
                id: t.Numeric(),
                name: t.String()
            })
        }, {
            secrets: 'Fischl von Luftschloss Narfidort',
            sign: ['profile']
        })
    })
```

Elysia then sign and unsign cookie value automatically.

## Constructor
You can use Elysia constructor to set global cookie `secret`, and `sign` value to apply to all route globally instead of inlining to every route you need.

```ts twoslash
import { Elysia, t } from 'elysia'

new Elysia({
    cookie: {
        secrets: 'Fischl von Luftschloss Narfidort',
        sign: ['profile']
    }
})
    .get('/', ({ cookie: { profile } }) => {
        profile.value = {
            id: 617,
            name: 'Summoning 101'
        }
    }, {
        cookie: t.Cookie({
            profile: t.Object({
                id: t.Numeric(),
                name: t.String()
            })
        })
    })
```

## Cookie Rotation
Elysia handle Cookie's secret rotation automatically.

Cookie Rotation is a migration technique to sign a cookie with a newer secret, while also be able to verify the old signature of the cookie.

```ts twoslash
import { Elysia } from 'elysia'

new Elysia({
    cookie: {
        secrets: ['Vengeance will be mine', 'Fischl von Luftschloss Narfidort']
    }
})
```

## Config
Below is a cookie config accepted by Elysia.

### secret
The secret key for signing/un-signing cookies.

If an array is passed, will use Key Rotation.

Key rotation is when an encryption key is retired and replaced by generating a new cryptographic key.



---

# patterns/cookie.md


# Cookie
To use Cookie, you can extract the cookie property and access its name and value directly.

There's no get/set, you can extract the cookie name and retrieve or update its value directly.
```ts twoslash
import { Elysia } from 'elysia'

new Elysia()
    .get('/', ({ cookie: { name } }) => {
        // Get
        name.value

        // Set
        name.value = "New Value"
    })
```

By default, Reactive Cookie can encode/decode type of object automatically allowing us to treat cookie as an object without worrying about the encoding/decoding. **It just works**.

## Reactivity
The Elysia cookie is reactive. This means that when you change the cookie value, the cookie will be updated automatically based on approach like signal.

A single source of truth for handling cookies is provided by Elysia cookies, which have the ability to automatically set headers and sync cookie values.

Since cookies are Proxy-dependent objects by default, the extract value can never be **undefined**; instead, it will always be a value of `Cookie<unknown>`, which can be obtained by invoking the **.value** property.

We can treat the cookie jar as a regular object, iteration over it will only iterate over an already-existing cookie value.

## Cookie Attribute
To use Cookie attribute, you can either use one of the following:

1. Setting the property directly
2. Using `set` or `add` to update cookie property.

See [cookie attribute config](/patterns/cookie-signature#config) for more information.

### Assign Property
You can get/set the property of a cookie like any normal object, the reactivity model synchronizes the cookie value automatically.

```ts twoslash
import { Elysia } from 'elysia'

new Elysia()
    .get('/', ({ cookie: { name } }) => {
        // get
        name.domain

        // set
        name.domain = 'millennium.sh'
        name.httpOnly = true
    })
```

## set
**set** permits updating multiple cookie properties all at once through **reset all property** and overwrite the property with a new value.

```ts twoslash
import { Elysia } from 'elysia'

new Elysia()
    .get('/', ({ cookie: { name } }) => {
        name.set({
            domain: 'millennium.sh',
            httpOnly: true
        })
    })
```

## add
Like **set**, **add** allow us to update multiple cookie properties at once, but instead, will only overwrite the property defined instead of resetting.

## remove
To remove a cookie, you can use either:
1. name.remove
2. delete cookie.name

```ts twoslash
import { Elysia } from 'elysia'

new Elysia()
    .get('/', ({ cookie, cookie: { name } }) => {
        name.remove()

        delete cookie.name
    })
```

## Cookie Schema
You can strictly validate cookie type and providing type inference for cookie by using cookie schema with `t.Cookie`.

```ts twoslash
import { Elysia, t } from 'elysia'

new Elysia()
    .get('/', ({ cookie: { name } }) => {
        // Set
        name.value = {
            id: 617,
            name: 'Summoning 101'
        }
    }, {
        cookie: t.Cookie({
            name: t.Object({
                id: t.Numeric(),
                name: t.String()
            })
        })
    })
```

## Nullable Cookie
To handle nullable cookie value, you can use `t.Optional` on the cookie name you want to be nullable.

```ts twoslash
import { Elysia, t } from 'elysia'

new Elysia()
    .get('/', ({ cookie: { name } }) => {
        // Set
        name.value = {
            id: 617,
            name: 'Summoning 101'
        }
    }, {
        cookie: t.Cookie({
            name: t.Optional(
                t.Object({
                    id: t.Numeric(),
                    name: t.String()
                })
            )
        })
    })
```


---

# patterns/documentation.md


# Creating Documentation
Elysia has first-class support and follows OpenAPI schema by default.

Allowing any Elysia server to generate a Swagger page and serve as documentation automatically by using just 1 line of the Elysia Swagger plugin.

To generate the Swagger page, install the plugin:
```bash
bun add @elysiajs/swagger
```

And register the plugin to the server:
```typescript twoslash
import { Elysia } from 'elysia'
import { swagger } from '@elysiajs/swagger'

const app = new Elysia()
    .use(swagger())
```

For more information about Swagger plugin, see the [Swagger plugin page](/plugins/swagger).

## Route definitions
`schema` is used to customize the route definition, not only that it will generate an OpenAPI schema and Swagger definitions, but also type validation, type-inference and auto-completion.

However, sometime defining a type only isn't clear what the route might work. You can use `schema.detail` fields to explictly define what the route is all about.

```typescript twoslash
import { Elysia, t } from 'elysia'
import { swagger } from '@elysiajs/swagger'

new Elysia()
    .use(swagger())
    .post('/sign-in', ({ body }) => body, {
        body: t.Object(
            {
                username: t.String(),
                password: t.String()
            },
            {
                description: 'Expected an username and password'
            }
        ),
        detail: {
            summary: 'Sign in the user',
            tags: ['authentication']
        }
    })
```

The detail fields follows an OpenAPI V3 definition with auto-completion and type-safety by default.

Detail is then passed to Swagger to put the description to Swagger route.


---

# patterns/group.md


<script setup>
    import Playground from '../../components/nearl/playground.vue'
    import { Elysia } from 'elysia'

    const demo1 = new Elysia()
        .post('/user/sign-in', () => 'Sign in')
        .post('/user/sign-up', () => 'Sign up')
        .post('/user/profile', () => 'Profile')

    const demo2 = new Elysia()
        .group('/user', (app) =>
            app
                .post('/sign-in', () => 'Sign in')
                .post('/sign-up', () => 'Sign up')
                .post('/profile', () => 'Profile')
        )

    const users = new Elysia({ prefix: '/user' })
        .post('/sign-in', () => 'Sign in')
        .post('/sign-up', () => 'Sign up')
        .post('/profile', () => 'Profile')

    const demo3 = new Elysia()
        .get('/', () => 'hello world')
        .use(users)
</script>

# Grouping Routes

When creating a web server, you would often have multiple routes sharing the same prefix:

```typescript twoslash
import { Elysia } from 'elysia'

new Elysia()
    .post('/user/sign-in', () => 'Sign in')
    .post('/user/sign-up', () => 'Sign up')
    .post('/user/profile', () => 'Profile')
    .listen(3000)
```

<Playground :elysia="demo1" />

This can be improved with `Elysia.group`, allowing us to apply prefixes to multiple routes at the same time by grouping them together:

```typescript twoslash
import { Elysia } from 'elysia'

new Elysia()
    .group('/user', (app) =>
        app
            .post('/sign-in', () => 'Sign in')
            .post('/sign-up', () => 'Sign up')
            .post('/profile', () => 'Profile')
    )
    .listen(3000)
```

<Playground :elysia="demo2" />

This code behaves the same as our first example and should be structured as follows:

| Path          | Result  |
| ------------- | ------- |
| /user/sign-in | Sign in |
| /user/sign-up | Sign up |
| /user/profile | Profile |

`.group()` can also accept an optional guard parameter to reduce boilerplate of using groups and guards together:

```typescript twoslash
import { Elysia, t } from 'elysia'

new Elysia()
    .group(
        '/user', 
        { 
            body: t.Literal('Rikuhachima Aru')
        },
        (app) => app
            .post('/sign-in', () => 'Sign in')
            .post('/sign-up', () => 'Sign up')
            .post('/profile', () => 'Profile')
    )
    .listen(3000)
```

You may find more information about grouped guards in [scope](/essential/scope.html).

## Prefixing

We can separate a group into a separate plugin instance to reduce nesting by providing a **prefix** to the constructor.

```typescript twoslash
import { Elysia } from 'elysia'

const users = new Elysia({ prefix: '/user' })
    .post('/sign-in', () => 'Sign in')
    .post('/sign-up', () => 'Sign up')
    .post('/profile', () => 'Profile')

new Elysia()
    .use(users)
    .get('/', () => 'hello world')
    .listen(3000)
```

<Playground :elysia="demo3" />


---

# patterns/lazy-loading-module.md


# Lazy-Loading Module
Modules are eagerly loaded by default.

Elysia loads all modules then registers and indexes all of them before starting the server. This enforces that all the modules have loaded before it starts accepting requests.

While this is fine for most applications, it may become a bottleneck for a server running in a serverless environment or an edge function, in which the startup time is important.

Lazy-loading can help decrease startup time by deferring modules to be gradually indexed after the server start.

Lazy-loading modules are a good option when some modules are heavy and importing startup time is crucial.

By default, any async plugin without await is treated as a deferred module and the import statement as a lazy-loading module.

Both will be registered after the server is started.

## Deferred Module
The deferred module is an async plugin that can be registered after the server is started.

```typescript twoslash
// @filename: files.ts
export const loadAllFiles = async () => <string[]>[]

// @filename: plugin.ts
// ---cut---
// plugin.ts
import { Elysia } from 'elysia'
import { loadAllFiles } from './files'

export const loadStatic = async (app: Elysia) => {
    const files = await loadAllFiles()

    files.forEach((file) => app
        .get(file, () => Bun.file(file))
    )

    return app
}
```

And in the main file:
```typescript twoslash
// @filename: plugin.ts
import { Elysia } from 'elysia'

export const loadAllFiles = async () => <string[]>[]

export const loadStatic = async (app: Elysia) => {
    const files = await loadAllFiles()

    files.forEach((file) => app
        .get(file, () => Bun.file(file))
    )

    return app
}

// @filename: index.ts
// ---cut---
// plugin.ts
import { Elysia } from 'elysia'
import { loadStatic } from './plugin'

const app = new Elysia()
    .use(loadStatic)
```

Elysia static plugin is also a deferred module, as it loads files and registers files path asynchronously.

## Lazy Load Module
Same as the async plugin, the lazy-load module will be registered after the server is started.

A lazy-load module can be both sync or async function, as long as the module is used with `import` the module will be lazy-loaded.

```typescript twoslash
// @filename: plugin.ts
import { Elysia } from 'elysia'

export default new Elysia()

// @filename: index.ts
// ---cut---
import { Elysia } from 'elysia'

const app = new Elysia()
    .use(import('./plugin'))
```

Using module lazy-loading is recommended when the module is computationally heavy and/or blocking.

To ensure module registration before the server starts, we can use `await` on the deferred module.

## Testing
In a test environment, we can use `await app.modules` to wait for deferred and lazy-loading modules.

```typescript twoslash
import { describe, expect, it } from 'bun:test'
import { Elysia } from 'elysia'

describe('Modules', () => {
    it('inline async', async () => {
        const app = new Elysia()
              .use(async (app) =>
                  app.get('/async', () => 'async')
              )

        await app.modules

        const res = await app
            .handle(new Request('http://localhost/async'))
            .then((r) => r.text())

        expect(res).toBe('async')
    })
})
```


---

# patterns/macro.md


# Macro

Macro allows us to define a custom field to the hook.

**Elysia.macro** allows us to compose custom heavy logic into a simple configuration available in hook, and **guard** with full type safety.

```typescript twoslash
import { Elysia } from 'elysia'

const plugin = new Elysia({ name: 'plugin' })
    .macro(({ onBeforeHandle }) => ({
        hi(word: string) {
            onBeforeHandle(() => {
                console.log(word)
            })
        }
    }))

const app = new Elysia()
    .use(plugin)
    .get('/', () => 'hi', {
        hi: 'Elysia'
    })
```

Accessing the path should log **"Elysia"** as the results.

## API

**macro** should return an object, each key is reflected to the hook, and the provided value inside the hook will be sent back as the first parameter.

In previous example, we create **hi** accepting a **string**.

We then assigned **hi** to **"Elysia"**, the value was then sent back to the **hi** function, and then the function added a new event to **beforeHandle** stack.

Which is an equivalent of pushing function to **beforeHandle** as the following:

```typescript twoslash
import { Elysia } from 'elysia'

const app = new Elysia()
    .get('/', () => 'hi', {
        beforeHandle() {
            console.log('Elysia')
        }
    })
```

**macro** shine when a logic is more complex than accepting a new function, for example creating an authorization layer for each route.

```typescript twoslash
// @filename: auth.ts
import { Elysia } from 'elysia'

export const auth = new Elysia()
    .macro(() => {
        return {
            isAuth(isAuth: boolean) {},
            role(role: 'user' | 'admin') {},
        }
    })

// @filename: index.ts
// ---cut---
import { Elysia } from 'elysia'
import { auth } from './auth'

const app = new Elysia()
    .use(auth)
    .get('/', () => 'hi', {
        isAuth: true,
        role: 'admin'
    })
```

The field can accept anything ranging from string to function, allowing us to create a custom life cycle event.

macro will be executed in order from top-to-bottom according to definition in hook, ensure that the stack should be handle in correct order.

## Parameters

**Elysia.macro** parameters to interact with the life cycle event as the following:

-   onParse
-   onTransform
-   onBeforeHandle
-   onAfterHandle
-   onError
-   onResponse
-   events - Life cycle store
    -   global: Life cycle of a global stack
    -   local: Life cycle of an inline hook (route)

Parameters start with **on** is a function to appends function into a life cycle stack.

While **events** is an actual stack that stores an order of the life-cycle event. You may mutate the stack directly or using the helper function provided by Elysia.

## Options

The life cycle function of an extension API accepts additional **options** to ensure control over life cycle events.

-   **options** (optional) - determine which stack
-   **function** - function to execute on the event

```typescript twoslash
import { Elysia } from 'elysia'

const plugin = new Elysia({ name: 'plugin' })
    .macro(({ onBeforeHandle }) => {
        return {
            hi(word: string) {
                onBeforeHandle(
                    { insert: 'before' }, // [!code ++]
                    () => {
                        console.log(word)
                    }
                )
            }
        }
    })
```

**Options** may accept the following parameter:

-   **insert**
    -   Where should the function be added
    -   value: **'before' | 'after'**
    -   @default: **'after'**
-   **stack**
    -   Determine which type of stack should be added
    -   value: **'global' | 'local'**
    -   @default: **'local'**


---

# patterns/mount.md


# Mount
WinterCG is a standard for web-interoperable runtimes. Supported by Cloudflare, Deno, Vercel Edge Runtime, Netlify Function, and various others, it allows web servers to run interoperably across runtimes that use Web Standard definitions like `Fetch`, `Request`, and `Response`.

Elysia is WinterCG compliant. We are optimized to run on Bun but also openly support other runtimes if possible.

In theory, this allows any framework or code that is WinterCG compliant to be run together, allowing frameworks like Elysia, Hono, Remix, Itty Router to run together in a simple function.

Adhering to this, we implemented the same logic for Elysia by introducing `.mount` method to run with any framework or code that is WinterCG compliant.

## Mount
To use **.mount**, [simply pass a `fetch` function](https://twitter.com/saltyAom/status/1684786233594290176):
```ts
import { Elysia } from 'elysia'

const app = new Elysia()
    .get('/', () => 'Hello from Elysia')
    .mount('/hono', hono.fetch)
```

A **fetch** function is a function that accepts a Web Standard Request and returns a Web Standard Response with the definition of:
```ts
// Web Standard Request-like object
// Web Standard Response
type fetch = (request: RequestLike) => Response
```

By default, this declaration is used by:
- Bun
- Deno
- Vercel Edge Runtime
- Cloudflare Worker
- Netlify Edge Function
- Remix Function Handler
- etc.

This allows you to execute all the aforementioned code in a single server environment, making it possible to interact seamlessly with Elysia. You can also reuse existing functions within a single deployment, eliminating the need for a reverse proxy to manage multiple servers.

If the framework also supports a **.mount** function, you can deeply nest a framework that supports it.
```ts
import { Elysia } from 'elysia'

const elysia = new Elysia()
    .get('/Hello from Elysia inside Hono inside Elysia')

const hono = new Hono()
    .get('/', (c) => c.text('Hello from Hono!'))
    .mount('/elysia', elysia.fetch)

const main = new Elysia()
    .get('/', () => 'Hello from Elysia')
    .mount('/hono', hono.fetch)
    .listen(3000)
```

## Reusing Elysia
Moreover, you can re-use multiple existing Elysia projects on your server.

```ts
import { Elysia } from 'elysia'

import A from 'project-a/elysia'
import B from 'project-b/elysia'
import C from 'project-c/elysia'

new Elysia()
    .mount(A)
    .mount(B)
    .mount(C)
```

If an instance passed to `mount` is an Elysia instance, it will be resolved with `use` automatically, providing type-safety and support for Eden by default.

This makes the possibility of an interoperable framework and runtime a reality.


---

# patterns/mvc.md


# MVC Pattern

Elysia is pattern agnostic framework, we the decision up to you and your team for coding patterns to use.

However, we found that several people are using the MVC pattern [(Model-View-Controller)](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) on Elysia, and found it's hard to decouple and handling with types.

This page is a guide to use Elysia with MVC pattern.

## Controller
1 Elysia instance = 1 controller.

**DO NOT** create a separate controller, use Elysia itself as a controller instead.

```typescript twoslash
const Controller = {
    hi(context: any) {}
}

const Service = {
    do1(v?: string) {},
    do2(v?: string) {}
}
// ---cut---
import { Elysia, t } from 'elysia'

// ❌ don't:
new Elysia()
    .get('/', Controller.hi)

// ✅ do:
new Elysia()
    // Get what you need
    .get('/', ({ query: { name } }) => {
        Service.do1(name)
        Service.do2(name)
    }, {
    	query: t.Object({
			name: t.String()
     	})
    })
```

Elysia does a lot to ensure type integrity, and if you pass an entire Context type to a controller, these might be the problems:
1. Elysia type is complex and heavily depends on plugin and multiple level of chaining.
2. Hard to type, Elysia type could change at anytime, especially with decorators, and store
3. Type casting may cause lost of type integrity or unable to ensure type and runtime code.
4. Harder for [Sucrose](/blog/elysia-10#sucrose) *(Elysia's "kind of" compiler)* to statically analyze your code

We recommended using object destructuring to extract what you need and pass it to **"Service"** instead.

By passing an entire `Controller.method` to Elysia is an equivalent of having 2 controllers passing data back and forth. It's against the design of framework and MVC pattern itself.

```typescript twoslash
const Service = {
    doStuff(stuff?: string) {
        return stuff
    }
}
// ---cut---
// ❌ don't:
import { Elysia, type Context } from 'elysia'

abstract class Controller {
    static root(context: Context<any, any>) {
        return Service.doStuff(context.stuff)
    }
}

new Elysia()
    .get('/', Controller.root)
```

Here's an example of what it looks like to do something similar in NestJS.
```typescript
// ❌ don't:
abstract class InternalController {
    static root(res: Response) {
        return Service.doStuff(res.stuff)
    }
}

@Controller()
export class AppController {
    constructor(private appService: AppService) {}

    @Get()
    root(@Res() res: Response) {
        return InternalController.root(res)
    }
}
```

Instead treat an Elysia instance as a controller itself.
```typescript twoslash
// @filename: service.ts
import { Elysia } from 'elysia'

export const HiService = new Elysia()
    .decorate({
        stuff: 'a',
        Hi: {
            doStuff(stuff: string) {
                return stuff
            }
        }
    })

// @filename: index.ts
// ---cut---
import { Elysia } from 'elysia'
import { HiService } from './service'

// ✅ do:
new Elysia()
    .use(HiService)
    .get('/', ({ Hi, stuff }) => {
        Hi.doStuff(stuff)
    })
```

If you would like to call or perform unit test on controller, use [Elysia.handle](/essential/route.html#handle).

```typescript twoslash
// @filename: service.ts
import { Elysia } from 'elysia'

export const HiService = new Elysia()
    .decorate({
        stuff: 'a',
        Hi: {
            doStuff(stuff: string) {
                return stuff
            }
        }
    })

// @filename: index.ts
// ---cut---
import { Elysia } from 'elysia'
import { HiService } from './service'

const app = new Elysia()
    .use(HiService)
    .get('/', ({ Hi, stuff }) => {
        Hi.doStuff(stuff)
    })

app.handle(new Request('http://localhost/'))
    .then(console.log)
```

Or even better, use [Eden](/eden/treaty/unit-test.html) with end-to-end type safety.

```typescript twoslash
// @filename: service.ts
import { Elysia } from 'elysia'

export const HiService = new Elysia()
    .decorate({
        stuff: 'a',
        Hi: {
            doStuff(stuff: string) {
                return stuff
            }
        }
    })

// @filename: index.ts
// ---cut---

import { Elysia } from 'elysia'
import { treaty } from '@elysiajs/eden'

import { HiService } from './service'

const AController = new Elysia()
    .use(HiService)
    .get('/', ({ Hi, stuff }) => Hi.doStuff(stuff))

const controller = treaty(AController)
const { data, error } = await controller.index.get()
```

## Service
Service is a set of utility/helper functions for each module, in our case, Elysia instance.

Any logic that can be decoupled from controller may be live inside a **Service**.

```typescript twoslash
import { Elysia, t } from 'elysia'

abstract class Service {
    static fibo(number: number): number {
        if(number < 2)
            return number

        return Service.fibo(number - 1) + Service.fibo(number - 2)
    }
}

new Elysia()
    .get('/fibo', ({ body }) => {
        return Service.fibo(body)
    }, {
        body: t.Numeric()
    })
```

If your service doesn't need to store a property, you may use `abstract class` and `static` instead to avoid allocating a class instance.

But if your service involves local mutation eg. caching, you may want to initiate an instance instead.

```typescript twoslash
import { Elysia, t } from 'elysia'

class Service {
    public cache = new Map<number, number>()

    fibo(number: number): number {
        if(number < 2)
            return number

        if(this.cache.has(number))
            return this.cache.get(number)!

        const a = this.fibo(number - 1)
        const b = this.fibo(number - 2)

        this.cache.set(number - 1, a)
        this.cache.set(number - 2, b)

        return a + b
    }
}

new Elysia()
    .decorate({
        Service: new Service()
    })
    .get('/fibo', ({ Service, body }) => {
        return Service.fibo(body)
    }, {
        body: t.Numeric()
    })
```

You may use [Elysia.decorate](/essential/context#decorate) to embed a class instance into Elysia, or not, it depends on your usecase.

Using [Elysia.decorate](/essential/context#decorate) is an equivalent of using **dependency injection** in NestJS:
```typescript
// Using dependency injection
@Controller()
export class AppController {
    constructor(service: Service) {}
}

// Using separate instance from dependency
const service = new Service()

@Controller()
export class AppController {
    constructor() {}
}
```

### Request Dependent Service

If your service is going to be used in multiple instances or may require some properties from the request, we recommend creating a dedicated Elysia instance as a **Service** instead.

Elysia handles [plugin deduplication](/essential/plugin.html#plugin-deduplication) by default, so you don't have to worry about performance, as it will be a singleton if you specify a **"name"** property.

```typescript twoslash
import { Elysia } from 'elysia'

const AuthService = new Elysia({ name: 'Service.Auth' })
    .derive({ as: 'scoped' }, ({ cookie: { session } }) => {
        return {
            Auth: {
                user: session.value
            }
        }
    })
    .macro(({ onBeforeHandle }) => ({
        isSignIn(value: boolean) {
            onBeforeHandle(({ Auth, error }) => {
                if (!Auth?.user || !Auth.user) return error(401)
            })
        }
    }))

const UserController = new Elysia()
    .use(AuthService)
    .guard({
        isSignIn: true
    })
    .get('/profile', ({ Auth: { user } }) => user)
```

## Model
Model or [DTO (Data Transfer Object)](https://en.wikipedia.org/wiki/Data_transfer_object) is handled by [Elysia.t (Validation)](/validation/overview.html#data-validation).

We recommended using [Elysia reference model](/validation/reference-model.html#reference-model) or creating an object or class of DTOs for each module.

1. Using Elysia's model reference
```typescript twoslash
import { Elysia, t } from 'elysia'

const AuthModel = new Elysia({ name: 'Model.Auth' })
    .model({
        'auth.sign': t.Object({
            username: t.String(),
            password: t.String({
                minLength: 5
            })
        })
    })

const UserController = new Elysia({ prefix: '/auth' })
    .use(AuthModel)
    .post('/sign-in', async ({ body, cookie: { session } }) => {
        return {
            success: true
        }
    }, {
        body: 'auth.sign'
    })
```

This allows approach provide several benefits.
1. Allow us to name a model and provide auto-completion.
2. Modify schema for later usage, or perform [remapping](/patterns/remapping.html#remapping).
3. Show up as "models" in OpenAPI compliance client, eg. Swagger.

## View
You may use Elysia HTML to do Template Rendering.

Elysia support JSX as template engine using [Elysia HTML plugin](/plugins/html)

You **may** create a rendering service or embedding view directly is up to you, but according to MVC pattern, you are likely to create a seperate service for handling view instead.

1. Embedding View directly, this may be useful if you have to render multiple view, eg. using [HTMX](https://htmx.org):
```tsx twoslash
import React from 'react'
// ---cut---
import { Elysia } from 'elysia'

new Elysia()
    .get('/', ({ query: { name } }) => {
        return (
            <h1>hello {name}</h1>
        )
    })
```

2. Dedicated View as a service:
```tsx twoslash
import React from 'react'
// ---cut---
import { Elysia, t } from 'elysia'

abstract class Render {
    static root(name?: string) {
        return <h1>hello {name}</h1>
    }
}

new Elysia()
    .get('/', ({ query: { name } }) => {
        return Render.root(name)
    }, {
    	query: t.Object({
			name: t.String()
		})
    })
```



---

# patterns/remapping.md


# Remapping
As the name suggest, this allow us to remap existing `state`, `decorate`, `model`, `derive` to anything we like to prevent name collision, or just wanting to rename a property.

By providing a function as a first parameters, the callback will accept current value, allowing us to remap the value to anything we like.
```ts
new Elysia()
    .state({
        a: "a",
        b: "b"
    })
    // Exclude b state
    .state(({ b, ...rest }) => rest)
```

This is useful when you have to deal with a plugin that has some duplicate name, allowing you to remap the name of the plugin:
```ts
new Elysia()
    .use(
        plugin
            .decorate(({ logger, ...rest }) => ({
                pluginLogger: logger,
                ...rest
            }))
    )
```

Remap function can be use with `state`, `decorate`, `model`, `derive` to helps you define a correct property name and preventing name collision.

## Affix
To provide a smoother experience, some plugins might have a lot of property value which can be overwhelming to remap one-by-one.

The **Affix** function, which consists of a **prefix** and **suffix**, allows us to effortlessly remap all properties of an instance, preventing the name collision of the plugin.

```ts
const setup = new Elysia({ name: 'setup' })
    .decorate({
        argon: 'a',
        boron: 'b',
        carbon: 'c'
    })

const app = new Elysia()
    .use(
        setup
            .prefix('decorator', 'setup')
    )
    .get('/', ({ setupCarbon }) => setupCarbon)
```

By default, **affix** will handle both runtime, type-level code automatically, remapping the property to camelCase as naming convention.

In some condition, you can also remap `all` property of the plugin:
```ts
const app = new Elysia()
    .use(
        setup
            .prefix('all', 'setup')
    )
    .get('/', ({ setupCarbon }) => setupCarbon)
```


---

# patterns/stream.md


# Stream
To return a response streaming out of the box by using a generator function with `yield` keyword.

```typescript twoslash
import { Elysia } from 'elysia'

const app = new Elysia()
	.get('/ok', function* () {
		yield 1
		yield 2
		yield 3
	})
```

This this example, we may stream a response by using `yield` keyword.

## Set headers
Elysia will defers returning response headers until the first chunk is yielded.

This allows us to set headers before the response is streamed.

```typescript twoslash
import { Elysia } from 'elysia'

const app = new Elysia()
	.get('/ok', function* ({ set }) {
		// This will set headers
		set.headers['x-name'] = 'Elysia'
		yield 1
		yield 2

		// This will do nothing
		set.headers['x-id'] = '1'
		yield 3
	})
```

Once the first chunk is yielded, Elysia will send the headers and the first chunk in the same response.

Setting headers after the first chunk is yielded will do nothing.

## Conditional Stream
If the response is returned without yield, Elysia will automatically convert stream to normal response instead.

```typescript twoslash
import { Elysia } from 'elysia'

const app = new Elysia()
	.get('/ok', function* ({ set }) {
		if (Math.random() > 0.5) return 'ok'

		yield 1
		yield 2
		yield 3
	})
```

This allows us to conditionally stream a response or return a normal response if necessary.

## Abort
While streaming a response, it's common that request may be cancelled before the response is fully streamed.

Elysia will automatically stop the generator function when the request is cancelled.

## Eden
Eden will will interpret a stream response as `AsyncGenerator` allowing us to use `for await` loop to consume the stream.

```typescript twoslash
import { Elysia } from 'elysia'
import { treaty } from '@elysiajs/eden'

const app = new Elysia()
	.get('/ok', function* () {
		yield 1
		yield 2
		yield 3
	})

const { data, error } = await treaty(app).ok.get()
if (error) throw error

for await (const chunk of data)
	console.log(chunk)
```


---

# patterns/unit-test.md


# Unit Test

Being WinterCG compliant, we can use Request / Response classes to test an Elysia server.

Elysia provides the **Elysia.handle** method, which accepts a Web Standard [Request](https://developer.mozilla.org/en-US/docs/Web/API/Request) and returns [Response](https://developer.mozilla.org/en-US/docs/Web/API/Response), simulating an HTTP Request.

Bun includes a built-in [test runner](https://bun.sh/docs/cli/test) that offers a Jest-like API through the `bun:test` module, facilitating the creation of unit tests.

Create **test/index.test.ts** in the root of project directory with the following:

```typescript twoslash
// test/index.test.ts
import { describe, expect, it } from 'bun:test'
import { Elysia } from 'elysia'

describe('Elysia', () => {
    it('return a response', async () => {
        const app = new Elysia().get('/', () => 'hi')

        const response = await app
            .handle(new Request('http://localhost/'))
            .then((res) => res.text())

        expect(response).toBe('hi')
    })
})
```

Then we can perform tests by running **bun test**

```bash
bun test
```

New requests to an Elysia server must be a fully valid URL, **NOT** a part of a URL.

The request must provide URL as the following:

| URL                   | Valid |
| --------------------- | ----- |
| http://localhost/user | ✅    |
| /user                 | ❌    |

We can also use other testing libraries like Jest or testing library to create Elysia unit tests.

## Eden Treaty test

We may use Eden Treaty to create an end-to-end type safety test for Elysia server as follows:

```typescript twoslash
// test/index.test.ts
import { describe, expect, it } from 'bun:test'
import { Elysia } from 'elysia'
import { treaty } from '@elysiajs/eden'

const app = new Elysia().get('/hello', 'hi')

const api = treaty(app)

describe('Elysia', () => {
    it('return a response', async () => {
        const { data, error } = await api.hello.get()

        expect(data).toBe('hi')
              // ^?
    })
})
```

See [Eden Treaty Unit Test](/eden/treaty/unit-test) for setup and more information.


---

# patterns/websocket.md


# WebSocket

WebSocket is a realtime protocol for communication between your client and server.

Unlike HTTP where our client repeatedly asking the website for information and waiting for a reply each time, WebSocket sets up a direct line where our client and server can send messages back and forth directly, making the conversation quicker and smoother without having to start over each message.

SocketIO is a popular library for WebSocket, but it is not the only one. Elysia uses [uWebSocket](https://github.com/uNetworking/uWebSockets) which Bun uses under the hood with the same API.

To use websocket, simply call `Elysia.ws()`:

```typescript twoslash
import { Elysia } from 'elysia'

new Elysia()
    .ws('/ws', {
        message(ws, message) {
            ws.send(message)
        }
    })
    .listen(3000)
```

## WebSocket message validation:

Same as normal route, WebSockets also accepts a **schema** object to strictly type and validate requests.

```typescript twoslash
import { Elysia, t } from 'elysia'

const app = new Elysia()
    .ws('/ws', {
        // validate incoming message
        body: t.Object({
            message: t.String()
        }),
        query: t.Object({
            id: t.String()
        }),
        message(ws, { message }) {
            // Get schema from `ws.data`
            const { id } = ws.data.query
            ws.send({
                id,
                message,
                time: Date.now()
            })
        }
    })
    .listen(3000)
```

WebSocket schema can validate the following:

-   **message** - An incoming message.
-   **query** - query string or URL parameters.
-   **params** - Path parameters.
-   **header** - Request's headers.
-   **cookie** - Request's cookie
-   **response** - Value returned from handler

By default Elysia will parse incoming stringified JSON message as Object for validation.

## Configuration

You can set Elysia constructor to set the Web Socket value.

```ts twoslash
import { Elysia } from 'elysia'

new Elysia({
    websocket: {
        idleTimeout: 30
    }
})
```

Elysia's WebSocket implementation extends Bun's WebSocket configuration, please refer to [Bun's WebSocket documentation](https://bun.sh/docs/api/websockets) for more information.

The following are a brief configuration from [Bun WebSocket](https://bun.sh/docs/api/websockets#create-a-websocket-server)

### perMessageDeflate

@default `false`

Enable compression for clients that support it.

By default, compression is disabled.

### maxPayloadLength

The maximum size of a message.

### idleTimeout

@default `120`

After a connection has not received a message for this many seconds, it will be closed.

### backpressureLimit

@default `16777216` (16MB)

The maximum number of bytes that can be buffered for a single connection.

### closeOnBackpressureLimit

@default `false`

Close the connection if the backpressure limit is reached.

## Methods

Below are the new methods that are available to the WebSocket route

## ws

Create a websocket handler

Example:

```typescript twoslash
import { Elysia } from 'elysia'

const app = new Elysia()
    .ws('/ws', {
        message(ws, message) {
            ws.send(message)
        }
    })
    .listen(3000)
```

Type:

```typescript
.ws(endpoint: path, options: Partial<WebSocketHandler<Context>>): this
```

endpoint: A path to exposed as websocket handler
options: Customize WebSocket handler behavior

## WebSocketHandler

WebSocketHandler extends config from [config](#configuration).

Below is a config which is accepted by `ws`.

## open

Callback function for new websocket connection.

Type:

```typescript
open(ws: ServerWebSocket<{
    // uid for each connection
    id: string
    data: Context
}>): this
```

## message

Callback function for incoming websocket message.

Type:

```typescript
message(
    ws: ServerWebSocket<{
        // uid for each connection
        id: string
        data: Context
    }>,
    message: Message
): this
```

`Message` type based on `schema.message`. Default is `string`.

## close

Callback function for closing websocket connection.

Type:

```typescript
close(ws: ServerWebSocket<{
    // uid for each connection
    id: string
    data: Context
}>): this
```

## drain

Callback function for the server is ready to accept more data.

Type:

```typescript
drain(
    ws: ServerWebSocket<{
        // uid for each connection
        id: string
        data: Context
    }>,
    code: number,
    reason: string
): this
```

## parse

`Parse` middleware to parse the request before upgrading the HTTP connection to WebSocket.

## beforeHandle

`Before Handle` middleware which execute before upgrading the HTTP connection to WebSocket.

Ideal place for validation.

## transform

`Transform` middleware which execute before validation.

## transformMessage

Like `transform`, but execute before validation of WebSocket message

## header

Additional headers to add before upgrade connection to WebSocket.


---

# plugins/bearer.md


# Bearer Plugin
Plugin for [elysia](https://github.com/elysiajs/elysia) for retrieving the Bearer token.

Install with:
```bash
bun add @elysiajs/bearer
```

Then use it:
```typescript
import { Elysia } from 'elysia'
import { bearer } from '@elysiajs/bearer'

const app = new Elysia()
    .use(bearer())
    .get('/sign', ({ bearer }) => bearer, {
        beforeHandle({ bearer, set }) {
            if (!bearer) {
                set.status = 400
                set.headers[
                    'WWW-Authenticate'
                ] = `Bearer realm='sign', error="invalid_request"`

                return 'Unauthorized'
            }
        }
    })
    .listen(3000)
```

This plugin is for retrieving a Bearer token specified in [RFC6750](https://www.rfc-editor.org/rfc/rfc6750#section-2).

This plugin DOES NOT handle authentication validation for your server. Instead, the plugin leaves the decision to developers to apply logic for handling validation check themselves.


---

# plugins/cors.md


# CORS Plugin
This plugin adds support for customizing [Cross-Origin Resource Sharing](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) behavior.

Install with:
```bash
bun add @elysiajs/cors
```

Then use it:
```typescript
import { Elysia } from 'elysia'
import { cors } from '@elysiajs/cors'

new Elysia()
    .use(cors())
    .listen(3000)
```

This will set Elysia to accept requests from any origin. 

## Config
Below is a config which is accepted by the plugin

### origin
@default `true`

Indicates whether the response can be shared with the requesting code from the given origins.

Value can be one of the following:
- **string** - Name of origin which will directly assign to [Access-Control-Allow-Origin](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Origin) header.
- **boolean** - If set to true, [Access-Control-Allow-Origin](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Origin) will be set to `*` (any origins)
- **RegExp** - Pattern to match request's URL, allowed if matched.
- **Function** - Custom logic to allow resource sharing, allow if `true` is returned.
    - Expected to have the type of:
    ```typescript
    cors(context: Context) => boolean | void
    ```
- **Array<string | RegExp | Function>** - iterate through all cases above in order, allowed if any of the values are `true`.

### allowedHeaders
@default `*`

Allowed headers for an incoming request.

Assign [Access-Control-Allow-Headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Headers) header.

Value can be one of the following:
- **string** - Expects either a single header or a comma-delimited string
    - eg: `'Content-Type, Authorization'`.
- **string[]** - Allow multiple HTTP headers.
    - eg: `['Content-Type', 'Authorization']`

### credentials
@default `true`

The Access-Control-Allow-Credentials response header tells browsers whether to expose the response to the frontend JavaScript code when the request's credentials mode [Request.credentials](https://developer.mozilla.org/en-US/docs/Web/API/Request/credentials) is `include`.

When a request's credentials mode [Request.credentials](https://developer.mozilla.org/en-US/docs/Web/API/Request/credentials) is `include`, browsers will only expose the response to the frontend JavaScript code if the Access-Control-Allow-Credentials value is true.

Credentials are cookies, authorization headers, or TLS client certificates.

Assign [Access-Control-Allow-Credentials](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Credentials) header.

### preflight
The preflight request is a request sent to check if the CORS protocol is understood and if a server is aware of using specific methods and headers.

Response with **OPTIONS** request with 3 HTTP request headers:
- **Access-Control-Request-Method**
- **Access-Control-Request-Headers**
- **Origin**

This config indicates if the server should respond to preflight requests.



---

# plugins/cron.md


# Cron Plugin

This plugin adds support for running cronjob in the Elysia server.

Install with:

```bash
bun add @elysiajs/cron
```

Then use it:

```typescript
import { Elysia } from 'elysia'
import { cron } from '@elysiajs/cron'

new Elysia()
    .use(
        cron({
            name: 'heartbeat',
            pattern: '*/10 * * * * *',
            run() {
                console.log('Heartbeat')
            }
        })
    )
    .listen(3000)
```

The above code will log `heartbeat` every 10 seconds.

## cron

Create a cronjob for the Elysia server.

type:

```
cron(config: CronConfig, callback: (Instance['store']) => void): this
```

`CronConfig` accepts the parameters specified below:

### name

Job name to register to `store`.

This will register the cron instance to `store` with a specified name, which can be used to reference in later processes eg. stop the job.

### pattern

Time to run the job as specified by [cron syntax](https://en.wikipedia.org/wiki/Cron) specified as below:

```
┌────────────── second (optional)
│ ┌──────────── minute
│ │ ┌────────── hour
│ │ │ ┌──────── day of the month
│ │ │ │ ┌────── month
│ │ │ │ │ ┌──── day of week
│ │ │ │ │ │
* * * * * *
```

This can be generated by tools like [Crontab Guru](https://crontab.guru/)



---

# plugins/graphql-apollo.md


# GraphQL Apollo Plugin
Plugin for [elysia](https://github.com/elysiajs/elysia) for using GraphQL Apollo.

Install with:
```bash
bun add graphql @elysiajs/apollo @apollo/server
```

Then use it:
```typescript
import { Elysia } from 'elysia'
import { apollo, gql } from '@elysiajs/apollo'

const app = new Elysia()
    .use(
        apollo({
            typeDefs: gql`
                type Book {
                    title: String
                    author: String
                }

                type Query {
                    books: [Book]
                }
            `,
            resolvers: {
                Query: {
                    books: () => {
                        return [
                            {
                                title: 'Elysia',
                                author: 'saltyAom'
                            }
                        ]
                    }
                }
            }
        })
    )
    .listen(3000)
```

Accessing `/graphql` should show Apollo GraphQL playground work with.

## Context
Because Elysia is based on Web Standard Request and Response which is different from Node's `HttpRequest` and `HttpResponse` that Express uses, results in `req, res` being undefined in context.

Because of this, Elysia replaces both with `context` like route parameters.
```typescript
const app = new Elysia()
    .use(
        apollo({
            typeDefs,
            resolvers,
            context: async ({ request }) => {
                const authorization = request.headers.get('Authorization')

                return {
                    authorization
                }
            }
        })
    )
    .listen(3000)
```


## Config
This plugin extends Apollo's [ServerRegistration](https://www.apollographql.com/docs/apollo-server/api/apollo-server/#options) (which is `ApolloServer`'s' constructor parameter).

Below are the extended parameters for configuring Apollo Server with Elysia.
### path
@default "/graphql"

Path to expose Apollo Server.

### enablePlayground
@default "process.env.ENV !== 'production'

Determine whether should Apollo should provide Apollo Playground.


---

# plugins/graphql-yoga.md


# GraphQL Yoga Plugin
This plugin integrates GraphQL yoga with Elysia

Install with:
```bash
bun add @elysiajs/graphql-yoga
```

Then use it:
```typescript
import { Elysia } from 'elysia'
import { yoga } from '@elysiajs/graphql-yoga'

const app = new Elysia()
    .use(
        yoga({
            typeDefs: /* GraphQL */`
                type Query {
                    hi: String
                }
            `,
            resolvers: {
                Query: {
                    hi: () => 'Hello from Elysia'
                }
            }
        })
    )
    .listen(3000)
```

Accessing `/graphql` in the browser (GET request) would show you a GraphiQL instance for the GraphQL-enabled Elysia server.

optional: you can install a custom version of optional peer dependencies as well:
```bash
bun add graphql graphql-yoga
```

## Resolver
Elysia uses [Mobius](https://github.com/saltyaom/mobius) to infer type from **typeDefs** field automatically, allowing you to get full type-safety and auto-complete when typing **resolver** types.

## Context
You can add custom context to the resolver function by adding **context**
```ts
import { Elysia } from 'elysia'
import { yoga } from '@elysiajs/graphql-yoga'

const app = new Elysia()
    .use(
        yoga({
            typeDefs: /* GraphQL */`
                type Query {
                    hi: String
                }
            `,
            context: {
                name: 'Mobius'
            },
            // If context is a function on this doesn't present
            // for some reason it won't infer context type
            useContext(_) {},
            resolvers: {
                Query: {
                    hi: async (parent, args, context) => context.name
                }
            }
        })
    )
    .listen(3000)
```

## Config
This plugin extends [GraphQL Yoga's createYoga options, please refer to the GraphQL Yoga documentation](https://the-guild.dev/graphql/yoga-server/docs) with inlining `schema` config to root.

Below is a config which is accepted by the plugin

### path
@default `/graphql`

Endpoint to expose GraphQL handler


---

# plugins/html.md


# HTML Plugin

Allows you to use [JSX](#jsx) and HTML with proper headers and support.

Install with:

```bash
bun add @elysiajs/html
```

Then use it:

```tsx
import { Elysia } from 'elysia'
import { html } from '@elysiajs/html'

new Elysia()
    .use(html())
    .get(
        '/html',
        () => `
            <html lang='en'>
                <head>
                    <title>Hello World</title>
                </head>
                <body>
                    <h1>Hello World</h1>
                </body>
            </html>`
    )
    .get('/jsx', () => (
        <html lang='en'>
            <head>
                <title>Hello World</title>
            </head>
            <body>
                <h1>Hello World</h1>
            </body>
        </html>
    ))
    .listen(3000)
```

This plugin will automatically add `Content-Type: text/html; charset=utf8` header to the response, add `<!doctype html>`, and convert it into a Response object.

## JSX
Elysia HTML is based on [@kitajs/html](https://github.com/kitajs/html) allowing us to define JSX to string in compile time to achieve high performance.

Name your file that needs to use JSX to end with affix **"x"**:
- .js -> .jsx
- .ts -> .tsx

To register the TypeScript type, please append the following to **tsconfig.json**:
```jsonc
// tsconfig.json
{
    "compilerOptions": {
        "jsx": "react",
        "jsxFactory": "Html.createElement",
        "jsxFragmentFactory": "Html.Fragment"
    }
}
```

That's it, now you can JSX as your template engine:
```tsx
import { Elysia } from 'elysia'
import { html } from '@elysiajs/html' // [!code ++]

new Elysia()
    .use(html()) // [!code ++]
    .get('/', () => (
        <html lang="en">
            <head>
                <title>Hello World</title>
            </head>
            <body>
                <h1>Hello World</h1>
            </body>
        </html>
    ))
    .listen(3000)
```

## XSS
Elysia HTML is based use of the Kita HTML plugin to detect possible XSS attacks in compile time.

You can use a dedicated `safe` attribute to sanitize user value to prevent XSS vulnerability.
```tsx
import { Elysia, t } from 'elysia'
import { html } from '@elysiajs/html'

new Elysia()
    .use(html())
    .post('/', ({ body }) => (
        <html lang="en">
            <head>
                <title>Hello World</title>
            </head>
            <body>
                <h1 safe>{body}</h1>
            </body>
        </html>
    ), {
        body: t.String()
    })
    .listen(3000)
```

However, when are building a large-scale app, it's best to have a type reminder to detect possible XSS vulnerabilities in your codebase.

To add a type-safe reminder, please install:
```sh
bun add @kitajs/ts-html-plugin
```

Then appends the following **tsconfig.json**
```jsonc
// tsconfig.json
{
    "compilerOptions": {
        "jsx": "react",
        "jsxFactory": "Html.createElement",
        "jsxFragmentFactory": "Html.Fragment",
        "plugins": [{ "name": "@kitajs/ts-html-plugin" }]
    }
}
```

## Options

### contentType

-   Type: `string`
-   Default: `'text/html; charset=utf8'`

The content-type of the response.

### autoDetect

-   Type: `boolean`
-   Default: `true`

Whether to automatically detect HTML content and set the content-type.

### autoDoctype

-   Type: `boolean | 'full'`
-   Default: `true`

Whether to automatically add `<!doctype html>` to a response starting with `<html>`, if not found.

Use `full` to also automatically add doctypes on responses returned without this plugin

```ts
// without the plugin
app.get('/', () => '<html></html>')

// With the plugin
app.get('/', ({ html }) => html('<html></html>'))
```

### isHtml

-   Type: `(value: string) => boolean`
-   Default: `isHtml` (exported function)

The function is used to detect if a string is a html or not. Default implementation if length is greater than 7, starts with `<` and ends with `>`.

Keep in mind there's no real way to validate HTML, so the default implementation is a best guess.


---

# plugins/jwt.md


# JWT Plugin
This plugin adds support for using JWT in Elysia handler

Install with:
```bash
bun add @elysiajs/jwt
```

Then use it:
```typescript
import { Elysia } from 'elysia'
import { jwt } from '@elysiajs/jwt'

const app = new Elysia()
    .use(
        jwt({
            name: 'jwt',
            secret: 'Fischl von Luftschloss Narfidort'
        })
    )
    .get('/sign/:name', async ({ jwt, cookie: { auth }, params }) => {
        auth.set({
            value: await jwt.sign(params),
            httpOnly: true,
            maxAge: 7 * 86400,
            path: '/profile',
        })

        return `Sign in as ${auth.value}`
    })
    .get('/profile', async ({ jwt, set, cookie: { auth } }) => {
        const profile = await jwt.verify(auth.value)

        if (!profile) {
            set.status = 401
            return 'Unauthorized'
        }

        return `Hello ${profile.name}`
    })
    .listen(3000)
```

## Config
This plugin extends config from [jose](https://github.com/panva/jose).

Below is a config that is accepted by the plugin.

### name
Name to register `jwt` function as.

For example, `jwt` function will be registered with a custom name.
```typescript
app
    .use(
        jwt({
            name: 'myJWTNamespace',
            secret: process.env.JWT_SECRETS!
        })
    )
    .get('/sign/:name', ({ myJWTNamespace, params }) => {
        return myJWTNamespace.sign(params)
    })
```

Because some might need to use multiple `jwt` with different configs in a single server, explicitly registering the JWT function with a different name is needed.

### secret
The private key to sign JWT payload with.

### schema
Type strict validation for JWT payload.



---

# plugins/opentelemetry.md


To start using OpenTelemetry, install `@elysiajs/opentelemetry` and apply plugin to any instance.

```typescript twoslash
import { Elysia } from 'elysia'
import { opentelemetry } from '@elysiajs/opentelemetry'

import { BatchSpanProcessor } from '@opentelemetry/sdk-trace-node'
import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-proto'

new Elysia()
	.use(
		opentelemetry({
			spanProcessors: [
				new BatchSpanProcessor(
					new OTLPTraceExporter()
				)
			]
		})
	)
```

![jaeger showing collected trace automatically](/blog/elysia-11/jaeger.webp)

Elysia OpenTelemetry is will **collect span of any library compatible OpenTelemetry standard**, and will apply parent and child span automatically.

In the code above, we apply `Prisma` to trace how long each query took.

By applying OpenTelemetry, Elysia will then:
- collect telemetry data
- Grouping relevant lifecycle together
- Measure how long each function took
- Instrument HTTP request and response
- Collect error and exception

You may export telemetry data to Jaeger, Zipkin, New Relic, Axiom or any other OpenTelemetry compatible backend.

![axiom showing collected trace from OpenTelemetry](/blog/elysia-11/axiom.webp)

Here's an example of exporting telemetry to [Axiom](https://axiom.co)
```typescript twoslash
const Bun = {
	env: {
		AXIOM_TOKEN: '',
		AXIOM_DATASET: ''
	}
}
// ---cut---
import { Elysia } from 'elysia'
import { opentelemetry } from '@elysiajs/opentelemetry'

import { BatchSpanProcessor } from '@opentelemetry/sdk-trace-node'
import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-proto'

new Elysia()
	.use(
		opentelemetry({
			spanProcessors: [
				new BatchSpanProcessor(
					new OTLPTraceExporter({
						url: 'https://api.axiom.co/v1/traces', // [!code ++]
						headers: { // [!code ++]
						    Authorization: `Bearer ${Bun.env.AXIOM_TOKEN}`, // [!code ++]
						    'X-Axiom-Dataset': Bun.env.AXIOM_DATASET // [!code ++]
						} // [!code ++]
					})
				)
			]
		})
	)
```

## OpenTelemetry SDK
Elysia OpenTelemetry is for applying OpenTelemetry to Elysia server only.

You may use OpenTelemetry SDK normally, and the span is run under Elysia's request span, it will be automatically appear in Elysia trace.

However, we also provide a `getTracer`, and `record` utility to collect span from any part of your application.

```typescript twoslash
const db = {
	query(query: string) {
		return new Promise<unknown>((resolve) => {
			resolve('')
		})
	}
}
// ---cut---
import { Elysia } from 'elysia'
import { record } from '@elysiajs/opentelemetry'

export const plugin = new Elysia()
	.get('', () => {
		return record('database.query', () => {
			return db.query('SELECT * FROM users')
		})
	})
```

## Record utility
`record` is an equivalent to OpenTelemetry's `startActiveSpan` but it will handle auto-closing and capture exception automatically.

You may think of `record` as a label for your code that will be shown in trace.

### Prepare your codebase for observability
Elysia OpenTelemetry will group lifecycle and read the **function name** of each hook as the name of the span.

It's a good time to **name your function**.

If your hook handler is an arrow function, you may refactor it to named function to understand the trace better otherwise, your trace span will be named as `anonymous`.

```typescript
const bad = new Elysia()
	// ⚠️ span name will be anonymous
	.derive(async ({ cookie: { session } }) => {
		return {
			user: await getProfile(session)
		}
	})

const good = new Elysia()
	// ✅ span name will be getProfile
	.derive(async function getProfile({ cookie: { session } }) {
		return {
			user: await getProfile(session)
		}
	})
```

## Config
This plugin extends OpenTelemetry SDK parameters options.

Below is a config which is accepted by the plugin

### autoDetectResources - boolean
Detect resources automatically from the environment using the default resource detectors.

default: `true`

### contextManager - ContextManager
Use a custom context manager.

default: `AsyncHooksContextManager`

### textMapPropagator - TextMapPropagator
Use a custom propagator.

default: `CompositePropagator` using W3C Trace Context and Baggage

### metricReader - MetricReader
Add a MetricReader that will be passed to the MeterProvider.

### views - View[]
A list of views to be passed to the MeterProvider.

Accepts an array of View-instances. This parameter can be used to configure explicit bucket sizes of histogram metrics.

### instrumentations - (Instrumentation | Instrumentation[])[]
Configure instrumentations.

By default `getNodeAutoInstrumentations` is enabled, if you want to enable them you can use either metapackage or configure each instrumentation individually.

default: `getNodeAutoInstrumentations()`

### resource - IResource
Configure a resource.

Resources may also be detected by using the autoDetectResources method of the SDK.

### resourceDetectors - Array<Detector | DetectorSync>
Configure resource detectors. By default, the resource detectors are [envDetector, processDetector, hostDetector]. NOTE: In order to enable the detection, the parameter autoDetectResources has to be true.

If resourceDetectors was not set, you can also use the environment variable OTEL_NODE_RESOURCE_DETECTORS to enable only certain detectors, or completely disable them:

- env
- host
- os
- process
- serviceinstance (experimental)
- all - enable all resource detectors above
- none - disable resource detection

For example, to enable only the env, host detectors:

```bash
export OTEL_NODE_RESOURCE_DETECTORS="env,host"
```

### sampler - Sampler
Configure a custom sampler. By default, all traces will be sampled.

### serviceName - string
Namespace to be identify as.

### spanProcessors - SpanProcessor[]
An array of span processors to register to the tracer provider.

### traceExporter - SpanExporter
Configure a trace exporter. If an exporter is configured, it will be used with a `BatchSpanProcessor`.

If an exporter OR span processor is not configured programmatically, this package will auto setup the default otlp exporter with http/protobuf protocol with a BatchSpanProcessor.

### spanLimits - SpanLimits
Configure tracing parameters. These are the same trace parameters used to configure a tracer.


---

# plugins/overview.md


# Overview

Elysia is designed to be modular and lightweight.

Following the same idea as Arch Linux (btw, I use Arch):

> Design decisions are made on a case-by-case basis through developer consensus

This is to ensure developers end up with a performant web server they intend to create. By extension, Elysia includes pre-built common pattern plugins for convenient developer usage:

## Official plugins:

-   [Bearer](/plugins/bearer) - retrieve [Bearer](https://swagger.io/docs/specification/authentication/bearer-authentication/) token automatically
-   [CORS](/plugins/cors) - set up [Cross-origin resource sharing (CORS)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
-   [Cron](/plugins/cron) - set up [cron](https://en.wikipedia.org/wiki/Cron) job
-   [Eden](/eden/overview) - end-to-end type safety client for Elysia
-   [GraphQL Apollo](/plugins/graphql-apollo) - run [Apollo GraphQL](https://www.apollographql.com/) on Elysia
-   [GraphQL Yoga](/plugins/graphql-yoga) - run [GraphQL Yoga](https://github.com/dotansimha/graphql-yoga) on Elysia
-   [HTML](/plugins/html) - handle HTML responses
-   [JWT](/plugins/jwt) - authenticate with [JWTs](https://jwt.io/)
-   [OpenTelemetry](/plugins/opentelemetry) - add support for OpenTelemetry
-   [Server Timing](/plugins/server-timing) - audit performance bottlenecks with the [Server-Timing API](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Server-Timing)
-   [Static](/plugins/static) - serve static files/folders
-   [Stream](/plugins/stream) - integrate response streaming and [server-sent events (SSEs)](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events)
-   [Swagger](/plugins/swagger) - generate [Swagger](https://swagger.io/) documentation
-   [tRPC](/plugins/trpc) - support [tRPC](https://trpc.io/)
-   [WebSocket](/patterns/websocket) - support [WebSockets](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket)

## Community plugins:

-   [BunSai](https://github.com/levii-pires/bunsai2) - full-stack agnostic framework for the web, built upon Bun and Elysia
-   [Create ElysiaJS](https://github.com/kravetsone/create-elysiajs) - scaffolding your Elysia project with the environment with easy (help with ORM, Linters and Plugins)!
-   [Lucia Auth](https://github.com/pilcrowOnPaper/lucia) - authentication, simple and clean
-   [Elysia Clerk](https://github.com/wobsoriano/elysia-clerk) - unofficial Clerk authentication plugin
-   [Elysia Polyfills](https://github.com/bogeychan/elysia-polyfills) - run Elysia ecosystem on Node.js and Deno
-   [Vite server](https://github.com/kravetsone/elysia-vite-server) - plugin which start and decorate [`vite`](https://vitejs.dev/) dev server in `development` and in `production` mode serve static (if it needed)
-   [Vite](https://github.com/timnghg/elysia-vite) - serve entry HTML file with Vite's scripts injected
-   [Nuxt](https://github.com/trylovetom/elysiajs-nuxt) - easily integrate elysia with nuxt!
-   [Remix](https://github.com/kravetsone/elysia-remix) - use [Remix](https://remix.run/) with `HMR` support (powered by [`vite`](https://vitejs.dev/))! Close a really long-standing plugin request [#12](https://github.com/elysiajs/elysia/issues/12)
-   [Connect middleware](https://github.com/kravetsone/elysia-connect-middleware) - plugin which allows you to use [`express`](https://www.npmjs.com/package/express)/[`connect`](https://www.npmjs.com/package/connect) middleware directly in Elysia!
-   [Elysia Helmet](https://github.com/DevTobias/elysia-helmet) - secure Elysia apps with various HTTP headers
-   [Vite Plugin SSR](https://github.com/timnghg/elysia-vite-plugin-ssr) - Vite SSR plugin using Elysia server
-   [OAuth 2.0](https://github.com/kravetsone/elysia-oauth2) - An plugin for [OAuth 2.0](https://en.wikipedia.org/wiki/OAuth) Authorization Flow with more than **42** providers and **type-safety**!
-   [OAuth2](https://github.com/bogeychan/elysia-oauth2) - handle OAuth 2.0 authorization code flow
-   [Elysia OpenID Client](https://github.com/macropygia/elysia-openid-client) - OpenID client based on [openid-client](https://github.com/panva/node-openid-client)
-   [Rate Limit](https://github.com/rayriffy/elysia-rate-limit) - simple, lightweight rate limiter
-   [Logysia](https://github.com/tristanisham/logysia) - classic logging middleware
-   [Logestic](https://github.com/cybercoder-naj/logestic) - An advanced and customisable logging library for ElysiaJS
-   [Logger](https://github.com/bogeychan/elysia-logger) - [pino](https://github.com/pinojs/pino)-based logging middleware
-   [Elylog](https://github.com/eajr/elylog) - simple stdout logging library with some customization
-   [Elysia Lambda](https://github.com/TotalTechGeek/elysia-lambda) - deploy on AWS Lambda
-   [Decorators](https://github.com/gaurishhs/elysia-decorators) - use TypeScript decorators
-   [Autoload](https://github.com/kravetsone/elysia-autoload) - filesystem router based on a directory structure that generates types for [Eden](https://elysiajs.com/eden/overview.html) with [`Bun.build`](https://github.com/kravetsone/elysia-autoload?tab=readme-ov-file#bun-build-usage) support
-   [Msgpack](https://github.com/kravetsone/elysia-msgpack) - allows you to work with [MessagePack](https://msgpack.org)
    [XML](https://github.com/kravetsone/elysia-xml) - allows you to work with XML
-   [Autoroutes](https://github.com/wobsoriano/elysia-autoroutes) - filesystem routes
-   [Group Router](https://github.com/itsyoboieltr/elysia-group-router) - filesystem and folder-based router for groups
-   [Basic Auth](https://github.com/itsyoboieltr/elysia-basic-auth) - basic HTTP authentication
-   [ETag](https://github.com/bogeychan/elysia-etag) - automatic HTTP [ETag](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/ETag) generation
-   [Basic Auth](https://github.com/eelkevdbos/elysia-basic-auth) - basic HTTP authentication (using `request` event)
-   [i18n](https://github.com/eelkevdbos/elysia-i18next) - [i18n](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/i18n) wrapper based on [i18next](https://www.i18next.com/)
-   [Elysia Request ID](https://github.com/gtramontina/elysia-requestid) - add/forward request IDs (`X-Request-ID` or custom)
-   [Elysia HTMX](https://github.com/gtramontina/elysia-htmx) - context helpers for [HTMX](https://htmx.org/)
-   [Elysia HMR HTML](https://github.com/gtrabanco/elysia-hmr-html) - reload HTML files when changing any file in a directory
-   [Elysia Inject HTML](https://github.com/gtrabanco/elysia-inject-html) - inject HTML code in HTML files
-   [Elysia HTTP Error](https://github.com/yfrans/elysia-http-error) - return HTTP errors from Elysia handlers
-   [Elysia Http Status Code](https://github.com/sylvain12/elysia-http-status-code) - integrate HTTP status codes
-   [NoCache](https://github.com/gaurishhs/elysia-nocache) - disable caching
-   [Elysia Tailwind](https://github.com/gtramontina/elysia-tailwind) - compile [Tailwindcss](https://tailwindcss.com/) in a plugin.
-   [Elysia Compression](https://github.com/gusb3ll/elysia-compression) - compress response
-   [Elysia IP](https://github.com/gaurishhs/elysia-ip) - get the IP Address
-   [OAuth2 Server](https://github.com/myazarc/elysia-oauth2-server) - developing an OAuth2 Server with Elysia
-   [Elysia Flash Messages](https://github.com/gtramontina/elysia-flash-messages) - enable flash messages
-   [Elysia AuthKit](https://github.com/gtramontina/elysia-authkit) - unnoficial [WorkOS' AuthKit](https://www.authkit.com/) authentication
-   [Elysia Error Handler](https://github.com/gtramontina/elysia-error-handler) - simpler error handling
-   [Elysia env](https://github.com/yolk-oss/elysia-env) - typesafe environment variables with typebox
-   [Elysia Drizzle Schema](https://github.com/Edsol/elysia-drizzle-schema) - Helps to use Drizzle ORM schema inside elysia swagger model.
-   [Unify-Elysia](https://github.com/qlaffont/unify-elysia) - Unify error code for Elysia
-   [Unify-Elysia-GQL](https://github.com/qlaffont/unify-elysia-gql) - Unify error code for Elysia GraphQL Server (Yoga & Apollo)
-   [Elysia Auth Drizzle](https://github.com/qlaffont/elysia-auth-drizzle) - Library who handle authentification with JWT (Header/Cookie/QueryParam).
-   [graceful-server-elysia](https://github.com/qlaffont/graceful-server-elysia) - Library inspired by [graceful-server](https://github.com/gquittet/graceful-server).
-   [Logixlysia](https://github.com/PunGrumpy/logixlysia) - A beautiful and simple logging middleware for ElysiaJS with colors and timestamps.
-   [Elysia Fault](https://github.com/vitorpldev/elysia-fault) - A simple and customizable error handling middleware with the possibility of creating your own HTTP errors
-   [Elysia Compress](https://github.com/vermaysha/elysia-compress) - ElysiaJS plugin to compress responses inspired by [@fastify/compress](https://github.com/fastify/fastify-compress)
-   [@labzzhq/compressor](https://github.com/labzzhq/compressor/) - Compact Brilliance, Expansive Results: HTTP Compressor for Elysia and Bunnyhop with gzip, deflate and brotli support.
-   [Elysia Accepts](https://github.com/morigs/elysia-accepts) - Elysia plugin for accept headers parsing and content negotiation
-   [Elysia Compression](https://github.com/chneau/elysia-compression) - Elysia plugin for compressing responses
-   [Elysia Logger](https://github.com/chneau/elysia-logger) - Elysia plugin for logging HTTP requests and responses inspired by [hono/logger](https://hono.dev/docs/middleware/builtin/logger)
-   [Elysia CQRS](https://github.com/jassix/elysia-cqrs) - Elysia plugin for CQRS pattern

## Complementaray projects:
-   [prismabox](https://github.com/m1212e/prismabox) - Generator for typebox schemes based on your database models, works well with elysia


---

# plugins/server-timing.md


# Server Timing Plugin
This plugin adds support for auditing performance bottlenecks with Server Timing API

Install with:
```bash
bun add @elysiajs/server-timing
```

Then use it:
```typescript
import { Elysia } from 'elysia'
import { serverTiming } from '@elysiajs/server-timing'

new Elysia()
    .use(serverTiming())
    .get('/', () => 'hello')
    .listen(3000)
```

Server Timing then will append header 'Server-Timing' with log duration, function name, and detail for each life-cycle function.

To inspect, open browser developer tools > Network > [Request made through Elysia server] > Timing.

![Developer tools showing Server Timing screenshot](/assets/server-timing.webp)

Now you can effortlessly audit the performance bottleneck of your server.

## Config
Below is a config which is accepted by the plugin

### enabled
@default `NODE_ENV !== 'production'`

Determine whether or not Server Timing should be enabled

### allow
@default `undefined`

A condition whether server timing should be log

### trace
@default `undefined`

Allow Server Timing to log specified life-cycle events:

Trace accepts objects of the following:
- request: capture duration from request
- parse: capture duration from parse
- transform: capture duration from transform
- beforeHandle: capture duration from beforeHandle
- handle: capture duration from the handle
- afterHandle: capture duration from afterHandle
- total: capture total duration from start to finish

## Pattern
Below you can find the common patterns to use the plugin.

- [Allow Condition](#allow-condition)

## Allow Condition
You may disable Server Timing on specific routes via `allow` property

```ts
import { Elysia } from 'elysia'
import { serverTiming } from '@elysiajs/server-timing'

new Elysia()
    .use(
        serverTiming({
            allow: ({ request }) => {
                return new URL(request.url).pathname !== '/no-trace'
            }
        })
    )
```


---

# plugins/static.md


# Static Plugin
This plugin can serve static files/folders for Elysia Server

Install with:
```bash
bun add @elysiajs/static
```

Then use it:
```typescript
import { Elysia } from 'elysia'
import { staticPlugin } from '@elysiajs/static'

new Elysia()
    .use(staticPlugin())
    .listen(3000)
```

By default, the static plugin default folder is `public`, and registered with `/public` prefix.

Suppose your project structure is:
```
| - src
  | - index.ts
| - public
  | - takodachi.png
  | - nested
    | - takodachi.png
```

The available path will become:
- /public/takodachi.png
- /public/nested/takodachi.png

## Config
Below is a config which is accepted by the plugin

### assets
@default `"public"`

Path to the folder to expose as static

### prefix
@default `"/public"`

Path prefix to register public files

### ignorePatterns
@default `[]`

List of files to ignore from serving as static files

### staticLimits
@default `1024`

By default, the static plugin will register paths to the Router with a static name, if the limits are exceeded, paths will be lazily added to the Router to reduce memory usage.
Tradeoff memory with performance.

### alwaysStatic
@default `false`

If set to true, static files will path will be registered to Router skipping the `staticLimits`.

### headers
@default `{}`

Set response headers of files

## Pattern
Below you can find the common patterns to use the plugin.

- [Single File](#single-file)

## Single file
Suppose you want to return just a single file, you can use `Bun.file` instead of using the static plugin
```typescript
new Elysia()
    .get('/file', () => Bun.file('public/takodachi.png'))
```


---

# plugins/stream.md


# Stream Plugin

::: warning
This plugin is in maintenance mode and will not receive new features. We recommend using the [Generator Stream instead](/patterns/stream)
:::

This plugin adds support for streaming response or sending Server-Sent Event back to the client.

Install with:
```bash
bun add @elysiajs/stream
```

Then use it:
```typescript
import { Elysia } from 'elysia'
import { Stream } from '@elysiajs/stream'

new Elysia()
    .get('/', () => new Stream(async (stream) => {
        stream.send('hello')

        await stream.wait(1000)
        stream.send('world')

        stream.close()
    }))
    .listen(3000)
```

By default, `Stream` will return `Response` with `content-type` of `text/event-stream; charset=utf8`.

## Constructor
Below is the constructor parameter accepted by `Stream`:
1. Stream:
    - Automatic: Automatically stream response from a provided value
        - Iterable
        - AsyncIterable
        - ReadableStream
        - Response
    - Manual: Callback of `(stream: this) => unknown` or `undefined`
2. Options: `StreamOptions`
    - [event](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events#event): A string identifying the type of event described
    - [retry](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events#retry): The reconnection time in milliseconds

## Method
Below is the method provided by `Stream`:

### send
Enqueue data to stream to send back to the client

### close
Close the stream

### wait
Return a promise that resolves in the provided value in ms

### value
Inner value of the `ReadableStream`

## Pattern
Below you can find the common patterns to use the plugin.
- [OpenAI](#openai)
- [Fetch Stream](#fetch-stream)
- [Server Sent Event](#server-sent-event)

## OpenAI
Automatic mode is triggered when the parameter is either `Iterable` or `AsyncIterable` streaming the response back to the client automatically.

Below is an example of integrating ChatGPT into Elysia.

```ts
new Elysia()
    .get(
        '/ai',
        ({ query: { prompt } }) =>
            new Stream(
                openai.chat.completions.create({
                    model: 'gpt-3.5-turbo',
                    stream: true,
                    messages: [{
                        role: 'user',
                        content: prompt
                    }]
                })
            )
    )
```

By default [openai](https://npmjs.com/package/openai) chatGPT completion returns `AsyncIterable` so you should be able to wrap the OpenAI in `Stream`.

## Fetch Stream
You can pass a fetch from an endpoint that returns the stream to proxy a stream.

This is useful for those endpoints that use AI text generation since you can proxy it directly, eg. [Cloudflare AI](https://developers.cloudflare.com/workers-ai/models/llm/#examples---chat-style-with-system-prompt-preferred).
```ts
const model = '@cf/meta/llama-2-7b-chat-int8'
const endpoint = `https://api.cloudflare.com/client/v4/accounts/${process.env.ACCOUNT_ID}/ai/run/${model}`

new Elysia()
    .get('/ai', ({ query: { prompt } }) =>
        fetch(endpoint, {
            method: 'POST',
            headers: {
                authorization: `Bearer ${API_TOKEN}`,
                'content-type': 'application/json'
            },
            body: JSON.stringify({
                messages: [
                    { role: 'system', content: 'You are a friendly assistant' },
                    { role: 'user', content: prompt }
                ]
            })
        })
    )
```

## Server Sent Event
Manual mode is triggered when the parameter is either `callback` or `undefined`, allowing you to control the stream.

### callback-based
Below is an example of creating a Server-Sent Event endpoint using a constructor callback

```ts
new Elysia()
    .get('/source', () =>
        new Stream((stream) => {
            const interval = setInterval(() => {
                stream.send('hello world')
            }, 500)

            setTimeout(() => {
                clearInterval(interval)
                stream.close()
            }, 3000)
        })
    )
```

### value-based
Below is an example of creating a Server-Sent Event endpoint using a value-based

```ts
new Elysia()
    .get('/source', () => {
        const stream = new Stream()

        const interval = setInterval(() => {
            stream.send('hello world')
        }, 500)

        setTimeout(() => {
            clearInterval(interval)
            stream.close()
        }, 3000)

        return stream
    })
```

Both callback-based and value-based streams work in the same way but with different syntax for your preference.


---

# plugins/swagger.md


# Swagger Plugin
This plugin generates a Swagger endpoint for an Elysia server

Install with:
```bash
bun add @elysiajs/swagger
```

Then use it:
```typescript
import { Elysia } from 'elysia'
import { swagger } from '@elysiajs/swagger'

new Elysia()
    .use(swagger())
    .get('/', () => 'hi')
    .post('/hello', () => 'world')
    .listen(3000)
```

Accessing `/swagger` would show you a Swagger UI with the generated endpoint documentation from the Elysia server.

## Config
Below is a config which is accepted by the plugin

### provider
@default `scalar`

UI Provider for documentation. Default to Scalar.

### scalar
Configuration for customizing Scalar.

Please refer to the [Scalar config](https://github.com/scalar/scalar?tab=readme-ov-file#configuration)

### swagger
Configuration for customizing Swagger.

Please refer to the [Swagger specification](https://swagger.io/specification/v2/).

### excludeStaticFile
@default `true`

Determine if Swagger should exclude static files.

### path
@default `/swagger`

Endpoint to expose Swagger

### exclude
Paths to exclude from Swagger documentation.

Value can be one of the following:
- **string**
- **RegExp**
- **Array<string | RegExp>**

## Pattern
Below you can find the common patterns to use the plugin.

## Change Swagger Endpoint
You can change the swagger endpoint by setting [path](#path) in the plugin config.

```typescript
import { Elysia } from 'elysia'
import { swagger } from '@elysiajs/swagger'

new Elysia()
    .use(swagger({
        path: '/v2/swagger'
    }))
    .listen(3000)
```

## Customize Swagger info
```typescript
import { Elysia } from 'elysia'
import { swagger } from '@elysiajs/swagger'

new Elysia()
    .use(swagger({
        documentation: {
            info: {
                title: 'Elysia Documentation',
                version: '1.0.0'
            }
        }
    }))
    .listen(3000)
```

## Using Tags
Elysia can separate the endpoints into groups by using the Swaggers tag system

Firstly define the available tags in the swagger config object

```typescript
app.use(
  swagger({
    documentation: {
      tags: [
        { name: 'App', description: 'General endpoints' },
        { name: 'Auth', description: 'Authentication endpoints' }
      ]
    }
  })
)
```

Then use the details property of the endpoint configuration section to assign that endpoint to the group 

```typescript
app.get('/', () => 'Hello Elysia', {
  detail: {
    tags: ['App']
  }
})

app.group('/auth', (app) =>
  app.post(
    '/sign-up',
    async ({ body }) =>
      db.user.create({
        data: body,
        select: {
          id: true,
          username: true
        }
      }),
    {
      detail: {
        tags: ['Auth']
      }
    }
  )
)
```

Which will produce a swagger page like the following
<img width="1446" alt="image" src="/assets/swagger-demo.webp">


---

# plugins/trpc.md


# tRPC Plugin
This plugin adds support for using [tRPC](https://trpc.io/)

Install with:
```bash
bun add @elysiajs/trpc @trpc/server @elysiajs/websocket 
```

Then use it:
```typescript
import { compile as c, trpc } from "@elysiajs/trpc";
import { initTRPC } from "@trpc/server";
import { Elysia, t as T } from "elysia";

const t = initTRPC.create();
const p = t.procedure;

const router = t.router({
  greet: p

    // 💡 Using Zod
    //.input(z.string())
    // 💡 Using Elysia's T
    .input(c(T.String()))
    .query(({ input }) => input),
});

export type Router = typeof router;

const app = new Elysia().use(trpc(router)).listen(3000);
```

## trpc
Accept the tRPC router and register to Elysia's handler.

type:
```
trpc(router: Router, option?: {
    endpoint?: string
}): this
```

`Router` is the TRPC Router instance.

### endpoint
The path to the exposed TRPC endpoint.


---

# validation/elysia-type.md


<script setup>
    import Card from '../../components/nearl/card.vue'
    import Deck from '../../components/nearl/card-deck.vue'
</script>

# Elysia Type

`Elysia.t` is based on TypeBox with pre-configuration for usage on the server while providing additional types commonly found on server-side validation.

You can find all of the source code of Elysia type in `elysia/type-system`.

The following are types provided by Elysia:

<Deck>
    <Card title="Numeric" href="#numeric">
        Accepts a numeric string or number and then transforms the value into a number
    </Card>
    <Card title="File" href="#file">
        A singular file. Often useful for <strong>file upload</strong> validation
    </Card>
    <Card title="Files" href="#files">
        Extends from <a href="#file">File</a>, but adds support for an array of files in a single field
    </Card>
    <Card title="Cookie" href="#cookie">
        Object-like representation of a Cookie Jar extended from Object type
    </Card>
    <Card title="Nullable" href="#nullable">
    Allow the value to be null but not undefined
    </Card>
    <Card title="Maybe Empty" href="#maybeempty">
        Accepts empty string or null value
    </Card>
</Deck>

## Numeric

Numeric accepts a numeric string or number and then transforms the value into a number.

```typescript
t.Numeric()
```

This is useful when an incoming value is a numeric string for example path parameter or query string.

Numeric accepts the same attribute as [Numeric Instance](https://json-schema.org/draft/2020-12/json-schema-validation#name-validation-keywords-for-num)

## File

A singular file. Often useful for **file upload** validation.

```typescript
t.File()
```

File extends attribute of base schema, with additional property as follows:

### type

A format of the file like image, video, audio.

If an array is provided, will attempt to validate if any of the format is valid.

```typescript
type?: MaybeArray<string>
```

### minSize

Minimum size of the file.

Accept number in byte or suffix of file unit:

```typescript
minSize?: number | `${number}${'k' | 'm'}`
```

### maxSize

Maximum size of the file.

Accept number in byte or suffix of file unit:

```typescript
maxSize?: number | `${number}${'k' | 'm'}`
```

#### File Unit Suffix:

The following are the specifications of the file unit:
m: MegaByte (1048576 byte)
k: KiloByte (1024 byte)

## Files

Extends from [File](#file), but adds support for an array of files in a single field.

```typescript
t.Files()
```

File extends attributes of base schema, array, and File.

## Cookie

Object-like representation of a Cookie Jar extended from Object type.

```typescript
t.Cookie({
    name: t.String()
})
```

Cookie extends attributes of [Object](https://json-schema.org/draft/2020-12/json-schema-validation#name-validation-keywords-for-obj) and [Cookie](https://github.com/jshttp/cookie#options-1) with additional properties follows:

### secrets

The secret key for signing cookies.

Accepts a string or an array of string

```typescript
secrets?: string | string[]
```

If an array is provided, [Key Rotation](https://crypto.stackexchange.com/questions/41796/whats-the-purpose-of-key-rotation) will be used, the newly signed value will use the first secret as the key.

## Nullable

Allow the value to be null but not undefined.

```typescript
t.Nullable(t.String())
```

## MaybeEmpty

Allow the value to be null and undefined.

```typescript
t.MaybeEmpty(t.String())
```

For additional information, you can find the full source code of the type system in [`elysia/type-system`](https://github.com/elysiajs/elysia/blob/main/src/type-system.ts).


---

# validation/error-provider.md


# Error Provider

There are 2 ways to provide a custom error message when the validation fails:

1. inline `error` property
2. Using [onError](/life-cycle/on-error) event

## Error Property

Elysia's offers an additional "**error**" property, allowing us to return a custom error message if the field is invalid.

```typescript twoslash
import { Elysia, t } from 'elysia'

new Elysia()
    .post('/', () => 'Hello World!', {
        body: t.Object(
            {
                x: t.Number()
            },
            {
                error: 'x must be a number'
            }
        )
    })
    .listen(3000)
```

The following is an example of usage of the error property on various types:

<table class="md-table">
<tr>
<td>TypeBox</td>
<td>Error</td>
</tr>

<tr>
<td>

```typescript
t.String({
    format: 'email',
    error: 'Invalid email :('
})
```

</td>
<td>

```
Invalid Email :(
```

</td>
</tr>

<tr>
<td>

```typescript
t.Array(
    t.String(),
    {
        error: 'All members must be a string'
    }
)
```

</td>
<td>

```
All members must be a string
```

</td>
</tr>

<tr>
<td>

```typescript
t.Object({
    x: t.Number()
}, {
    error: 'Invalid object UwU'
})
```

</td>
<td>

```
Invalid object UwU
```

</td>
</tr>
<tr>
<td>

```typescript
t.Object({
    x: t.Number({
        error({ errors, type, validation, value }) {
            return 'Expected x to be a number'
        }
    })
})
```

</td>
<td>

```
Expected x to be a number
```

</td>
</tr>

</table>

## Error message as function
Over a string, Elysia type's error can also accepts a function to programatically return custom error for each property.

The error function accepts same argument as same as `ValidationError`

```typescript twoslash
import { Elysia, t } from 'elysia'

new Elysia()
    .post('/', () => 'Hello World!', {
        body: t.Object({
            x: t.Number({
                error(error) {
                    return 'Expected x to be a number'
                }
            })
        })
    })
    .listen(3000)
```

::: tip
Hover over the `error` to see the type
:::

### Error is called per field
Please be cautious that the error function will only be called if the field is invalid.

Please consider the following table:

<table class="md-table">
<tr>
<td>Code</td>
<td>Body</td>
<td>Error</td>
</tr>

<tr>
<td>

```typescript twoslash
import { t } from 'elysia'
// ---cut---
t.Object({
    x: t.Number({
        error(error) {
            return 'Expected x to be a number'
        }
    })
})
```

</td>
<td>

```json
{
    x: "hello"
}
```

</td>
<td>
Expected x to be a number
</td>
</tr>

<tr>
<td>

```typescript twoslash
import { t } from 'elysia'
// ---cut---
t.Object({
    x: t.Number({
        error(error) {
            return 'Expected x to be a number'
        }
    })
})
```

</td>
<td>

```json
"hello"
```

</td>
<td>
(default error, `t.Number.error` is not called)
</td>
</tr>

<tr>
<td>

```typescript twoslash
import { t } from 'elysia'
// ---cut---
t.Object(
    {
        x: t.Number({
            error(error) {
                return 'Expected x to be a number'
            }
        })
    }, {
        error(error) {
            return 'Expected value to be an object'
        }
    }
)
```

</td>
<td>

```json
"hello"
```

</td>
<td>
Expected value to be an object
</td>
</tr>

</table>

## onError

We can customize the behavior of validation based on [onError](/life-cycle/on-error) event by narrowing down the error code call "**VALIDATION**".

```typescript
import { Elysia, t } from 'elysia'

new Elysia()
	.onError(({ code, error }) => {
		if (code === 'VALIDATION')
		    return error.message
	})
	.listen(3000)
```

Narrowed down error type, will be typed as `ValidationError` imported from 'elysia/error'.

**ValidationError** exposed a property name **validator** typed as [TypeCheck](https://github.com/sinclairzx81/typebox#typecheck), allowing us to interact with TypeBox functionality out of the box.

```typescript
import { Elysia, t } from 'elysia'

new Elysia()
    .onError(({ code, error }) => {
        if (code === 'VALIDATION')
            return error.validator.Errors(error.value).First().message
    })
    .listen(3000)
```

## Error list

**ValidationError** provides a method `ValidatorError.all`, allowing us to list all of the error causes.

```typescript twoslash
import { Elysia, t } from 'elysia'

new Elysia()
	.post('/', ({ body }) => body, {
		body: t.Object({
			name: t.String(),
			age: t.Number()
		}),
		error({ code, error }) {
			switch (code) {
				case 'VALIDATION':
                    console.log(error.all)

                    // Find a specific error name (path is OpenAPI Schema compliance)
					const name = error.all.find((x) => x.path === '/name')

                    // If has a validation error, then log it
                    if(name)
    					console.log(name)
			}
		}
	})
	.listen(3000)
```

For more information about TypeBox's validator, see [TypeCheck](https://github.com/sinclairzx81/typebox#typecheck).


---

# validation/overview.md


# Validation

The point of creating an API server is to take an input and process it.

We defined the shape of the data, allowing the client to send an input we agreed on to make everything behave normally.

However, a dynamic language like JavaScript doesn't validate the shape of an input by default.

An uninspected input may lead to unexpected behavior, missing data part, and in the worst case, a malicious intent to attack the server.

## Data Validation

Imagine data validation as having **someone** inspect every input for appropriate shape, so it won't break anything.

So we can have confidence in creating something without worrying about problem.

This **someone** is where Elysia takes part.

Elysia offers a complete schema builder to provide type safety for both runtime and compile time offering:

-   Infers to TypeScript Type automatically
-   Strict data validation
-   OpenAPI Schema to create Swagger documentation automatically

Elysia schema is exported as `Elysia.t` or short for **type**.

Elysia type is based on [Sinclair's TypeBox](https://github.com/sinclairzx81/typebox), a fast and extensive validation library.

## Why Elysia re-export TypeBox

Elysia extends the usage of TypeBox with a custom type for deep integration for Elysia's internal code generation.

Extending and customizing the default behavior of TypeBox to match for server-side validation.

For example, Elysia Type introduced some new types like:

-   **File**: A File or Blob of an HTTP Body
-   **Numeric**: Accept numeric string and convert to number
-   **ObjectString**: Stringified JSON, converted into Object
-   **Email Format**: Accept String that complies with email pattern

An integration like this should take care of the framework by default instead of relying on the user end to set up a custom type on every project, which is why Elysia decided to extend and re-export the TypeBox library instead.

## Chapter

This chapter is going to cover the basic usage of TypeBox and the new API introduced on Elysia type that is not provided in default TypeBox.

We recommended reading the essential chapter's [Schema](/essential/schema.html) first to understand the basic concept of Elysia type.

For a more in-depth topic, we recommend you to check out [TypeBox documentation](https://github.com/sinclairzx81/typebox), as dedicated documentation is more focused on each type behavior and additional settings it could provide.

Feel free to jump to the topic that interests you if you are already familiar with TypeBox.


---

# validation/primitive-type.md


# Primitive Type

TypeBox API is designed around and similar to TypeScript type.

There are a lot of familiar names and behaviors that intersect with TypeScript counter-parts like: **String**, **Number**, **Boolean**, and **Object** as well as more advanced features like **Intersect**, **KeyOf**, **Tuple** for versatility.

If you are familiar with TypeScript, creating a TypeBox schema has the same behavior as writing a TypeScript type except it provides an actual type validation in runtime.

To create your first schema, import `Elysia.t` from Elysia and start with the most basic type:

```typescript twoslash
import { Elysia, t } from 'elysia'

new Elysia()
  .post('/', ({ body }) => `Hello ${body}`, {
    body: t.String(),
  })
  .listen(3000);
```

This code tells Elysia to validate an incoming HTTP body, make sure that the body is String, and if it is String, then allow it to flow through the request pipeline and handler.

If the shape doesn't match, then it will throw an error, into [Error Life Cycle](/essential/life-cycle.html#events).

![Elysia Life Cycle](/assets/lifecycle.webp)

## Basic Type

TypeBox provides a basic primitive type with the same behavior as same as TypeScript type.

The following table lists the most common basic type:

<table class="md-table">
<tr>
<td>TypeBox</td>
<td>TypeScript</td>
</tr>

<tr>
<td>

```typescript
t.String()
```

</td>
<td>

```typescript
string
```

</td>
</tr>

<tr>
<td>

```typescript
t.Number()
```

</td>
<td>

```typescript
number
```

</td>
</tr>

<tr>
<td>

```typescript
t.Boolean()
```

</td>
<td>

```typescript
boolean
```

</td>
</tr>

<tr>
<td>

```typescript
t.Array(
    t.Number()
)
```

</td>
<td>

```typescript
number[]
```

</td>
</tr>

<tr>
<td>

```typescript
t.Object({
    x: t.Number()
})
```

</td>
<td>

```typescript
{
    x: number
}
```

</td>
</tr>

<tr>
<td>

```typescript
t.Null()
```

</td>
<td>

```typescript
null
```

</td>
</tr>

<tr>
<td>

```typescript
t.Literal(42)
```

</td>
<td>

```typescript
42
```

</td>
</tr>

</table>

Elysia extends all types from TypeBox allowing you to reference most of the API from TypeBox to use in Elysia.

See [TypeBox's Type](https://github.com/sinclairzx81/typebox#json-types) for additional types that are supported by TypeBox.

## Attribute

TypeBox can accept an argument for more comprehensive behavior based on JSON Schema 7 specification.

<table class="md-table">
<tr>
<td>TypeBox</td>
<td>TypeScript</td>
</tr>

<tr>
<td>

```typescript
t.String({
    format: 'email'
})
```

</td>
<td>

```typescript
saltyaom@elysiajs.com
```

</td>
</tr>

<tr>
<td>

```typescript
t.Number({
    minimum: 10,
    maximum: 100
})
```

</td>
<td>

```typescript
10
```

</td>
</tr>

<tr>
<td>

```typescript
t.Array(
    t.Number(),
    {
        /**
         * Minimum number of items
         */
        minItems: 1,
        /**
         * Maximum number of items
         */
        maxItems: 5
    }
)
```

</td>
<td>

```typescript
[1, 2, 3, 4, 5]
```

</td>
</tr>

<tr>
<td>

```typescript
t.Object(
    {
        x: t.Number()
    },
    {
        /**
         * @default false
         * Accept additional properties
         * that not specified in schema
         * but still match the type
         */
        additionalProperties: true
    }
)
```

</td>
<td>

```typescript
x: 100
y: 200
```

</td>
</tr>

</table>

See [JSON Schema 7 specification](https://json-schema.org/draft/2020-12/json-schema-validation) For more explanation for each attribute.



---

# validation/reference-model.md


# Reference Model
Sometimes you might find yourself declaring duplicated models, or re-using the same model multiple times.

With reference model, we can name our model and reuse them by referencing with name.

Let's start with a simple scenario.

Suppose we have a controller that handles sign-in with the same model.

```typescript twoslash
import { Elysia, t } from 'elysia'

const app = new Elysia()
    .post('/sign-in', ({ body }) => body, {
        body: t.Object({
            username: t.String(),
            password: t.String()
        }),
        response: t.Object({
            username: t.String(),
            password: t.String()
        })
    })
```

We can refactor the code by extracting the model as a variable, and reference them.
```typescript twoslash
import { Elysia, t } from 'elysia'

// Maybe in a different file eg. models.ts
const SignDTO = t.Object({
    username: t.String(),
    password: t.String()
})

const app = new Elysia()
    .post('/sign-in', ({ body }) => body, {
        body: SignDTO,
        response: SignDTO
    })
```

This method of separating the concerns is an effective approach but we might find ourselves reusing multiple models with different controllers as the app gets more complex.

We can resolve that by creating a "reference model"  allowing us to name the model and use auto-completion to reference it directly in `schema` by registering the models with `model`.

```typescript twoslash
import { Elysia, t } from 'elysia'

const app = new Elysia()
    .model({
        sign: t.Object({
            username: t.String(),
            password: t.String()
        })
    })
    .post('/sign-in', ({ body }) => body, {
        // with auto-completion for existing model name
        body: 'sign',
        response: 'sign'
    })
```

When we want to access the model's group, we can separate a `model` into a plugin which when registered will provide a set of models instead of multiple import.

```typescript twoslash
// auth.model.ts
import { Elysia, t } from 'elysia'

export const authModel = new Elysia()
    .model({
        sign: t.Object({
            username: t.String(),
            password: t.String()
        })
    })
```

Then in an instance file:
```typescript twoslash
// @filename: auth.model.ts
import { Elysia, t } from 'elysia'

export const authModel = new Elysia()
    .model({
        sign: t.Object({
            username: t.String(),
            password: t.String()
        })
    })

// @filename: index.ts
// ---cut---
// index.ts
import { Elysia } from 'elysia'
import { authModel } from './auth.model'

const app = new Elysia()
    .use(authModel)
    .post('/sign-in', ({ body }) => body, {
        // with auto-completion for existing model name
        body: 'sign',
        response: 'sign'
    })
```

This not only allows us to separate the concerns but also allows us to reuse the model in multiple places while reporting the model into Swagger documentation.

## Multiple Models
`model` accepts an object with the key as a model name and value as the model definition, multiple models are supported by default.

```typescript twoslash
// auth.model.ts
import { Elysia, t } from 'elysia'

export const authModel = new Elysia()
    .model({
        number: t.Number(),
        sign: t.Object({
            username: t.String(),
            password: t.String()
        })
    })
```

## Naming Convention
Duplicated model names will cause Elysia to throw an error. To prevent declaring duplicate model names, we can use the following naming convention.

Let's say that we have all models stored at `models/<name>.ts`, and declare the prefix of the model as a namespace.

```typescript twoslash
import { Elysia, t } from 'elysia'

// admin.model.ts
export const adminModels = new Elysia()
    .model({
        'admin.auth': t.Object({
            username: t.String(),
            password: t.String()
        })
    })

// user.model.ts
export const userModels = new Elysia()
    .model({
        'user.auth': t.Object({
            username: t.String(),
            password: t.String()
        })
    })
```

This can prevent naming duplication at some level, but in the end, it's best to let the naming convention decision up to your team's agreement is the best option.

Elysia provides an opinionated option for you to decide to prevent decision fatigue.


---

# validation/schema-type.md


<script setup>
    import Card from '../../components/nearl/card.vue'
    import Deck from '../../components/nearl/card-deck.vue'

    import Playground from '../../components/nearl/playground.vue'
    import { Elysia, t, ValidationError } from 'elysia'

    const demo1 = new Elysia()
        .get('/id/1', '1')
        .get('/id/a', () => {
            throw new ValidationError(
                'params',
                t.Object({
                    id: t.Numeric()
                }),
                {
                    id: 'a'
                }
            )
        })
</script>

# Schema Type

Elysia supports declarative schema with the following types:

<Deck>
    <Card title="Body" href="#body">
        Validate an incoming HTTP Message
    </Card>
    <Card title="Query" href="#query">
        Query string or URL parameter
    </Card>
    <Card title="Params" href="#params">
        Path parameters
    </Card>
    <Card title="Header" href="#header">
        Header of the request
    </Card>
    <Card title="Cookie" href="#cookie">
        Cookie of the request
    </Card>
    <Card title="Response" href="#response">
        Response of the request
    </Card>
</Deck>



---

