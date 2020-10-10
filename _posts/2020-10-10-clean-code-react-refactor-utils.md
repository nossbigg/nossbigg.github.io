---
layout: post
title: "Clean Code React: Refactor your utils.js/ts!"
date: 2020-10-10 20:00:00 +0800
tags: [clean-code, react]
---

<center>
<img src="/assets/2020-10-10-clean-code-react-refactor-utils/paint-organized-mess.jpg" />
<br />
<br />
<i>"Hey, I take pride in MY organized mess!"</i> (<a href="https://pixabay.com/photos/brush-color-colors-colours-craft-1838983/">source</a>)
</center>
<br />

# Problem

<img src="/assets/2020-10-10-clean-code-react-refactor-utils/utils-large-file.png" />

Your `utils.ts` has become 200+ lines long 😨🤦‍♂️

# Triage

> "Wait, how did we end up here?"

- As we **add more functionality** to our React components, our files get **bigger** (of course they do)
- A standard strategy to **combat growing file sizes** is to move **non-JSX** logic into a companion `utils.ts` that sits in the same folder as the initial React component.
- Over a couple of sprints, this `utils.ts` gets stuffed with **more and more** functions until we get an (extremely) **bloated** `utils.ts`. _Foie gras, anyone?_ 🦆

<br />

> "Surely it's not that bad, right? 💁‍♀️"

<center>
<img src="/assets/2020-10-10-clean-code-react-refactor-utils/this-is-fine-kc-green.jpg" />
<br />
<br />
<i>"This is fine"</i>
<br />
Illustration by <a href="http://kcgreendotcom.com/index.html">KC Green</a>
</center>
<br />

No, it's bad - _**Really bad** stuff_. Here's why:

- **Increased entropy**: Chance of cross-context function calls (and we don't have `import`/`require` syntax to rely on as hints)
- **Increased fatigue**: Extremely fatiguing for developers to reason about multiple contexts, even if code does not cross contexts
- **Increased risk of errors**: High risk of human error due to high volume of logic and multiple contexts within same file
- **Poor discoverability**: Context information is lost under generic `utils.ts` filename
- **Large test suite**: Tests for all contexts within `utils.ts` would usually sit under `utils.test.ts` (unless you've already taken the effort to break it down into `__test__/context_xyz.test.ts`)
- **Poor cohesion**: Low overall cohesion (even if code is grouped context-wise), since a single file contains code which encompasses multiple contexts

# Solution

Steps:

- **Refactor** `utils.ts` into smaller files within `/utils` folder
- **Use** meaningful names for each file to indicate context
- **Colocate** associated helper functions to specific contexts
- **Export** utils as a group via `utils/index.ts`

Example code: [refactor-your-utils](https://github.com/nossbigg/clean-code-react/tree/main/src/examples/refactor-your-utils)

**Before:**

<details markdown='1'>
<summary>utils.ts</summary>

```javascript
import { MyItem, MyItemPriceCurrency } from "../common/typedefs";

// item price utils
const sortItemsByPriceAmountAscending = (items: MyItem[]): MyItem[] =>
  items.sort((a, b) => a.price.amount - b.price.amount);

export const getCheapestItem = (items: MyItem[]): MyItem => {
  return sortItemsByPriceAmountAscending(items)[0];
};

export const getMostExpensiveItem = (items: MyItem[]): MyItem => {
  return sortItemsByPriceAmountAscending(items).reverse()[0];
};

// item currency utils
export const filterBySgdItem = (items: MyItem[]) =>
  items.filter(filterByItem("SGD"));

export const filterByJpyItem = (items: MyItem[]) =>
  items.filter(filterByItem("JPY"));

const filterByItem = (targetCurrency: MyItemPriceCurrency) => (item: MyItem) =>
  item.price.currency === targetCurrency;

// item component utils
export const makeItemComponentKey = (item: MyItem) => item.id;
```

</details>

<details markdown='1'>
<summary>utils.test.ts</summary>

```javascript
import { MyItem } from "../common/typedefs";
import {
  filterByJpyItem,
  filterBySgdItem,
  getCheapestItem,
  getMostExpensiveItem,
  makeItemComponentKey,
} from "./utils";

describe("utils", () => {
  const appleItem: MyItem = {
    id: "abcd",
    title: "Apple",
    price: { amount: 12.3, currency: "SGD" },
  };
  const pearItem: MyItem = {
    id: "efgh",
    title: "Pear",
    price: { amount: 10.23, currency: "SGD" },
  };
  const cornItem: MyItem = {
    id: "ijkl",
    title: "Corn",
    price: { amount: 30.01, currency: "JPY" },
  };
  const mockItems: MyItem[] = [appleItem, pearItem, cornItem];

  it("gets cheapest item", () => {
    expect(getCheapestItem(mockItems)).toEqual(pearItem);
  });

  it("gets most expensive item", () => {
    expect(getMostExpensiveItem(mockItems)).toEqual(cornItem);
  });

  it("filters by sgd items", () => {
    expect(filterBySgdItem(mockItems)).toEqual([appleItem, pearItem]);
  });

  it("filters by jpy items", () => {
    expect(filterByJpyItem(mockItems)).toEqual([cornItem]);
  });

  it("makes item component key", () => {
    expect(makeItemComponentKey(appleItem)).toEqual("abcd");
  });
});
```

</details>

<br />
**After:**

<details markdown='1'>
<summary>utils/itemFilterUtils.ts</summary>

```javascript
import { MyItem, MyItemPriceCurrency } from "../../common/typedefs";

export const filterBySgdItem = (items: MyItem[]) =>
  items.filter(filterByItem("SGD"));

export const filterByJpyItem = (items: MyItem[]) =>
  items.filter(filterByItem("JPY"));

const filterByItem = (targetCurrency: MyItemPriceCurrency) => (item: MyItem) =>
  item.price.currency === targetCurrency;
```

</details>

<details markdown='1'>
<summary>utils/itemSortUtils.ts</summary>

```javascript
import { MyItem } from "../../common/typedefs";

const sortItemsByPriceAmountAscending = (items: MyItem[]): MyItem[] =>
  items.sort((a, b) => a.price.amount - b.price.amount);

export const getCheapestItem = (items: MyItem[]): MyItem => {
  return sortItemsByPriceAmountAscending(items)[0];
};

export const getMostExpensiveItem = (items: MyItem[]): MyItem => {
  return sortItemsByPriceAmountAscending(items).reverse()[0];
};
```

</details>

<details markdown='1'>
<summary>utils/itemRenderUtils.ts</summary>

```javascript
import { MyItem } from "../../common/typedefs";

export const makeItemComponentKey = (item: MyItem) => item.id;
```

</details>

<details markdown='1'>
<summary>utils/index.ts</summary>

```javascript
export * from "./itemFilterUtils";
export * from "./itemRenderUtils";
export * from "./itemSortUtils";
```

</details>

<details markdown='1'>
<summary>utils/__test__/itemFilterUtils.test.ts</summary>

```javascript
import { filterByJpyItem, filterBySgdItem } from "../itemFilterUtils";
import { testMocks } from "./testMocks";

describe("itemFilterUtils", () => {
  const { mockItems, appleItem, pearItem, cornItem } = testMocks;

  it("filters by sgd items", () => {
    expect(filterBySgdItem(mockItems)).toEqual([appleItem, pearItem]);
  });

  it("filters by jpy items", () => {
    expect(filterByJpyItem(mockItems)).toEqual([cornItem]);
  });
});
```

</details>

<details markdown='1'>
<summary>utils/__test__/itemSortUtils.test.ts</summary>

```javascript
import { getCheapestItem, getMostExpensiveItem } from "../itemSortUtils";
import { testMocks } from "./testMocks";

describe("itemSortUtils", () => {
  const { mockItems, pearItem, cornItem } = testMocks;

  it("gets cheapest item", () => {
    expect(getCheapestItem(mockItems)).toEqual(pearItem);
  });

  it("gets most expensive item", () => {
    expect(getMostExpensiveItem(mockItems)).toEqual(cornItem);
  });
});
```

</details>

<details markdown='1'>
<summary>utils/__test__/itemRenderUtils.test.ts</summary>

```javascript
import { makeItemComponentKey } from "../itemRenderUtils";
import { testMocks } from "./testMocks";

describe("itemRenderUtils", () => {
  const { appleItem } = testMocks;

  it("makes item component key", () => {
    expect(makeItemComponentKey(appleItem)).toEqual("abcd");
  });
});
```

</details>

<br />

# Summary

Here's why you should **refactor** (read: _kill_) all your `utils.ts`:

- **Low entropy**: Colocation of context-specific core and helper functions help to reduce cross-context function calls
- **Less fatigue**: Single-context files are far easier to reason about
- **Less risk of errors**: Narrower contexts reduces risk of human error
- **Good discoverability**: Well-named files help with overall discoverability of project
- **Small test suites**: Smaller test suites are easier to maintain
- **Good cohesion**: High file-level cohesion, since a single file contains only code that is specific to a single context

As always, keep your code ⭐️ _sparkling clean!_ ⭐️ &nbsp;Peace out ✌️
