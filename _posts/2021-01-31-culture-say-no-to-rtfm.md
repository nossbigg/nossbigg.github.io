---
layout: post
title: "Culture: Say No To RTFM"
date: 2021-01-31 08:00:00 +0800
tags: [culture]
---

<center>
<img src="/assets/2021-01-31-culture-say-no-to-rtfm/dublin-2344423_640.jpg" />
<br />
<br />
<i>"The business logic is already implemented in the codebase - go find it there."</i>
<br />
The codebase:
<br />
(<a href="https://pixabay.com/photos/dublin-trinity-college-library-2344423/">source</a>)
</center>
<br />

# Context

> "Please refer to the codebase, as all the business logic is already there."

We've all been there, and **it sucks.** 😩

- **Scouring wiki pages** scattered across the organization, often hitting many unrelated projects along the way and **still not getting what you want**
- **Reading stale documentation** that was correct probably 3 releases ago
- **Guessing (and second-guessing) the intent of code** amidst:
  - Poorly-named functions (eg. `computeSpecialFormattedText()`),
  - Poorly-named files (eg. `page.tsx`, `utils.ts`),
  - Reams of dead code, and
  - Code that spreads context too thinly across too many files

# Prologue: What's RTFM?

**RTFM**

> RTFM is an initialism and internet slang for the expression "read the f\*\*king manual" – typically used to reply to a question that has already been answered in the documentation, user guide, owner's manual, man page, online help, internet forum, software documentation or FAQ etc.

(source: [wikipedia](https://en.wikipedia.org/wiki/RTFM))

# Why doing context acquisition alone is hard

> "Working code captures the essence of any business logic in its entirety"

While the _"proof is indeed in the pudding"_, there are often **barriers** to the **comprehensibility of existing systems** due to:

- How workflows are **conceptualized at the business level**, and
- How workflows are **implemented and represented in code form**

Thus, it is not uncommon that such **"gaps"** can grow into **huge chasms** in short order, owing to extenuating **business** and **engineering** requirements such as:

- Failure to address **tech debt** due to high pressure to deliver new business features
- **Lack of clean code** (see: [Clean Code React: Refactor your utils.js/ts!](/2020/10/10/clean-code-react-refactor-utils.html))
- **Wrong abstractions** (eg: [Prefer duplication over the wrong abstraction](https://sandimetz.com/blog/2016/1/20/the-wrong-abstraction))
- For more ideas, **SourceMaking** has an [excellent resource on Antipatterns](https://sourcemaking.com/antipatterns)!

# How to make it better?

> "Engineers with the greatest contextual knowledge of their subsystems (eg. microservice, microfrontend) are in the best position to share their contexts"

Engineers with **contextual knowledge** are able to **direct newcomers** to the **right places** to look at with **surgical precision**, eg:

- What are the **repositories** involved? (eg. business logic spread across different microservices)
- What **files and lines of code** that contain said business logic?
- What are the **wiki articles** that describes said business logic?
- Who are the **right people to ask** (technical/engineering) for more details about certain features?

> "What are the right places to look at?"

**"Looking at the right places"** is hugely beneficial to the newcomers (_this cannot be overstated!_), as it:

- **Improves overall productivity** (because you get to exactly what's needed for the task at hand)
- **Reduces frustration** for the newcomer (because it takes a lot of time to acquire context from code, we've all been there 😥)
- **Reduces time and effort** on the newcomer side (because there's less code to look at)

# Conclusion

> "RTFM culture is in nobody's interest **ever** - it slows everything and everyone down."

> "Say no to RTFM; start today!"

To the one **giving** the context:

- Be **willing to share** the context freely! - Not everything needs to be a barter trade.
- Spend a few minutes to gather the requisite **pieces of knowledge** and **context** required for the task
  - Eg. related files, lines of code, repositories, wiki articles, other contacts with said context
- Share this information in a **terse** and **timely manner** to whoever that is asking for it

To the one **receiving** the context:

- Be **gracious in your interactions** when asking for help, regardless of whether you get it or not - You are asking for a favour, after all.
- **Do your homework**, and **formulate the problem description** clearly:
  - Eg. What's the **use case** or adaptation that you are seeking to achieve?
  - Eg. What are the **possible code changes** with this new feature?
  - Eg. What are the **possible architectural changes** with this new feature?
  - Eg. For debugging issues, it would be useful to provide **links and screenshots** of the said issue (eg. Kibana links to logs, Datadog alerts)
  - These steps go a long way in helping **the other party quickly understand** where you are coming from (and reducing their frustration too, because context-sharing is a two-way street! 😅)
- Don't forget to **return the favour**! - Return the help, and give credit and thanks where it is due.
