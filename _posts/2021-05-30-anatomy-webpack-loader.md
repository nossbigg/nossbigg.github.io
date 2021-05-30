---
layout: post
title: "Anatomy of a Webpack Loader"
date: 2021-05-30 08:00:00 +0800
tags: [webpack]
---

<center>
<img src="/assets/2021-05-30-anatomy-webpack-loader/open-end-wrench-2452245_640.jpg" />
<br />
<br />
(<a href="https://pixabay.com/photos/open-end-wrench-mechanic-repair-2452245/">source</a>)
</center>
<br />

This is my attempt at an **extremely abridged** version of explaining what a Webpack Loader does 😅

_aka. Webpack loader basics_

# Q: Why do we use Webpack Loaders?

Webpack Loaders help to <u>transform</u> <u>non-JavaScript imports</u> into <u>useable JavaScript values</u>.

Examples:

1. [`css-loader`](https://github.com/webpack-contrib/css-loader): Exposes `.css` **styling information** as a **JS object**
1. [`yaml-loader`](https://github.com/eemeli/yaml-loader): Exposes `.yaml` **file contents** as a **JS object**
1. [`file-loader`](https://github.com/webpack-contrib/file-loader): **Emits file** in build + exposes file as **static URL**

# Loader: Basic Structure

A webpack loader principally does **3 things**:

1. **Read contents of a file**

   - Loaders are used in **Webpack Rules** [(see more)](https://v4.webpack.js.org/configuration/module/#ruleuse)
   - Each rule contains options to **match/exclude files** (eg. `Rule.test`, `Rule.include`, `Rule.exclude`, etc...)
   - Matched files will be **fed** into the specified loader
   - When files are fed into a loader, the loader will have access to the **contents of each file**

   <details markdown="1">
   <summary>See more: Example Webpack Rule definition</summary>
   ```javascript
   // webpack.config.js
   module.exports = {
     module: {
       rules: [
         // other rules...
         {
           test: /locales\.json$/,
           type: "javascript/auto",
           loader: require.resolve("i18n-locales-loader"),
           options: {
             esModule: true,
             buildLocalesPath: ["static", "locales"],
           },
         },
         // other rules...
       ],
     },
   };
   ```
   </details>
   <br/>

2. **Export JavaScript variables**

   - It's usually useful to **expose** _some_ **JavaScript information** based on the input file content.
   - Eg. `yaml-loader`

   ```yaml
   # someYamlFile.yaml
   ---
   name: samuel
   ```

   1. Given `import yamlContent from './someYamlFile.yaml'` + `yaml-loader`,
   1. In order for `yamlContent` to contain the information from the YAML file (ie. `{"name": "samuel"}`),
   1. `yaml-loader` **reads** the YAML file [(see loader code)](https://github.com/eemeli/yaml-loader/blob/a0959a95005a55eddd76d4f45f99c7f56921a64f/index.js#L32),
   1. `yaml-loader` **exports** the YAML file contents using `export default` [(see loader code)](https://github.com/eemeli/yaml-loader/blob/a0959a95005a55eddd76d4f45f99c7f56921a64f/index.js#L51)

   <br/>

3. **Emit files**

   - Depending on the use case, a loader can **choose** to **emit file(s)**. (ie. not all loaders emit files.)
   - Eg. `file-loader`

   1. Given `import svgFile from './someSvgFile.svg'` + `file-loader`,
   1. In order for `svgFile` to contain a **string** referring to the **static asset's URL**,
   1. `file-loader` **emits** the `.svg` file [(see loader code)](https://github.com/webpack-contrib/file-loader/blob/59df66a7116cad786d45c1c2170aec66b6fa65b5/src/index.js#L81),
   1. `file-loader` **constructs** the static asset's URL [(see loader code)](https://github.com/webpack-contrib/file-loader/blob/59df66a7116cad786d45c1c2170aec66b6fa65b5/src/index.js#L36-L54),
   1. `file-loader` **exports** the URL as a string using `export default` [(see loader code)](https://github.com/webpack-contrib/file-loader/blob/59df66a7116cad786d45c1c2170aec66b6fa65b5/src/index.js#L87)

   <br/>

4. _(Bonus) Do anything else_

   - Technically, since a loader is written in JavaScript, a loader can contain other code for _telemetry_ or _build statistics_ purposes.

   <br/>

# Explained Example: `i18n-locales-loader`

_Note: Yes, this is a shameless plug of my own_ `i18n-locales-loader` _webpack loader_ 😂

Summary: `i18n-locales-loader` [(GitHub Repo)](https://github.com/nossbigg/i18n-locales-loader) splits up `locales.json` files into locale-specific files eg. `en.json`, `id.json`

**What it does:**

1. Parses `locales.json` to a JSON object [(see loader code)](https://github.com/nossbigg/i18n-locales-loader/blob/e71588f0ce3e8ad35b1d730a5c23faa626e222bd/src/i18n-locales-loader.js#L18)
2. Creates JSON object for every locale [(see loader code)](https://github.com/nossbigg/i18n-locales-loader/blob/e71588f0ce3e8ad35b1d730a5c23faa626e222bd/src/utils/makeLocalesMap.js#L16)
3. Generates static asset URL path for every locale [(see loader code)](https://github.com/nossbigg/i18n-locales-loader/blob/e71588f0ce3e8ad35b1d730a5c23faa626e222bd/src/utils/makeLocalesMap.js#L25)
4. Emits `.json` file for every locale [(see loader code)](https://github.com/nossbigg/i18n-locales-loader/blob/e71588f0ce3e8ad35b1d730a5c23faa626e222bd/src/utils/emitLocaleFiles.js#L7)
5. Creates JSON object containing locale to static asset URL path mapping [(see loader code)](https://github.com/nossbigg/i18n-locales-loader/blob/e71588f0ce3e8ad35b1d730a5c23faa626e222bd/src/utils/makeLocalesWebpathMap.js#L3-L14)
6. Exports JSON object using `export default` [(see loader code)](https://github.com/nossbigg/i18n-locales-loader/blob/e71588f0ce3e8ad35b1d730a5c23faa626e222bd/src/utils/makeModuleOutput.js#L1-L10)

# Summary and References

- For purposes of education, I have used _simpler loaders_ to more clearly illustrate the key principles of a webpack loader.
- For an example of the _full potential_ of a loader, you may take a look at [`vue-loader`](https://github.com/vuejs/vue-loader) and an explanation of [how it works](https://github.com/vuejs/vue-loader/tree/b179fb9c1b3bc0507f021be208c359d0dedeec08#how-it-works). (_Warning: It's very complex!_ 🤯)
- Other references:
  - [SurviveJS: Webpack](https://survivejs.com/webpack/foreword/): Good resource for a more rigorous treatment of explaining what Webpack does.
  - [Loaders: Webpack](https://v4.webpack.js.org/concepts/loaders/): Official Webpack documentation for Loaders
  - [Concepts: Webpack](https://v4.webpack.js.org/concepts/): Official Webpack documentation

I hope this article helps with **understanding** how webpack loaders work, and perhaps even help you in **crafting your own loaders**! 🙃
