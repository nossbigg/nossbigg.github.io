---
layout: post
title: "JS: OO/FP Composition Styles"
date: 2020-10-19 19:00:00 +0800
tags: [javascript, functional-programming]
---

<center>
<img src="/assets/2020-10-19-js-oo-fp-composition-styles/js-one-way-laughing.png" />
<br />
<br />
<i>"haha javascript go brrrrrrr"</i>
</center>
<br />

# Preface

> Point 13: There should be one-- and preferably only one --obvious way to do it.
> <br/>- [The Zen of Python](https://www.python.org/dev/peps/pep-0020/)

We JavaScript developers _get it_ - There's a lot of different ways to write JavaScript code for any given task, perhaps too many ways 😩

As writing code involves - to a very large extent - **composing** (ie. combining) code in certain ways, let us explore how this looks like in JavaScript for both **Object-Oriented** (OO) and **Functional Programming** (FP) styles.

**Important Concepts**

Under the hood, JavaScript **only** uses Prototypal Inheritance.

- JavaScript doesn't have _real_ Class-based Inheritance - JavaScript _emulates_ Class-based Inheritance through the use of _the prototype chain_.

**Attribution**

This article draws heavily on Eric Elliott's excellent writeup on [Common Misconceptions About Inheritance in JavaScript](https://medium.com/javascript-scene/common-misconceptions-about-inheritance-in-javascript-d5d9bab29b0a). I highly recommend that you read it too! Also, thanks Eric!

# Context

The code examples are built around this problem space:

1. Declare an `Animal` _base type/parent class_ that supports
   - `name:string`
   - `getName: () => string`
2. Declare a `Dog` _type/subclass_ that supports
   - `Animal` interactions
   - `tricks: string[]`
   - `getTricks: () => string[]`
3. Declare a `Cat` _type/subclass_ that supports
   - `Animal` interactions
   - `isSprayed: boolean`
   - `checkIsSprayed: () => boolean`

Language: `TypeScript`

Example code: [oo-fp-composition](https://github.com/nossbigg/clean-code-react/tree/main/src/examples/oo-fp-composition)

# JS: OO Styles

#### 1. Delegation: ES6 `class`

```javascript
class Animal {
  name: string;

  constructor(name: string) {
    this.name = name;
  }

  getName(): string {
    return this.name;
  }
}

class Dog extends Animal {
  tricks: string[];

  constructor(name: string, tricks: string[]) {
    super(name);
    this.tricks = tricks;
  }

  getTricks(): string[] {
    return this.tricks;
  }
}

class Cat extends Animal {
  isSprayed: boolean;

  constructor(name: string, isSprayed: boolean) {
    super(name);
    this.isSprayed = isSprayed;
  }

  checkIsSprayed(): boolean {
    return this.isSprayed;
  }
}

export const jetDog = new Dog("Jet", ["fetch"]);

export const jonCat = new Cat("Jon", true);
```

This is _the_ canonical OO Class-based Inheritance syntax - No surprises here.

The `class` syntax is actually **a huge boon for everyone**, since it **hides away the complexities** and **standardizes** the implementation of Class-based Inheritance in JavaScript - Developers no longer have to handwrite this, which can be an error-prone process.

#### 2. Delegation: `proto`

```javascript
const animalProto = {
  getName(): string {
    return (this as any).name;
  },
};

const dogMixin = {
  getTricks(): string[] {
    return (this as any).tricks;
  },
};

const catMixin = {
  checkIsSprayed(): boolean {
    return (this as any).isSprayed;
  },
};

const makeDog = (name: string, tricks: string[]) => {
  const dog = Object.assign(Object.create(animalProto), dogMixin, {
    name,
    tricks,
  });
  return dog;
};

const makeCat = (name: string, isSprayed: boolean) => {
  const cat = Object.assign(Object.create(animalProto), catMixin, {
    name,
    isSprayed,
  });
  return cat;
};

export const jetDog = makeDog("Jet", ["fetch"]);

export const jonCat = makeCat("Jon", true);
```

Here's what's going on:

1. Use `Object.create()` to create a new object with `animalProto` as its prototype, thereby reusing the `getName()` logic.
2. Assign `dogMixin/catMixin` to new object to add instance methods to object
3. Assign instance variables (eg. `name`, `tricks, `isSprayed`) to make instance variables available to instance methods.

Key points to note:

- Instance methods access instance variables through the own object's `this` reference.
- For `getName()` the `Dog`/`Cat` objects access its functionality through _climbing down the prototype chain_.
- For `getTricks()`/`checkIsSprayed()`, the `Dog`/`Cat` objects access its functionality through the _object's own immediate property_ (**no** delegation required)

#### 3. Mixin - `Object.assign()`

```javascript
const animalMixin = {
  getName(): string {
    return (this as any).name;
  },
};

const dogMixin = {
  getTricks(): string[] {
    return (this as any).tricks;
  },
};

const catMixin = {
  checkIsSprayed(): boolean {
    return (this as any).isSprayed;
  },
};

const makeDog = (name: string, tricks: string[]) => {
  const dog = Object.assign({}, animalMixin, dogMixin, { name, tricks });
  return dog;
};

const makeCat = (name: string, isSprayed: boolean) => {
  const cat = Object.assign({}, animalMixin, catMixin, { name, isSprayed });
  return cat;
};

export const jetDog = makeDog("Jet", ["fetch"]);

export const jonCat = makeCat("Jon", true);
```

The key difference between OO Delegate `proto` and OO Mixin is that the `getName()` functionality is no longer accessed by prototype chain, but as a method with the object's own property.

#### 4. Functional Inheritance

```javascript
const animalMixin = (name: string) => {
  return {
    getName(): string {
      return name;
    },
  };
};

const dogMixin = (tricks: string[]) => {
  return {
    getTricks(): string[] {
      return tricks;
    },
  };
};

const catMixin = (isSprayed: boolean) => {
  return {
    checkIsSprayed(): boolean {
      return isSprayed;
    },
  };
};

const makeDog = (name: string, tricks: string[]) => {
  const dog = Object.assign({}, animalMixin(name), dogMixin(tricks));
  return dog;
};

const makeCat = (name: string, isSprayed: boolean) => {
  const cat = Object.assign({}, animalMixin(name), catMixin(isSprayed));
  return cat;
};

export const jetDog = makeDog("Jet", ["fetch"]);

export const jonCat = makeCat("Jon", true);
```

This example takes the OO Mixin a step further by implementing **data hiding** - external code can no longer access the instance variables of the objects. In fact, the objects don't have instance variables, but just instance methods!

# JS: FP Styles

#### 1. Pointfree

```javascript
export type AnimalType = { name: string };

export type DogType = AnimalType & { tricks: string[] };

export type CatType = AnimalType & { isSprayed: boolean };

export const getName = (a: AnimalType): string => a.name;

export const getTricks = (d: DogType): string[] => d.tricks;

export const checkIsSprayed = (c: CatType): boolean => c.isSprayed;

const makeDog = (name: string, tricks: string[]): DogType => {
  return {
    name,
    tricks,
  };
};

const makeCat = (name: string, isSprayed: boolean): CatType => {
  return {
    name,
    isSprayed,
  };
};

export const jetDog: DogType = makeDog("Jet", ["fetch"]);

export const jonCat: CatType = makeCat("Jon", true);
```

In pointfree style, objects are pure data structures with neither instance methods nor mutating methods.

All methods are implemented as functions that accept interfaces.

_Note: Ok, I know that strictly speaking, it should be `prop('tricks')`, and not `d.tricks`. Man, it's hard being in the FP club_ 😢

#### 2. Inline Handlers

```javascript
export type AnimalType = { name: string, getName(): string };

export type DogType = AnimalType & { tricks: string[], getTricks(): string[] };

export type CatType = AnimalType & {
  isSprayed: boolean,
  checkIsSprayed(): boolean,
};

const makeGetNameHandler = (name: string) => () => name;

const makeDog = (name: string, tricks: string[]): DogType => {
  return {
    name,
    tricks,
    getName: makeGetNameHandler(name),
    getTricks: () => tricks,
  };
};

const makeCat = (name: string, isSprayed: boolean): CatType => {
  return {
    name,
    isSprayed,
    getName: makeGetNameHandler(name),
    checkIsSprayed: () => isSprayed,
  };
};

export const jetDog: DogType = makeDog("Jet", ["fetch"]);

export const jonCat: CatType = makeCat("Jon", true);
```

This pattern of **attaching functions to objects** enjoys high cohesion, and is quite a common pattern in JS codebases.

Also, this approach is _almost_ a 1-1 match to the OO Mixin style 🧐

# Observations & Takeaways

- OO Mixin and OO FI styles are **strikingly similar** to FP styles, which surprised me 🤔 (_gets off my FP high-horse_ 😂)
- OO Delegation ES6, in the long run, suffers from [**leaky abstraction**](https://en.wikipedia.org/wiki/Leaky_abstraction) and [**fragile base class**](https://en.wikipedia.org/wiki/Fragile_base_class) problems, as already alluded to in Eric Elliott's article.
- FP styles + OO Mixin + OO FI styles avoid the _two aforementioned problems_ because they allow you to include **only the exact functionality** in an object. **Composition over Inheritance [QED](https://en.wikipedia.org/wiki/Q.E.D.)** 🤩
- FP Pointfree is a better option in terms of **refactoring**, as you'd be able to refactor methods (which are smaller code units) into their own files eg. `getDogTricks.ts`, instead of refactoring handler generators to their own files eg. `makeDogMethods.ts`. This is useful for scenarios when methods get too big. _Alas, we're nitpicking here. Potato, Po-tah-toh_ 🤷‍♂️

  - I initially thought that FP Pointfree and FP Inline Handlers would turn out to be around the same, but alas, a worked example has given us better clarity 👍

- _In my humble opinion_, reusing functionality via Mixin style or FP styles are a **clearer** means of doing so vis-a-vis Delegation style, owing to a 'flatter' dependency structure.

  > Why bother with the prototype chain when you can just create an instance method as a property that exists at the same level of the object?

  > How about achieving the same outcome by declaring methods that exist outside of the object, ala pointfree style?
