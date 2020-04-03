---
layout: post
title: 'Words, Words, Words: DSL in Ruby'
tags:
- chef
- ruby
- chef dsl deep dive
date: 2015-05-04
draft: true
---

You have a program, and you want your users to be able to read and write it easily. You need the power, portability and flexibility, but still has to be easy to read ... you need

A good Ruby DSL lets you drop a lot of the words and punctuation that normally get in the way of reading and writing a DSL, without having to leave the power and flexibility of a general-purpose language that can do anything.  But there are a fair few tricks to using Ruby to *make* such a DSL. I'll try to keep to a minimum of required Ruby knowledge, and explain why the DSL is the way it is, and how it's done.

Chef has a relatively good (and common) example of a DSL, in its *recipe DSL*:

```ruby
## Make a file with Hello World in it
file '/Users/jkeiser/x.txt' do
  content 'Hello World'
end
## create a machine and deploy my web application using a "recipe"
machine 'mario' do
  recipe 'mywebapp'
end
## install package 'apache'
package 'apache' do
  version '~> 2'
end
```

Since that's the one I know best, that's where I'll focus first; but the techniques here are nearly universal to DSLs.

<div id="toc"></div>

## Ruby DSL basics: defining words using methods

Chef's DSL is in Ruby, and Chef is too.  In Ruby, all DSLs are actually *method calls* on an object. So let's first look at how you normally do it in Ruby.  In Ruby, a DSL is a way to type words that look right to you, and have them call *methods* that do what you want.  Say we want to define a DSL that can recycle files by moving them into a `trash` directory:

```ruby
recycle '/Users/jkeiser/files'
undelete '/Users/jkeiser/files'
```

To do this, we define a couple of functions:

```ruby
require 'fileutils'
def recycle(directory)
  FileUtils.mv(directory, File.join('/Users/jkeiser/trash', directory))
end
def undelete(directory)
  FileUtils.mv(File.join('/Users/jkeiser/trash', directory), directory)
end
```

When you type `recycle 'filename'`, Ruby calls the `recycle` method with that directory as an argument.  That's all she wrote.

### Parameters: making a file

Let's put that together and build a basic `file` DSL:

```ruby
def file(path, content)
  puts "Writing #{content} to #{path}!"
  IO.write(path, content)
end

file '/Users/jkeiser/x.txt', 'Hello World'
```

It works, but it ain't pretty.  What the heck is Hello World for?

### Property Names: using hash arguments

Now, this is a little bit ugly ... we'd like to have a name for `content`.

```ruby
def file(path, content: "")
  puts "Writing #{content} to #{path}!"
  IO.write(path, content)
end

file '/Users/jkeiser/x.txt',
  content: 'Hello World'
```

What we did here was add `content: ""`, a Ruby *named parameter* that tells Ruby "when the user says `content: "Hello World"`, set `content` to "Helo World". If they don't pass it to me, set `content` to ""." This is a fantastically versatile mechanism, and you can even pass multiple parameters this way.

### Why it's not ideal

What if you wanted to say "I would like this directory to have these four files in it"?

With what we have so far, we'd probably write something like this:

```ruby
def directory(path)
  Dir.mkdir(path)
end

def file(path, content: "")
  puts "Writing #{content} to #{path}!"
  IO.write(path, content)
end

## Using Ruby methods like above
directory '/Users/jkeiser/nash'
file '/Users/jkeiser/nash/a.txt',
  content: 'I think that I shall never see'
file '/Users/jkeiser/nash/b.txt',
  content: 'A billboard lovely as a tree'
file '/Users/jkeiser/nash/c.txt',
  content: 'Indeed, unless the billboards fall'
file '/Users/jkeiser/nash/d.txt',
  content: 'I\'ll never see a tree at all'
```

It's pretty clear, but it feels like the computer is not helping us as much as it could. I'm repeating myself a lot.

## Contextual DSL

The real power of DSL comes when you give methods *context* using objects and methods. Methods alone can't do the job. In the last section we made a less than satisfactory DSL to create directories and files, that made us repeat a bunch of stuff, over and over. Let's see how we might do it better with objects.

### Contextual DSL with ruby objects

What if the `directory` just remembered the path I told it, so the files could use it and I could save myself a little eyestrain? We just need a sort of "local workspace" to dump information. In Ruby, that is an *object*, whose DSL is defined with a *class*.

A class is defined with the `class` keyword. Classes have methods, including DSL, defined with `def <dsl word>(<parameters>)`. You can call the `new` method on the class to create a new object instance, which is a sort of workspace where the DSL can stash and recall information. You can have multiple such objects, even with the same class, and they all get to have their own workspace.

Here's how we can use it for our DSL:

```ruby
class Directory
  attr_accessor :path

  def file(file_path, content: "")
    # If the path is relative (like "a.txt"), append it to the directory path.
    file_path = File.expand_path(file_path, path)
    puts "Writing #{content} to #{file_path}!"
    IO.write(file_path, content)
  end
end

d = Directory.new
d.path = "/Users/jkeiser/nash"
d.file "a.txt",
  content: "I think that I shall never see"
d.file "b.txt",
  content: "A billboard lovely as a tree"
d.file "c.txt",
  content: "Indeed, unless the billboards fall"
d.file "d.txt",
  content: "I\'ll never see a tree at all"
```

The point of all this ceremony was the first line of `file`, where we access `path` *from the parent directory*. Things to note here:

1. When you typed `d = Directory.new`, you created a new object with the DSL containing `path` and `file`.  It has a
2. When you typed `d.path = "/Users/jkeiser/nash"`, you stashed that string away in `d`.
3. `d.file` is the DSL word: it means "run the `def file` code, but inside the object `d`."  That way, `def file` can access `d.path`.

This is super functional.  You can create multiple objects, and they will each have a different recipe.  But, it's still not *pretty*.  I'm still repeating stuff.

### Clean DSL with instance_eval

Probably the best thing to do is get rid of all those `d`s.  I shouldn't have to keep repeating them over and over again. Ruby helps us here again, with `instance_eval`:

```ruby
d = Directory.new
d.path = "/Users/jkeiser/nash"
d.instance_eval do
  file "a.txt",
    content: "I think that I shall never see"
  file "b.txt",
    content: "A billboard lovely as a tree"
  file "c.txt",
    content: "Indeed, unless the billboards fall"
  file "d.txt",
    content: "I\'ll never see a tree at all"
end
```

The code between `do` and `end` is a *block*.  It's Ruby like any other code; when you type a word like `file` it calls a method, just like normal.

What `instance_eval` does is make it so instead of *global* methods like our first examples, now it's calling methods inside our new `Directory`, `d`.

For those interested, Ruby is actually doing `self.file` under the covers every time you just type `file`.  `instance_eval` just sets `self` to `d` before running your block.

### Wrapping it up in a block parameter

Now we've made the inside of the DSL nice.  Let's make the outside nice too!

```ruby
def directory(path, &block)
  d = Directory.new
  d.path = path
  d.instance_eval(&block)
end

directory '/Users/jkeiser/nash' do
  file "a.txt",
    content: "I think that I shall never see"
  file "b.txt",
    content: "A billboard lovely as a tree"
  file "c.txt",
    content: "Indeed, unless the billboards fall"
  file "d.txt",
    content: "I\'ll never see a tree at all"
end
```

Now this is looking much more readable.

### A final step

If you really want to make it just like Chef, you need `content` to be a DSL
word as well.  This has some real benefits--particularly the fact that you can write methods that do more than just store stuff (like `file`).

All we have to do to get the benefit for `File` is do the same thing we did for `Directory`:

```ruby
class Directory
  attr_accessor :path

  # NEW
  class File
    attr_accessor :path

    # Calling content "Hello World" sets @content.
    # Calling "content" with no arguments gets @content.
    def content(text=nil)
      if text
        @content = text
      else
        @content
      end
    end
  end

  # NEW
  def file(path, &block)
    f = File.new
    f.path = File.expand_path(file_path, path)
    f.instance_eval(&block)
    puts "Writing #{f.content} to #{f.path}!"
    IO.write(f.path, f.content)
  end
end

def directory(path, &block)
  d = Directory.new
  d.path = path
  d.instance_eval(&block)
  Dir.mkdir(d.path)
end

directory "/Users/jkeiser/nash" do
  file "a.txt" do
    content "I think that I shall never see"
  end
  file "b.txt" do
    content "A billboard lovely as a tree"
  end
  file "c.txt" do
    content "Indeed, unless the billboards fall"
  end
  file "d.txt" do
    content "I\'ll never see a tree at all"
  end
end
```

The big new thing we did here was the `content` method, which is accessing
`@content`. An `@variable` is always stored in the current object (so that when you say `@content = "Hello World"` inside `f`, and later ask for `@content`, `"Hello World"` comes back.  The `if` statement inside of

Note: `attr_accessor :blah` just creates methods that get and set `@blah`.  We had to create our own accessor because we want the syntax `content "Hello World"` to work, and `attr_accessor` only supports `content = "Hello World"` with an `=` sign.

## That's a wrapper

Clearly, you have to do some work to get all those words to act the way you want them to :) Next time, we'll explore how Chef works to make that easy, so that the process of *making* a resource DSL is as easy as *using* it.

{% include chef-dsl-deep-dive-toc.md %}
