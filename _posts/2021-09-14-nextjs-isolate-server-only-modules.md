---
layout: post
title: "How to Isolate Next.js Server-Only Modules"
date: 2021-09-14 08:00:00 +1000
tags: [webpack]
---

<center>
<img src="/assets/2021-09-14-nextjs-isolate-server-only-modules/spiral-3473806_640.jpg" />
<br />
<br />
(<a href="https://pixabay.com/photos/spiral-kringel-circle-nature-plant-3473806/">source</a>)
</center>
<br />

# Introduction

- When performing a server-side render of a next.js React page component, some server-only code may be required (eg. fetch data from a in-memory server cache, server logging)
- However, some server-only code:
  1. _Should not_ be executed on the client
     - eg. bundling an in-memory XHR cache that was only meant to be used on the server-side
  2. _Cannot_ be executed on the client
     - eg. emitting server logs via `hot-shots`, which requires a `dgram` dependency that does not exist on the client bundle.
- Thus, isolating next.js server-only modules helps to ensure that we avoid bundling server-only code into our client bundles.
- There are two general **2 approaches** to do this:
  1. Isolate via calling server-only code within `getServerSideProps()`
  2. Isolate via module import alias

# Scenario

Given the following setup:

1. A page is hosted on a next.js instance
2. Page needs to make 1x XHR call as part of its server-side rendering step
3. XHR call may be cached by using a `memcached` instance

Goal: We want to ensure that all `memcached`-related code is **not included** in the next.js client bundle.

_Note_: Refer to my example repo [nextjs-ssr-localcache-example](https://github.com/nossbigg/nextjs-ssr-localcache-example) for code and documentation references.

# Approach 1: Isolate via calling server-only code within `getServerSideProps()`

This is the **recommended way** to isolate server-only modules.

- Given that the server-only code is called only within `getServerSideProps()`,
- When the client bundle build step is performed,
- Then next.js will perform [tree-shaking for `getServerSideProps()`](https://github.com/vercel/next.js/discussions/12607#discussioncomment-12358), thus ensuring that server-only code will neither be bundled nor invoked on the client side.

Note: This also works for `getStaticProps()` as well.

Steps:

1. Invoke server-only code in `getServerSideProps()`

   ```typescript
   // UserCachedSspPage.tsx
   const User = (props) => { ... };

   export async function getServerSideProps() {
      const xhrCacheHelper = makeXhrCacheHelper();
      if (xhrCacheHelper) {
         const { getKey } = xhrCacheHelper;
         const dataString = (await getKey("userData")) as string;
         ...
      }
      ...
   }
   ```

   [(Code)](https://github.com/nossbigg/nextjs-ssr-localcache-example/blob/8ca786f4f0f01fd571c3a85f2ef7d4d807f8de60/pages/user-cached-ssp/UserCachedSspPage.tsx)

# Approach 2: Isolate via module import alias

However, if you're in either of these scenarios:

1. You are on a version of next.js `<9.3`, where `getServerSideProps()` is not available
   - eg. executing server-only code within `getInitialProps()`
2. You want to execute some server-side code within React components that are invoked during a server-side page render
   - eg. Collecting performance metrics and emitting logs on server-side only

...then you would need to **manually isolate server-only code**.

Steps:

1. Organize server-only code in a separate folder (eg. `src/SERVER_ONY_MODULES`)

   ```typescript
   // src/SERVER_ONLY_MODULES/xhrCacheHelper/xhrCacheHelper.ts
   import Memcached from "memcached";

   export const makeXhrCacheHelper = () => {
     const memcached = new Memcached("...");

     const closeConnection = () => { ... };
     const setKey = (key, value) => { ... };
     const getKey =  (key) => { ... };
     return { setKey, getKey, closeConnection };
   };
   ```

   [(Code)](https://github.com/nossbigg/nextjs-ssr-localcache-example/blob/8ca786f4f0f01fd571c3a85f2ef7d4d807f8de60/src/SERVER_ONLY_MODULES/xhrCacheHelper/xhrCacheHelper.ts)

   `xhrCacheHelper()` defines an object with functions to get/set values with the `memcached` instance.

2. Access server-only code using import aliases (eg. `@SERVER_ONLY_MODULES`)

   ```typescript
   // UserCachedIpPage.tsx
   const getMakeXhrCacheHelper = async () => {
     const module = await import("@SERVER_ONLY_MODULES");
     if (!module) {
       return noop;
     }
     return module.makeXhrCacheHelper;
   };

   const User = (props) => { ... };

   User.getInitialProps = async () => {
      const makeXhrCacheHelper = await getMakeXhrCacheHelper();
      ...
   }
   ```

   [(Code)](https://github.com/nossbigg/nextjs-ssr-localcache-example/blob/8ca786f4f0f01fd571c3a85f2ef7d4d807f8de60/pages/user-cached-ip/UserCachedIpPage.tsx)

   `getMakeXhrCacheHelper()` is a utility that defensively checks for the existence of the module; this is necessary as the `import()` statement would not resolve to the actual module (ie. return `undefined`) when executed on the client browser.

3. Isolate server-only module imports for client bundle builds (via webpack config override via next.config.js)

   ```typescript
   // next.config.js
   const webpackConfigFn = (config, { isServer }) => {
     // for webpack to resolve import alias correctly
     config.resolve.alias["@SERVER_ONLY_MODULES"] = "src/SERVER_ONLY_MODULES";

     if (!isServer) {
       // resolve @SERVER_ONLY_MODULES as empty module on client
       config.resolve.alias["@SERVER_ONLY_MODULES"] = false;
     }

     return config;
   };
   ```

   [(Code)](https://github.com/nossbigg/nextjs-ssr-localcache-example/blob/8ca786f4f0f01fd571c3a85f2ef7d4d807f8de60/next.config.js)

   We provide configuration to support the `@SERVER_ONLY_MODULES` import alias, as well as to resolve `@SERVER_ONLY_MODULES` to an empty module on the client browser.

# Appendix

This article is inspired by _Arunoda Susiripala_'s article, [SSR and Server Only Modules](https://arunoda.me/blog/ssr-and-server-only-modules) (referenced by official next.js documentation).

However, I found that I was still getting bundle build errors and client bundle runtime errors - This article fleshes out an alternative approach to isolating server-only modules.

Likewise as with Arunoda's article, I hope this article improves the developer community's awareness of this issue 🙃
