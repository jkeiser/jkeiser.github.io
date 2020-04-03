---
title: "Simdjson 0.3: My Contribution to Reducing Global WArming"
description: "We just released simdjson 0.3, a ridiculously fast
JSON parser written in C++. And I do mean ridiculously fast: it's 2.5x faster than the fastest
parser out there!"
date: 2020-04-01
tags: ["simdjson", "c++"]
draft: true
---

# simdjson 0.3: My Contribution to Reducing Global Warming

We just released [simdjson 0.3](https://lemire.me/blog/2020/03/31/we-released-simdjson-0-3-the-fastest-json-parser-in-the-world-is-even-better/), a ridiculously fast
JSON parser written in C++. And I do mean ridiculously fast: it's 2.5x faster than the fastest
parser out there!

simdjson 0.3 improves a lot over 0.2:
* Drastically simplifies the API
* Improves performance by another 15%
* Provides simple, flexible error handling whether you use exceptions or error codes
* Includes batch parsing for multiple documents in one file/buffer (ndjson) at 2-4x the normal speed
* Parses floating-point numbers precisely, with no loss in performance
* Adds a quickstart, usage docs / tutorials, and API docs

The source has also been substantially refactored for maintainability and code reuse.

## Why It Matters

When I first saw this project a year ago, my jaw dropped to the floor. I still haven't managed to
pick it back up. JSON is decades-old, ridiculously simple and absolutely everywhere. 2x improvement
in parsing just *shouldn't be possible*. It shows we've been doing something fundamentally wrong.

And it makes a big difference. Servers spend a huge amount of time reading and writing JSON. At
Chef, over 50% of the total was spent on JSON! Cut that time in half and you not only serve users
faster, you need fewer computers to do it. Which means less power, less cooling, less of everything
that is fouling the world I'm leaving to my daughters and son.

## Learning and Growing

There were many contributors to this, it's not the work of one or even two people. That said, I'm
very proud of how much I've learned and was able to contribute. Before this project, my
understanding of low-level performance was just good enough to avoid being stupid. I'd never really
gone through and majorly fine-tuned performance like this, and didn't really know how.

But in the last year I've learned a ton from @lemire, @geofflangdale, @travisdowns, @ioioioio, @DBT, @zwegner and many others. With the exception of exact floating-point parsing and batch parsing, I
wrote most of the features above. With their pointers and guidance, I even designed what might be
the fastest UTF-8 validation algorithm in the world, which is astonishing given that I've always has
an aversion to low-level bit-twiddling :)

And there's still more to learn, so I'm not done.