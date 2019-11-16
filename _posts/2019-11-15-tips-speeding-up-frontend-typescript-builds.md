---
layout: post
title:  "Tips on Speeding Up Frontend TypeScript Builds"
date:   2019-11-15 13:00:00 +0800
categories: frontend
---
<center>
<img src="/assets/2019-11-15-tips-speeding-up-frontend-typescript-builds/god-banishes-adam-eve-from-eden.jpg" height="400"/>
<br />
<i>God banishes Adam and Eve from the Garden of Eden</i> (<a href="https://www.jw.org/en/library/books/bible-stories-lessons/2/adam-and-eve-disobeyed-god/">source</a>)
</center>
<br />

# Prologue

<i><sup>4</sup> But the serpent said to the woman, “You will not surely die.</i> 

<i><sup>5</sup> For God knows that when you eat of it your eyes will be opened, and you will be like God, knowing good and evil.”</i>

<i>Genesis 3</i>

<br />

Having `react-script eject`-ed ourselves from the Eden of `create-react-app`, here's a few tips on how to speed up your Frontend Typescript builds! 🙃🐑 

# Tips

#### 1. Profile your webpack `build` using `speed-measure-webpack-plugin` (or equivalent)

<img src="/assets/2019-11-15-tips-speeding-up-frontend-typescript-builds/speed-measure-webpack-plugin-sample.png"/>

- Use `speed-measure-webpack-plugin` to get insight into the webpack `build` step.
- Tackle the loaders/plugins that take the longest times (if possible)

See more:
- `speed-measure-webpack-plugin` [(link)](https://github.com/stephencookdev/speed-measure-webpack-plugin)


#### 2. Remove `eslint-loader` (or equivalent) during webpack dev/prd build steps

- Linters can slow down your build times significantly once your project becomes sufficiently large.
- This is especially painful for developers, as:
```
longer build times = longer feedback loops = less productivity
```
- Do the linting elsewhere - eg. `git hooks` or `CI stages` - instead of slowing down the build with a linter.

_Example: `package.json`: pre-commit linting via `husky` git hooks + `eslint`_
```json
{
    "scripts": {
        "lint": "eslint 'src/**/*.{js,jsx,ts,tsx}'"
    },
    "husky": {
        "hooks": {
            "pre-commit": "yarn lint"
        }
    }
}
```

#### 3. Diagnose `tsc` 

<img src="/assets/2019-11-15-tips-speeding-up-frontend-typescript-builds/tsc-diagnostics-sample.png"/>

- Use `tsc -diagnostics` to get insight into the `tsc` compile step
- Use `tsc --listFiles` to see what files `tsc` is using for the compile step
- If there are extra files that aren't required for the build, tweak the `files`/`include`/`exclude` properties in `tsconfig.json` to see if you can exclude these files, as:

```
Fewer files = less code to transpile = shorter build times
```

_Example: `tsconfig.json`: Include all source files but exclude all tests (ie. `*.spec.ts`) (reference: official documentation)_
```json
{
    "include": [
        "src/**/*"
    ],
    "exclude": [
        "node_modules",
        "**/*.spec.ts"
    ]
}
```

See more: 
- [Compiler Options · TypeScript](https://www.typescriptlang.org/docs/handbook/compiler-options.html)
- [tsconfig.json · TypeScript](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html)

#### 4. Try other TypeScript loaders

- Try switching to other TypeScript loaders to see if you can get faster build times.
- My colleague swapped out `awesome-typescript-loader` with `ts-loader` for our project, and that alone **halved** the build time!
- Don't forget to look into the configuration settings the loader that you're using to squeeze more time savings out from it.

_Example: Configuration for `awesome-typescript-loader`_
```json
{
    "useCache": true,
    "forceIsolatedModules": true,
    "transpileOnly": true
}
```

See more:
- `ts-loader` [(link)](https://github.com/TypeStrong/ts-loader)
- `awesome-typescript-loader` [(link)](https://github.com/s-panferov/awesome-typescript-loader)

#### 5. Use Babel to transpile your entire project

- Since Babel 7, Babel is able to natively transpile TypeScript into JavaScript.
- You can experiment with a Babel-first approach to building your project (instead of `tsc` as the usual default for TypeScript projects)
- To do this, just remove your existing TypeScript loader and configure `babel-loader` to read your `.ts/.tsx` files.

_Example: `webpack.config.js`: `babel-loader` accepting `.tsx` files (reference: [react-scripts's `webpack.config.js`](https://github.com/facebook/create-react-app/blob/master/packages/react-scripts/config/webpack.config.js))_
```javascript
{
    module: {
        rules: [
            {
              test: /\.(js|mjs|jsx|ts|tsx)$/,
              include: paths.appSrc,
              loader: require.resolve('babel-loader'),
            }
        ]
    }
}
```

- Caveat: Babel **won't** typecheck your code, so it's prudent to 'typecheck' the project.

_Example: `package.json`: `typecheck` script_
```json
{
    "scripts": {
        "typecheck": "tsc --noEmit"
    }
}
```

See more:
- [Babel 7 Released · Babel](https://babeljs.io/blog/2018/08/27/7.0.0#typescript-support-babel-preset-typescript)
- [TypeScript and Babel 7 \| TypeScript](https://devblogs.microsoft.com/typescript/typescript-and-babel-7/)
- [Choosing between Babel and TypeScript - LogRocket Blog](https://blog.logrocket.com/choosing-between-babel-and-typescript-4ed1ad563e41/)

# Footnote

While understanding the full build process may be daunting for greenhorns, keep experimenting with different methods to speed up build times, all whilst staying curious as to how these different techniques work. Happy grokking! ⛏