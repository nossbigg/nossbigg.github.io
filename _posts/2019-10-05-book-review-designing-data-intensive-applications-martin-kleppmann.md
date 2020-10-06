---
layout: post
title:  "Book Review: Designing Data-Intensive Applications by Martin Kleppmann"
date:   2019-10-05 15:00:00 +0800
tags: [book-reviews]
---
<center>
<img src="/assets/2019-10-05-book-review-designing-data-intensive-applications-martin-kleppmann/designing-data-intensive-applications-kleppmann-cover.jpg" height="400"/>
<br />
<i>Designing Data-Intensive Applications</i> by Martin Kleppmann (<a href="https://dataintensive.net/">link</a>)
</center>
<br />

# What I needed answers for:
> "_In a massively distributed and concurrent systems scenario, how can one use databases (SQL/NoSQL) in a performant manner and yet still preserve data integrity/correctness?_"

# Key points:
- Traditional SQL databases (eg. [PostgreSQL](https://www.postgresql.org/)) are ‘**bundled**’ (ie. [transactions](https://en.wikipedia.org/wiki/Database_transaction), [2PL](https://en.wikipedia.org/wiki/Two-phase_locking), [indexes](https://en.wikipedia.org/wiki/Database_index), [joins](https://en.wikipedia.org/wiki/Join_(SQL)), [foreign key constraints](https://en.wikipedia.org/wiki/Foreign_key), [MVCC](https://en.wikipedia.org/wiki/Multiversion_concurrency_control), etc)
- You can ‘**unbundle** the database’ in order to get more performance out of systems, but there are caveats
- ‘Unbundling the database’ means that you’ll have to build all the system guarantees that comes with ‘bundled’ databases by yourself
- General principles of building ‘unbundled’ systems
  - Use systems that rely on **write-ahead-log** ([WAL](https://en.wikipedia.org/wiki/Write-ahead_logging)) (eg. [Kafka](https://kafka.apache.org/)) for their durability and atomicity guarantees
  - Design system actions to be **idempotent** (ie. the side effect of an action is guaranteed to be executed only once, even if you repeat the execution of the given action) (eg. Consumer that sends out email should only send at most one email to the customer)
  - Increase system throughput by **increasing partitions**, instead of using coordination (eg. [2PC](https://en.wikipedia.org/wiki/Two-phase_commit_protocol)) or consensus-based (eg. [Paxos](https://en.wikipedia.org/wiki/Paxos_%28computer_science%29)) techniques
  - Determine the **constraints** of the system
    - Eg. Unique username required?
    - Eg. Same business event should only occur at most once per day?
  - Determine the required **correctness** of the system
    - Eg. Ok to sometimes drop events? (eg. analytics systems)
    - Eg. Ok to process events out-of-order?
    - Eg. Ok to have duplicate events?
  - Use **event IDs** to dedupe events
- Systems are composed of `state` and `application`, ie. database and code
- Systems can be **synchronous** (eg. request/response) or **asynchronous** (eg. subscribing to events)
- Start to think of systems in a **dataflow paradigm** (eg. when someone messages you, data flows asynchronously from the other person’s phone to the server, and then you receive the message via a socket)

# Why this book is awesome:
- Solid suggestions on how to make scalable data systems ✅ (especially Chapter 12, _Unbundling Databases_, and _Aiming For Correctness_🎖🙆‍♂️🤯)
- Effective presentation of pertinent concepts about distributed and concurrent systems 
- Plenty of cross-references (note: a _boatload_ of them!) for you to learn (even) more about data systems
  - Eg. [Pat Helland and David Campbell. Building on quicksand](https://arxiv.org/abs/0909.1788)
- Beautiful illustrations at the start of every chapter

# Tips on how to approach this book
- This book covers many topics, some of which may not be immediately useful to you at the moment (eg. for me, it was Chapter 4. _Encoding and Evolution_)
- Best way to let this book hold your attention is to start with the Index and the Summary
- Mark out what catches your eye and focus on those things

# Closing thoughts:
- I concur with Kevin Scott’s glowing endorsement of the book: 

> “This book should be required reading for software engineers…” - Kevin Scott
 
- As software engineers, the applications that we build principally consists of two things: `state` and `application code`. 
- This book is a great introduction on how to handle **data at rest** (eg. databases), **data at motion** (eg. event streams), and the general principles on how to do both at scale.
