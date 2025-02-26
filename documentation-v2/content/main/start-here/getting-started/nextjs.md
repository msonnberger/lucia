---
order: 0
title: "Getting started in Next.js"
description: "Learn how to set up Lucia in your Next.js project"
---

Install Lucia using your package manager of your choice.

```
npm i lucia
pnpm add lucia
yarn add lucia
```

## Next.js support

Lucia supports both the old pages and new app directory in Next.js, with support for Node.js and the edge runtime as well. However, it will only work as expected when the pages directory is used with Node.js. Currently, Next.js does not provide a way to set cookies inside pages when using the app directory or the pages directory with the edge runtime. As such, Lucia cannot store renewed session ids under certain conditions.

## Initialize Lucia

Import [`lucia()`](/reference/lucia/main#lucia) from `lucia` and initialize it in its own module (file). Export `auth` and its type as `Auth`. Make sure to pass the `nextjs()` middleware. We also need to provide an `adapter` but since it'll be specific to the database you're using, we'll cover that in the next section.

```ts
// auth/lucia.ts
import { lucia } from "lucia";
import { nextjs } from "lucia/middleware";

// expect error
export const auth = lucia({
	env: "DEV", // "PROD" if deployed to HTTPS
	middleware: nextjs()
});

export type Auth = typeof auth;
```

## Setup your database

Lucia uses adapters to connect to your database. We provide official adapters for a wide range of database options, but you can always [create your own](/extending-lucia/database-adapters-api). The schema and usage are described in each adapter's documentation. The example below is for the Prisma adapter.

```ts
// auth/lucia.ts
import { lucia } from "lucia";
import { nextjs } from "lucia/middleware";
import { prisma } from "@lucia-auth/adapter-prisma";
import { PrismaClient } from "@prisma/client";

const client = new PrismaClient();

const auth = lucia({
	env: "DEV", // "PROD" if deployed to HTTPS
	middleware: nextjs(),
	adapter: prisma(client)
});
```

### Adapters for database drivers and ORMs

- [`better-sqlite3`](/database-adapters/better-sqlite3): SQLite
- [Mongoose](/database-adapters/mongoose): MongoDB
- [`mysql2`](/database-adapters/mysql2): MySQL
- [`pg`](/database-adapters/pg): PostgreSQL
- [Prisma](/database-adapters/prisma): MongoDB, MySQL, PostgreSQL, SQLite
- [Redis](/database-adapters/redis): Redis

### Provider specific adapters

- [Cloudflare D1](/database-adapters/cloudflare-d1)
- [PlanetScale serverless](/database-adapters/planetscale-serverless)

## Set up types

Create a TS declaration file (`lucia.d.ts`) and declare a `Lucia` namespace. The import path for `Auth` is where you initialized `lucia()`.

```ts
// lucia.d.ts
/// <reference types="lucia" />
declare namespace Lucia {
	type Auth = import("./auth/lucia").Auth;
	type DatabaseUserAttributes = {};
	type DatabaseSessionAttributes = {};
}
```

## Polyfill

If you're using Node.js version 18 or below, you need to polyfill the [Web Crypto API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API). This is not required if you're using runtimes other than Node.js (Deno, Bun, Cloudflare Workers, etc) or using Node.js v20 and above.

```ts
import { lucia } from "lucia";
import "lucia/polyfill/node";

export const auth = lucia({
	// ...
});
```

Optionally, instead of doing a side-effect import, add the `--experimental-global-webcrypto` flag when running `next`.

```json
// package.json
{
	// ...
	"scripts": {
		"dev": "NODE_OPTIONS=--experimental-global-webcrypto next dev",
		"start": "NODE_OPTIONS=--experimental-global-webcrypto next start"
		// ...
	}
	// ...
}
```
