---
layout: post
title: "Experience Report: eslint-plugin-fp"
date: 2020-10-08 20:00:00 +0800
tags: [experience-report]
---

<center>
<img src="/assets/2020-10-07-experience-report-eslint-plugin-fp/eslint-fp-colleagues-dislike-me.jpg" />
<br />
<br />
<i>"Why do my colleagues dislike me?"</i>
</center>
<br />

# What is 'eslint-plugin-fp'?

[`eslint-plugin-fp`](https://github.com/jfmengels/eslint-plugin-fp) is an [`eslint`](https://eslint.org/) plugin that enforces **functional programming principles and rules** on your JavaScript/TypeScript code. (eg. _no mutations, no classes, no loops_)

# A gentle prologue

For React `class` components, it can be a... _bloodbath_ 😨

Consider the following: `MyComponent`

**Before:**

<img src="/assets/2020-10-07-experience-report-eslint-plugin-fp/react-cc-bloodbath.png" />

Okay, so here's what went _wrong_, according to `eslint-plugin-fp`:

- It's a class component. **No classes**
- It's using `this`. **No `this`**

Let's remedy this situation 🔧

**After:**

<img src="/assets/2020-10-07-experience-report-eslint-plugin-fp/react-fc-better.png" />

Lint passed! 🧼

# The 4 Horsemen of the (Functional Programming) Apocalypse

Since it is not a stretch to surmise that this linter would _lay waste_ to most normal [OO](https://en.wikipedia.org/wiki/Object-oriented_programming)-centric codebases...

Here's **4 common issues** that you'd face: (coupled with their corresponding antidotes, of course 🧪)

<details markdown="1">
<summary><b>1.</b> <i>no-class</i></summary>

**Before:**

```javascript
type MyReactComponentState = { counter: number };

/* eslint-disable fp/no-class */
/* eslint-disable fp/no-mutation */
/* eslint-disable fp/no-this */
/* eslint-disable fp/no-let */
export class MyReactComponent extends React.Component<
  {},
  MyReactComponentState
> {
  constructor(props) {
    super(props);
    this.state = { counter: 0 };
  }

  incrementCounter() {
    this.setState((previousState) => {
      return { counter: previousState.counter + 1 };
    });
  }

  render() {
    return (
      <div>
        <span>Counter: {this.state.counter}</span>
        <button
          data-testid="button-increment"
          onClick={() => this.incrementCounter()}
        >
          Increment
        </button>
      </div>
    );
  }
}
```

**After:**

```javascript
export const MyReactComponentEdit: React.FC = () => {
  const [counter, setCounter] = useState(0);

  return (
    <div>
      <span>Counter: {counter}</span>
      <button
        data-testid="button-increment"
        onClick={() => setCounter(counter + 1)}
      >
        Increment
      </button>
    </div>
  );
};
```

</details>

<details markdown="1">
<summary><b>2.</b> <i>no-this + no-mutation</i></summary>

**Before:**

```javascript
/* eslint-disable fp/no-this */
/* eslint-disable fp/no-mutation */
/* eslint-disable fp/no-class */
export class MyNameClass {
  name: string;

  constructor(name: string) {
    this.name = name;
  }

  getNamePrompt(): string {
    return `my name is ${this.name}`;
  }
}
```

**After:**

```javascript
type MyNameClassEdit = { getNamePrompt: () => string };

export const makeMyNameClassEdit = (name: string): MyNameClassEdit => {
  const getNamePrompt = (): string => {
    return `my name is ${name}`;
  };

  return { getNamePrompt };
};
```

</details>

<details markdown="1">
<summary><b>3.</b> <i>no-loops</i></summary>

**Before:**

```javascript
/* eslint-disable fp/no-mutation */
/* eslint-disable fp/no-let */
/* eslint-disable fp/no-loops */
/* eslint-disable fp/no-mutating-methods */
export const capitalizeOddIndexChars = (input: string): string => {
  const splittedInput = input.split("");
  const withOddIndexCapitalized = [];

  for (let index = 0; index < splittedInput.length; index++) {
    let currentCharacter = splittedInput[index];

    const isOddIndex = index % 2 !== 0;
    if (isOddIndex) {
      currentCharacter = currentCharacter.toUpperCase();
    }

    withOddIndexCapitalized.push(currentCharacter);
  }

  const result = withOddIndexCapitalized.join("");
  return result;
};
```

**After:**

```javascript
export const capitalizeOddIndexChars_edit = (input: string): string => {
  const splittedInput = input.split("");

  const withOddIndexCapitalized = splittedInput.map(
    (currentCharacter, index) => {
      const isOddIndex = index % 2 !== 0;
      if (!isOddIndex) {
        return currentCharacter;
      }
      return currentCharacter.toUpperCase();
    }
  );

  const result = withOddIndexCapitalized.join("");
  return result;
};
```

Instead of using the `for` loop, we opt for `.map()` instead.

</details>

<details markdown="1">
<summary><b>4.</b> <i>no-throw</i></summary>

**Before:**

```javascript
/* eslint-disable fp/no-throw */
export const validateItems = (items: MyItem[]): boolean => {
  items.forEach((item) => {
    const { name, price } = item;
    if (price > PRICE_THRESHOLD) {
      throw new Error(`item '${name}' price exceeded`);
    }
  });

  return true;
};
```

**After:**

```javascript
type ValidateItemsEditResult = { pass: boolean, errMessage: string };

export const validateItems_edit = (
  items: MyItem[]
): ValidateItemsEditResult => {
  const initialResult: ValidateItemsEditResult = { pass: true, errMessage: "" };

  const result: ValidateItemsEditResult = items.reduce((acc, item) => {
    const { name, price } = item;
    if (price > PRICE_THRESHOLD) {
      return { pass: false, errMessage: `item '${name}' price exceeded` };
    }
    return acc;
  }, initialResult);

  return result;
};
```

Note: For this example, we are emulating the _multi-valued return_ technique, which is more commonly found in [Python](https://rosettacode.org/wiki/Return_multiple_values#Python) and [Golang](https://rosettacode.org/wiki/Return_multiple_values#Go)

</details>

Example code: [exp-report-eslint-plugin-fp](https://github.com/nossbigg/clean-code-react/tree/main/src/examples/exp-report-eslint-plugin-fp)

# Recommendations

_Personally_, I think it boils down to whether you favor the **functional programming code style**. If you're a fan of it...

<center>
I'd <b>highly recommend</b> using this plugin! 🤩
</center>
<br />

_PS: You don't have to drink all the functional programming kool-aid to use it (read: `monads`, `applicative`, `functors`). I'd wager you'd still get **a lot** of mileage out of this plugin, even if it enforces just a small subset of functional programming principles._

# Parting Thoughts

If you do intend to use it at work, **be helpful** in helping your peers to write in this code style - I'd admit that it's quite a paradigm shift, so be helpful! 🤗

Just don't make your peers go...

<br />
<center>
<img src="/assets/2020-10-07-experience-report-eslint-plugin-fp/eslint-fp-errors.png" />
<br />
↓
<br />
<img src="/assets/2020-10-07-experience-report-eslint-plugin-fp/agk-keyboard-smash.gif" />
</center>
<br />

_Note: I am not responsible if your cultural performance rating drops because of this_ 😂

Until then, happy hacking! 🤓

<br />

# Appendix: Dev notes

<details markdown="1">
<summary>See More</summary>

For most JS/TS codebases:

1. You'll probably have to disable `no-nil`, as there's a legitimate use case not to return a value at the end of a function.

   - _eg. canonical Jest test structure doesn't require a return value at the end of every `describe()` and `test()` block._

2. You'll probably have to disable `no-unused-expression`, as almost _all_ React projects involving using library functions without using the function's return value - these functions are often used to invoke side effects.

   - eg. `ReactDOM.render()` or `serviceWorker.register()`

</details>
