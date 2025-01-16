<!--
Use the below Q/A template to help.

---

###
<details>
<summary>Answer</summary>

</details>

---

### When I deploy to Payload Cloud, my build gets stuck on...
<details>
<summary>Answer</summary>

</details>
-->

# FAQ's and Troubleshooting V3

Hey Payload,

It would be nice if the docs could include a `FAQ's` section somewhere for the most common issues users encounter while building their shiny new Payload V3 app. 

In the meantime, I've compiled a list of the most common user issues I've encountered from over 130 pages of Discord discussions and threads for all to browse. I hope it helps users, contributors, and devs alike in getting to answers quickly. The intention here is for this to be a resource one can easily share a link to for a user in need.

I've intentionally omitted questions about [Custom Components](https://payloadcms.com/docs/admin/components) from this list as those are typically so specific that it's hard to generalize to everyone, and [the docs](https://payloadcms.com/docs/getting-started/what-is-payload) do a pretty good job here anyway.

If there are common questions you've noticed are missing, feel free to add them below or send me a DM on Discord and I'll add it here. The following list of Q & A's will appear in no particular order.

---

### How come my images work locally but not when deployed to Vercel or other serverless providers?
<details>
<summary>Answer</summary>

The most likely cause of this is because you're storing those images in the local filesystem without using a storage-adapter. Serverless systems lack a permanent file system and are typically ephemeral, so Payload ends up looking for images that no longer exist which results in not found errors. The solution is to use a [storage-adapter](https://payloadcms.com/docs/upload/storage-adapters) when deploying to a serverless provider.
</details>

---

### How can I group related collections together in the admin ui?
<details>
<summary>Answer</summary>

You can group related collections together in the admin ui dashboard and navbar by specifying an `admin.group` property in your collection or global configuration. Learn more [here](https://payloadcms.com/docs/admin/collections#admin-options).
</details>

---

### Why is the Local API saying it can't find one of my collections / behaving unexpectedly in hooks?
<details>
<summary>Answer</summary>

The most commong issue for this is missing a `req` before referencing `payload`. Instead of `payload.find(...)`, it should be `req.payload.find`. This is only true if you are not destructuring `payload` out of the `req` beforehand.

Bad:
```ts
const beforeChangeHook: CollectionBeforeChangeHook = async ({
  data,
  req
}) => {
  const findResult = await payload.find({...})
  // ...
  return data
}
```

Good:
```ts
const beforeChangeHook: CollectionBeforeChangeHook = async ({
  data,
  req
}) => {
  const findResult = await req.payload.find({...}) // Notice the req
  // ...
  return data
}
```

If you're still experiencing issues with the Payload Local API, then [see here](https://github.com/akhrarovsaid/payload-v3-faqs?tab=readme-ov-file#im-using-the-local-api-with-db-postgres-to-perform-an-update-to-a-document-why-does-the-operation-hang-or-never-execute) for another potential fix.
</details>

---

### Why am I getting an error about my Payload dependencies having mismatched versions?
<details>
<summary>Answer</summary>

This error indicates that some or all of your Payload packages aren't on the same version. To fix this, ensure all of your packages match your Payload version in `package.json`, delete `node_modules` along with your `package-lock.json` (or `pnpm-lock.yaml`, or `yarn.lock`), and reinstall your dependencies.
</details>

---

### Why am I getting `"[ Server ] Error: Dependency react is on version 18.3.1, but >=19.0.0-rc-65a56d0e-20241020 or greater is required. Please update this dependency."`
<details>
<summary>Answer</summary>

This error indicates that you have an incompatible `react` and/or `react-dom` version installed. Payload V3 requires React and React-dom to be on version 19, as well as Nextjs on version 15 or greater. See recommended versions to pin to your `package.json` [here](https://github.com/payloadcms/payload/blob/main/templates/website/package.json#L47-L48).
</details>

---

### Why am I getting `"In HTML, <html> cannot be a child of <body>. This will cause a hydration error."`
<details>
<summary>Answer</summary>

The most common cause of this issue is because you have a root `layout.tsx` that wraps around your `(payload)` folder. There already exists a `layout.tsx` inside the (payload) folder. This causes Payload to inherit a layout where there are `html` and `body` tags nested within eachother which is an invalid `html` structure. To fix this, ensure you don't have a root layout wrapping the `(payload)` folder. Have a look at how the [website template](https://github.com/payloadcms/payload/tree/main/templates/website/src/app) handles this.
</details>

---

### How can I use Payload v3 with `<my-favourite-framework>`
<details>
<summary>Answer</summary>

If you're using `NextJS` v15 then Payload can install directly into your application. For all other frameworks, you can run Payload in headless mode, you can have two projects - one for your application, and one for Payload to run in.
</details>

---

### Can I use the Payload Local API in my frontend?
<details>
<summary>Answer</summary>

It depends. If you're running Payload inside your NextJS application, then you can use the Local API anywhere as long as it remains on the server. If you're using Payload outside of NextJS, then please [refer to the docs here](https://payloadcms.com/docs/local-api/outside-nextjs).
</details>

---

### I added Versions & Drafts to my collection config and now all of my documents are missing!
<details>
<summary>Answer</summary>

This happens because when you enable versions, they are stored differently in the database from ordinary collections. Your documents need to be populated into the `_your-collection_versions` table to work properly. See [a helpful response from Dan about this here](https://github.com/payloadcms/payload/discussions/5353#discussioncomment-8828018) and [this one too](https://payloadcms.com/community-help/discord/i-dont-see-my-previous-collections-after-adding-in-versions).
</details>

---

### Why is Payload loading my changes so slowly in `dev` mode?
<details>
<summary>Answer</summary>

There are many possible reasons for this. Payload, itself, is quite heavily optimized and receives updates to improve performance frequently. The most common factor for this is using a remote database while developing. This can add unnecessary overhead as schema changes must be pushed and pulled to somewhere remote. Consider developing with a database located on your local machine.
</details>

---

### Why am I getting "Javascript heap out of memory" when trying to build my Payload app?
<details>
<summary>Answer</summary>

There are many possible reasons for this. This could be the result of poorly optimized application code, uncontained recursive logic, or that the build machine simply does not have enough RAM to build the application. To fix this, ensure your logic and code work as expected. If this is caused by insufficient RAM, consider adding a swapfile or increasing the RAM size of the instance.
</details>

---

### When I make a change to one of my Collection docs/Globals, those changes are not reflected in my frontend. How come?
<details>
<summary>Answer</summary>

In order for the frontend to update when changes to documents occur, there needs to be some kind of logic present which revalidates the necessary resource. This is commonly handled via an `afterChange` hook. See how the template handles this [for globals](https://github.com/payloadcms/payload/tree/main/templates/website/src/Header), and [for pages](https://github.com/payloadcms/payload/tree/main/templates/website/src/collections/Pages). [Learn more](https://payloadcms.com/docs/hooks/collections#afterchange) about the `afterChange` hook.
</details>

---

### How do I change the root `/admin` route?
<details>
<summary>Answer</summary>

In order to change the root `/admin` route, you need to add a `routes.admin` property to your Payload config _and_ change the `admin` directory to match as this is where the admin ui lives. See the docs on this [here](https://payloadcms.com/docs/admin/overview#customizing-routes).
</details>

---

### I've updated my Payload package from beta.* to v3 stable. Now I'm getting `<some-error>` errors, how can I resolve these?
<details>
<summary>Answer</summary>

The most common cause of errors when migrating from v3 beta to v3 stable is by missing instructions for breaking changes. To fix this, you should carefully go through the [release notes](https://github.com/payloadcms/payload/releases) of every release between your beta version and the version you wish to migrate to. You can use [this site](https://payload-releases-filter.vercel.app/) to help check for breaking changes quickly.

Credit to @linobino1 for the excellent breaking changes utility site!
</details>

---

### How do I migrate from Payload v2.x to v3.x?
<details>
<summary>Answer</summary>

See the [comprehensive migration guide here](https://github.com/payloadcms/payload/blob/main/docs/migration-guide/overview.mdx).
</details>

---

### How do I use `<some-component-name>` from the `@payloadcms/ui` package?
<details>
<summary>Answer</summary>

While we wait for comprehensive documentation and guides for the `ui` package, the best resource to learn about components here remains to be the `ui` package itself, as well as the monorepo `next` package where many of these components are consumed and used with best practices.
</details>

---

### I've deployed my Payload app to a Serverless provider, why is the initial load of the site so slow now?
<details>
<summary>Answer</summary>

Serverless platforms suffer from cold-starts. This happens when there are no "warm" instances of your function are running, so they must be spun up and can encounter overhead due to having to start from scratch. This is not a Payload issue, but a trade-off when deploying to serverless and something to consider. Read [this article](https://vercel.com/guides/how-can-i-improve-serverless-function-lambda-cold-start-performance-on-vercel) from Vercel to learn more. Some common patterns here are to use your providers native method of keeping your functions warm, running a cron-job to periodically send requests to your functions to keep them warm, or to use a different deployment target altogether (such as a VPS).
</details>

---

### How can I get Tailwind to work with the Payload Admin?
<details>
<summary>Answer</summary>

To get Tailwind and Shad/cn working with the Payload Admin ui, you can simply install Tailwind and include the layers in the `custom.css` file located in the `(payload)` folder. However, make sure not to include the `@layer base;` directive as the preflight styles from this will interfere with the `payload-default` styles that are used to style the admin app.
</details>

---

### Why are my relationships and uploads typed with `<Media or Relationship> | <string or number>`?
<details>
<summary>Answer</summary>

This happens because any given collection config may specify deeply nested relationships. Payload has no way to know, in advance, at what depth to fetch those relationships. As a result, to optimize requests, some docs may return relationships that contain only id's. A common fix for this is to use the `depth` property of the API you are using. See [here for Local API](https://payloadcms.com/docs/queries/depth#local-api), here [for REST API](https://payloadcms.com/docs/queries/depth#rest-api). Learn more about depth [here](https://payloadcms.com/docs/queries/depth).
</details>

---

### How can I prevent some users from accessing the Payload Admin?
<details>
<summary>Answer</summary>

You may use the `admin` access control function of your auth-enabled collection. See [here for more details](https://payloadcms.com/docs/access-control/collections#admin).
</details>

---

### I've deployed my Payload app, now I'm getting "You are not allowed to perform this action" errors. How come?
<details>
<summary>Answer</summary>

This can happen for numerous reasons. The two most common ones are:
- You have not configured CORS for your domain correctly. [Learn more](https://payloadcms.com/docs/configuration/overview#cross-origin-resource-sharing-cors).
- Your application is running on a different PORT than expected, commonly when other applications are running on the same port (3000 by default).
</details>

---

### Why am I getting `Error: missing secret key. A secret key is needed to secure Payload.`
<details>
<summary>Answer</summary>

A secret key is required to secure Payload properly. In order to add one, you must add a `PAYLOAD_SECRET` environment variable to your `.env` file. See more about this [in the docs](https://payloadcms.com/docs/production/deployment#the-secret-key).
</details>

---

### I have an idea for a feature request, how should I proceed?
<details>
<summary>Answer</summary>

The best way to make sure your feature request gets looked at is to create a Github discussion with the "Feature Request" flag on it. These get checked regularly, and allows your request to get feedback from devs and the broader community. See [discussions here](https://github.com/payloadcms/payload/discussions).
</details>

---

### I'm using `db-postgres` and now I'm getting an error related to the PostgreSQL 63 character table name limit, what should I do?
<details>
<summary>Answer</summary>

Unfortunately this is an optimization limitation enforced by PostgreSQL. Many fields can commonly be found deeply nested and, as a result, will sometimes have long table names in the database. These fields expose the `dbName` property which allow you to specify your own table name for them to use. A full list of fields that expose the `dbName` property are:

- [Arrays](https://payloadcms.com/docs/fields/array)
- [Blocks](https://payloadcms.com/docs/fields/blocks#block-configs)
- [Select](https://payloadcms.com/docs/fields/select)
</details>

---

### I'm using the Local API with `db-postgres` to perform an `update` to a document, why does the operation hang or never execute?
<details>
<summary>Answer</summary>

This happens because the Local API may be performing a transaction and, as such, requries a `req` PayloadRequest to be passed in order to execute correctly. [Learn more about transactions](https://payloadcms.com/docs/local-api/overview#transactions). Passing a `req` property is encouraged even if you're not using `db-postgres`.
</details>

---

### I'm getting a "Module not found: Can't resolve 'fs'" error, how do I resolve this?
<details>
<summary>Answer</summary>

While this could be the result of user-code error, two common reasons this happens are:
- You are using `NextJS` and have an `'edge'` runtime variable export somewhere in your code.
- You are using a server-only component inside of a client component.

If the above are not true, then you can try deleting `node_modules` and reinstalling your dependencies.
</details>

---

### Help, I've connected to my prod | staging | test db while running a dev environment. Now I'm getting `"It looks like you've run Payload in dev mode, meaning you've dynamically pushed changes to your database.If you'd like to run migrations, data loss will occur. Would you like to proceed?"` How can I revert this?
<details>
<summary>Answer</summary>

This warning is to prevent people from running `dev` mode (which force pushes schema changes) and migrations against the same database. This can cause issues and is inadvisable. It is also highly inadvisable to run `dev` against your database in production.

To resolve this issue simply connect to your db, using a CLI utility or a GUI such as pgAdmin, and delete rows where `batch` equals -1 in the `payload_migrations` table.
</details>

---

### How come my uploaded media work locally but not when deployed?
<details>
<summary>Answer</summary>

There could be numerous reasons why uploaded media don't work when deployed. Double check that you have configured your application firewall correctly, and that the rules of the provider you've chosen to serve media does not block requests from your application. You can also check that you're allowing the hostname of your image provider in `next.config.js` via the `remotePatterns` property ([see here](https://nextjs.org/docs/app/api-reference/components/image#remotepatterns)).
</details>

---

### I'd like to configure Cloudflare R2 as my storage system - what do I need to do to make it work?
<details>
<summary>Answer</summary>

Since R2 is compatible with the S3 API, all you need is the S3 storage adapter. See [docs for S3 adapter here](https://payloadcms.com/docs/upload/storage-adapters#s3-storage).
</details>
