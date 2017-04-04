---
layout: post
title: 'Separation of Kerns Introduction: Significant Whitespace'
tags:
    - whitespace
    - language design
---

Separation of Kerns: Significant Whitespace
===========================================

This is the introduction to Separation of Kerns, a series on language design choices around empty space. Other parts:

* [Introduction]({% post_url 2017-04-04-separation-of-kerns-introduction %})
* [Part 1: Statements as Sentences]({% post_url 2017-04-04-statements-as-sentences-separation-of-kerns-part-1 %})
* Part 2: Newline Sequences
* Part 3: Bare Function Calls
* Part 4: Indented Blocks
* Part 5: Tight Expression Grouping

My ideal programming language has significant whitespace.

There, I said it.

And I'm already getting that shivery feeling in my boots. Whitespace in programming languages is the subject of massive holy wars. It triggers deep emotions and kneejerk reactions from all sides, in the way that only the most practically insignificant and intensely personal matters tend to do.

Some people see "whitespace significance" as a synonym for "Python's indentation rules," however, and I'd like to talk about other rules (some with the potential to be even more controversial, IMO, like tight expression grouping in Ruby). So:

> Significant whitespace in a programming language is any way that horizontal or vertical spacing changes the meaning of a program.

Why does this matter, anyway? It's just syntax. There are way more important issues in language design, like speed and memory and expressiveness.

Syntax is how humans and computers both express and understand a program. That matters: if computers and humans don't agree on a program's meaning, it will not do what the human intends. That's the definition of a bug.

I won't lie, I have personal preferences too :) I hate when a program is hard to scan, or I have to slow down to understand something not directly relevant to the code I want to understand or problem I want to solve.

There are five of these that I've chosen to look at. Each section will cover the different flavors of each one, the pros and cons I've encountered, and wot I think. They are in rough order of controversy:

* **Statements as sentences**: Using spaces and newlines to separate terms in a statement.
* **Newline sequences**: Allowing newlines to terminate statements instead of requiring `;`.
* **Bare function calls**: `sqrt 2` instead of `sqrt(2)`.
* **Indented blocks**: Nesting by indentation *a la* YAML, Python or Haskell.
* **Tight Expression Grouping**: Where `sqrt - 2` is different than `sqrt -2`.

I'd like to discuss it here to find out what I've missed: are there more issues I haven't thought about yet? Are there other innovative or interesting uses of whitespace worth looking at? It's all game.