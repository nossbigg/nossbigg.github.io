---
layout: post
title: "How to: React SSR Next.js + Google reCAPTCHA v3"
date: 2020-10-17 10:00:00 +0800
tags: [react]
---

<center>
<img src="/assets/2020-10-17-howto-react-ssr-nextjs-recaptcha/among-us-sus.jpg" />
<br />
<br />
<i>"man, some of these requests be looking sus... 🤔"</i>
</center>
<br />

# Use Case

- You want to protect some **sensitive information** (eg. `item price`) behind Google reCAPTCHA to deter other companies from scraping your site
- You want to use `Next.js` SSR to have **speedy renders** and **improved SEO performance** for your React site

# Solution

Example code: [react-csr-ssr-recaptcha-example](https://github.com/nossbigg/react-csr-ssr-recaptcha-example)

#### A Prelude: React CSR + Google reCAPTCHA

Here's the standard flow of integrating reCATPCHA with a _vanilla_ React Client-Side Rendering (CSR) instance:

<center><img src="/assets/2020-10-17-howto-react-ssr-nextjs-recaptcha/react-csr-recaptcha.svg" /></center>

1. Load Google reCAPTCHA script via `public/index.html`
2. Upon page load, acquire reCAPTCHA token
3. Upon acquiring reCAPTCHA token, make request to `/getItem` endpoint to retrieve sensitive data
4. Backend validates reCAPTCHA token
5. Upon successful request, update page with data

#### React SSR Goals

Here are the goals we would like to achieve with React Server-Side Rendering:

1. Populate initial page render with non-sensitive data

   - For SEO to pick up non-sensitive data
   - For faster overall page loads for less powerful devices

2. Use reCAPTCHA to ensure origin of request is not a bot

   - To deter web scrapers from scraping valuable data (eg. `item prices, flight ticket prices, booking slots`, etc...)
   - Note: reCAPTCHA is not a foolproof method, but it's a good _first-cut deterrent_.

3. Upon page load, make a request that retrieves sensitive data and updates page with sensitive data

#### React SSR + Google reCAPTCHA

Here's the _general gist_ of how it's done:

<center><img src="/assets/2020-10-17-howto-react-ssr-nextjs-recaptcha/react-ssr-recaptcha.svg" /></center>

**Next.js SSR Instance:**

1. Load Google reCAPTCHA script into Next.js `_document.tsx`
2. Render initial page with non-sensitive item data (because that's the _whole point_ of SSR rendering 😅)
3. Upon page load, Do `useRecaptchaHook()` to acquire reCAPTCHA token
4. Do `useGetProtectedInfoHook()` to make endpoint `/getItem` request with reCAPTCHA token in request header
5. Update item data on page on request success

_Code:_

<details markdown='1'>
<summary>pages/_document.tsx</summary>

```javascript
import React from "react";
import Document, { Html, Head, Main, NextScript } from "next/document";
import { RECAPTCHA_SITE_KEY } from "./recaptchaEnvVars";

class MyDocument extends Document {
  render() {
    const recaptchaScriptSource = `https://www.google.com/recaptcha/api.js?render=${RECAPTCHA_SITE_KEY}`;

    return (
      <Html>
        <Head>
          <script src={recaptchaScriptSource}></script>
        </Head>
        <body>
          <Main />
          <NextScript />
        </body>
      </Html>
    );
  }
}

export default MyDocument;
```

</details>

<details markdown='1'>
<summary>pages/index.tsx</summary>

```javascript
import React from "react";
import { GetServerSideProps } from "next";
import { useRecaptchaHook } from "../src/common/useRecaptchaHook";
import { useGetProtectedInfoHook } from "../src/common/useGetProtectedInfoHook";
import { RECAPTCHA_SITE_KEY } from "./recaptchaEnvVars";

type IndexPageType = IndexPageServerSideProps;
type IndexPageServerSideProps = { unprotectedInfo: Object };

const IndexPage: React.FC<IndexPageType> = (props) => {
  const { unprotectedInfo } = props;

  const token = useRecaptchaHook(RECAPTCHA_SITE_KEY);
  const protectedInfo = useGetProtectedInfoHook(token);

  return (
    <div>
      Hello World!
      <br />
      Unprotected Info: {JSON.stringify(unprotectedInfo)}
      <br />
      Protected Info: {JSON.stringify(protectedInfo)}
    </div>
  );
};

export const getServerSideProps: GetServerSideProps<IndexPageServerSideProps> = async () => {
  let respJson: Object = {};
  try {
    const resp = await fetch("http://localhost:3005/getItem");
    respJson = await resp.json();
  } catch (e) {}

  return { props: { unprotectedInfo: respJson } };
};

export default IndexPage;
```

</details>

<details markdown='1'>
<summary>src/common/useRecaptchaHook.tsx</summary>

```javascript
import { useState, useEffect } from "react";

export const useRecaptchaHook = (recaptchaSiteKey: string) => {
  const [recaptchaToken, setRecaptchaToken] = useState("");

  useEffect(() => {
    if (recaptchaToken) {
      return;
    }

    const { grecaptcha } = window as any;
    grecaptcha.ready(async () => {
      const recaptchaAction = { action: "submit" };
      const token = await grecaptcha.execute(recaptchaSiteKey, recaptchaAction);
      setRecaptchaToken(token);
    });
  }, [recaptchaSiteKey, recaptchaToken, setRecaptchaToken]);

  return recaptchaToken;
};
```

</details>

<details markdown='1'>
<summary>src/common/useGetProtectedInfoHook.tsx</summary>

```javascript
import { useState, useEffect } from "react";
import { isObjectEmpty } from "./utils";

export const useGetProtectedInfoHook = (recaptchaToken: string) => {
  const [info, setInfo] = useState({});

  useEffect(() => {
    if (!recaptchaToken) {
      return;
    }

    if (!isObjectEmpty(info)) {
      return;
    }

    const requestHeaders = { recaptcha_token: recaptchaToken };
    fetch("http://localhost:3005/getItem", { headers: requestHeaders })
      .then(async (resp) => {
        const respJson = await resp.json();
        setInfo(respJson);
      })
      .catch(() => {});
  }, [info, setInfo, recaptchaToken]);

  return info;
};
```

</details>

**Backend Express.js Instance**

1. On `/getItem` endpoint request, validate reCAPTCHA token with Google reCAPTCHA `/siteverify`
2. Upon reCAPTCHA `/siteverify` response, use `success` and `score` values to determine token validity
3. Upon token invalidity, return `403 Forbidden`
4. Upon token validity, return sensitive item data

Code:

<details markdown='1'>
<summary>backend/app.js</summary>

```javascript
const express = require("express");
const cors = require("cors");
const getItemHandler = require("./getItemHandler");

const BACKEND_PORT = process.env.BACKEND_PORT;

const runApp = () => {
  const app = express();
  // cors to allow local setup
  const corsMiddleware = cors();

  app.get("/getItem", corsMiddleware, (req, res) => {
    getItemHandler(req, res);
  });

  // for preflight CORS request
  app.options("/getItem", corsMiddleware, (_, res) => {
    res.status(204);
    res.end();
  });

  app.listen(BACKEND_PORT, () =>
    console.log(`Example app listening at http://localhost:${BACKEND_PORT}`)
  );

  return app;
};

module.exports = runApp;
```

</details>

<details markdown='1'>
<summary>backend/getItemHandler.js</summary>

```javascript
const validateRecaptchaToken = require("./validateRecaptchaToken");

const getItemHandler = async (req, res) => {
  const { recaptcha_token } = req.headers;

  const isRecaptchaTokenPresent = !!recaptcha_token;
  if (isRecaptchaTokenPresent) {
    await getProtectedData(req, res);
    return;
  }

  getUnprotectedData(req, res);
};

const getUnprotectedData = (_, res) => {
  const item = { item: { name: "Beyerdynamic DT 1350" } };
  res.status(200);
  res.json(item);
};

const getProtectedData = async (req, res) => {
  const { recaptcha_token } = req.headers;
  const isValidToken = await validateRecaptchaToken(recaptcha_token);
  if (!isValidToken) {
    res.status(403);
    res.end();
    return;
  }

  const item = { item: { name: "Beyerdynamic DT 1350", price: "123" } };
  res.status(200);
  res.json(item);
};

module.exports = getItemHandler;
```

</details>

<details markdown='1'>
<summary>backend/validateRecaptchaToken.js</summary>

```javascript
const fetch = require("node-fetch");
const { RECAPTCHA_SECRET_KEY } = require("./recaptchaEnvVars");

const validateRecaptchaToken = async (token) => {
  if (!token) {
    return false;
  }

  const recaptchaOptions = {
    secret: RECAPTCHA_SECRET_KEY,
    response: token,
  };
  const fetchOptions = {
    method: "POST",
    body: `secret=${recaptchaOptions.secret}&response=${recaptchaOptions.response}`,
    headers: { "Content-type": "application/x-www-form-urlencoded" },
  };

  try {
    const resp = await fetch(
      "https://www.google.com/recaptcha/api/siteverify",
      fetchOptions
    );
    const respJson = await resp.json();
    const { success, score } = respJson;

    const isValidRecaptchaAttempt = success && score > 0.5;
    return isValidRecaptchaAttempt;
  } catch (e) {
    return false;
  }
};

module.exports = validateRecaptchaToken;
```

</details>

#### Developer Experience Considerations

**Considerations:**

- As a developer, I should be able to make request to an endpoint (eg. `/item/123`)
  - Without a reCAPTCHA token, I should only retrieve non-sensitive data
  - With a reCAPTCHA token, I should retrieve non-sensitive and sensitive data
    - If reCAPTCHA token is invalid, I should get a `403 Forbidden` response

**Operating principle:**

- A **single endpoint** should be able to serve **both** _non-sensitive_ and _sensitive_ data.
- If we have two endpoints (ie. one for non-sensitive data, and another for sensitive data), you would essentially **double** the number of endpoints to support for each resource 😩

# Summary

All in all, Next.js SSR + reCAPTCHA integration is pretty doable! 👍

- You get the benefits of fast renders and better SEO performance 🚀
- You get to protect your sensitive data 🕶

Happy building!
