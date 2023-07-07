---
type: example
summary: Query a D1 database from a SvelteKit application.
tags:
  - SvelteKit
  - Svelte
  - D1
pcx_content_type: configuration
title: Query D1 from SvelteKit
weight: 3
layout: example
---

[SvelteKit](https://kit.svelte.dev/) is a full-stack framework that combines the Svelte front-end framework with Vite for server-side capabilities and rendering. You can query D1 from SvelteKit by configuring a [server endpoint](https://kit.svelte.dev/docs/routing#server) with a binding to your D1 database(s).

To set up a new SvelteKit site on Cloudflare Pages that can query D1:

1. Refer to [the Svelte guide](/pages/framework-guides/deploy-a-svelte-site/) and Svelte's [Cloudflare adapter](https://kit.svelte.dev/docs/adapter-cloudflare).
2. Install the Cloudflare adapter within your SvelteKit project: `npm i -D @sveltejs/adapter-cloudflare`.
3. Bind a D1 database [to your Pages Function](/pages/platform/functions/bindings/#d1-databases).
4. Pass the `--d1=BINDING_NAME` flag when developing locally. `BINDING_NAME` should match what call in your code: for example, `--d1=DB`.

The following example shows you how to create a server endpoint configured to query D1.

* Bindings are available on the `platform` parameter passed to each endpoint, via `platform.env.BINDING_NAME`.
* With SvelteKit's [file-based routing](https://kit.svelte.dev/docs/routing), the server endpoint defined in `src/routes/api/users/+server.ts` is available at `/api/users` within your SvelteKit app.

The example also shows you how to configure both your app-wide types within `src/app.d.ts` to recognize your `D1Database` binding, import the `@sveltejs/adapter-cloudflare` adapter into `svelte.config.js`, and configure it to apply to all of your routes.

{{<tabs labels="ts">}}
{{<tab label="ts" default="true">}}
```ts
---
filename: src/routes/api/users/+server.ts
---
import type { RequestHandler } from "@sveltejs/kit";

/** @type {import('@sveltejs/kit').RequestHandler} */
export async function GET({ request, platform }) {
  let result = await platform.env.DB.prepare(
    "SELECT * FROM users LIMIT 5"
  ).run();
  return new Response(JSON.stringify(result));
}
```
```ts
---
filename: src/app.d.ts
---
// See https://kit.svelte.dev/docs/types#app
// for information about these interfaces
declare global {
  namespace App {
    // interface Error {}
    // interface Locals {}
    // interface PageData {}
    interface Platform {
      env: {
        DB: D1Database;
      };
      context: {
        waitUntil(promise: Promise<any>): void;
      };
      caches: CacheStorage & { default: Cache };
    }
  }
}

export {};
```
```js
---
filename: svelte.config.js
---
import adapter from '@sveltejs/adapter-cloudflare';

export default {
    kit: {
        adapter: adapter({
            // See below for an explanation of these options
            routes: {
                include: ['/*'],
                exclude: ['<all>']
            }
        })
    }
};

```
{{</tab>}}
{{</tabs>}}