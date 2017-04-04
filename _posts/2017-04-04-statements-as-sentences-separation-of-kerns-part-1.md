Statements as Sentences
=======================

This is part 1 of Separation of Kerns, a series on language design choices around spacing. Other parts:

* [Introduction]({% post_url 2017-04-04-separation-of-kerns-introduction %})
* [Part 1: Statements as Sentences]({% post_url 2017-04-04-statements-as-sentences-separation-of-kerns-part-1 %})
* Part 2: Newline Sequences
* Part 3: Bare Function Calls
* Part 4: Indented Blocks
* Part 5: Tight Expression Grouping

We'll start with what shouldn't be controversial: **whitespace is significant in all popular languages.** These languages have a single caveat, and minimize all other whitespace:

> In C++ and similar languages, the presence of whitespace is significant, but the type and amount is not.

By doing this, we treat statements like typical *sentences* in printed material, making the compromise match something people are familiar with. Program statements generally follows some form of sentence structure, where characters = letters, tokens = words, and statements = sentences. At minimum, popular languages allow spaces to separate identifiers, including C++:

    ```c++
    intx;  // Variable access.
    int x; // Variable declaration.
    ```

What's not significant in most languages is the *amount* of whitespace. If you've seen one space or tab, you've seen 'em all.

    ```c++
    int x;    // Variable declaration.
    int    x; // Same variable declaration.
    ```

Also like sentences in books, C++ and other languages allow statements to cross lines:

    ```c++
    int x;    // Variable declaration.
    int  
     x;       // Same variable declaration *again*.
    ```

Why do languages we allow space to work like this? Because languages are for computers *and* people to read. Our brains are massive pattern matching neural networks that are great at quickly judging spatial relationships. Computers, on the other hand, only understand space as "just another character/pixel," and are pretty bad at spatial recognition and ambiguity resolution (at least at doing it quickly).

The compromise some languages have found is to treat whitespace as a human thing, with an analogy to sentences, with the sole above exception.

Now that we've established the rules nearly everyone follows, we can move into whitespace choices that make languages *different*. All we've talked about so far is sentences. But whence paragraphs? Next time.