---
layout: post
title: "Clean Code React: Use concise snapshots!"
date: 2020-10-05 15:00:00 +0800
tags: [clean-code, react]
---

<center>
<img src="/assets/2020-10-05-clean-code-react-concise-snapshots/really-long-snapshot.jpg" />
<br />
<br />
<i>"809 lines? Really?"</i>
</center>
<br />

# Problem

We've all been there: `.snapshot` files with 800+ lines 🙄🤦‍♂️

# Solution

> "_The understandability of your snapshot is inversely proportional to the length of the snapshot._"

<br />
<center><b>Keep your snapshots small (eg. <50 lines). The smaller, the better!</b></center>
<br />

With small snapshots:

- Easier to see what's changed (since diffs are small 🐤)

With enormous snapshots:

- Harder to see what's changed, especially when dealing with large diffs (read: _100+ lines_ of changes 😭)
- Far more likely to **fall into misuse**, as other developers may simply update the snapshot **without** fully understanding its implications.

# Examples

Example code: [concise-snapshots](https://github.com/nossbigg/clean-code-react/tree/main/src/examples/concise-snapshots)

For example, given this `MyDatePicker` React component:

```javascript
import React from "react";
import { DatePicker } from "antd";

type MyDatePickerProps = { placeholder: string };

export const MyDatePicker: React.FC<MyDatePickerProps> = (props) => {
  const { placeholder } = props;
  return <DatePicker placeholder={placeholder} picker="month" />;
};
```

**Bad**

Running this following test case, which uses `enzyme` `mount`:

```javascript
it("bad: creates an unmaintanably large snapshot", () => {
  const wrapper = mount(<MyDatePicker placeholder="some-placeholder" />);
  expect(wrapper).toMatchSnapshot();
});
```

yields the following snapshot:

<img src="/assets/2020-10-05-clean-code-react-concise-snapshots/long-snapshot.jpg" />

That's **51 lines** of snapshot for just **4 lines** of production code! 😢

This is because we used `mount`, which causes the actual `Antd.DatePicker` component to be rendered.

By using `mount`, we have thus exposed the _implementation details_ of the `Antd.DatePicker` component - this introduces a lot of **noise** in our snapshots.

Remember, we would like to only test **our own code** that we have written, and not `Antd.DatePicker`'s code.

**Better**

Instead, we can do this - by using `enzyme` `shallow`:

```javascript
it("matches snapshot", () => {
  const wrapper = shallow(<MyDatePicker placeholder="some-placeholder" />);
  expect(wrapper).toMatchSnapshot();
});
```

yields the following snapshot:

<img src="/assets/2020-10-05-clean-code-react-concise-snapshots/short-snapshot.jpg" />

We went from 51 lines to just **6 lines**. Now, that's far more maintainable! 🥳

Importantly, we have ensured that the `props` that we have passed to `Antd.DatePicker` is asserted against 👍

**Best**

Snapshots should always be used with **descriptive tests** that elucidates the functionality of the component.

```javascript
it("passes correct props to DatePicker", () => {
  const wrapper = shallow(<MyDatePicker placeholder="some-placeholder" />);

  const datePicker = wrapper.find(DatePicker);
  expect(datePicker.exists()).toBe(true);

  const expectedProps = { picker: "month", placeholder: "some-placeholder" };
  expect(datePicker.props()).toEqual(expectedProps);
});
```

Trust me, you **don't want** to work on a new codebase that _only_ has snapshots 😂
