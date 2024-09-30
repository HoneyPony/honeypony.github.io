# Lessons Learned Building a Lox to C Transpiler

I've probably read [Crafting Interpreters](https://craftinginterpreters.com/) at least a half dozen times, but I had never actually worked through the book--just read it, appreciated the fantastic writing, and applied some of its techniques for parsing and lexing (and maybe a couple other ones) when relevant.

But I also have a desire to make a scripting language for [my custom game engine](https://github.com/HoneyPony/ponygame). Not for any particularly practical reason, but just because it seems fun, and because I think I can make something that is pleasant, at least for myself, to use.

However, jumping head-first into the deep end of compiler development has been, at best, a bit of a failure. You can see [my previous attempt](https://github.com/HoneyPony/ponyscript) fizzled out pretty quickly.

So over the summer I thought I should finally sit down and work through Crafting Interpreters, so that I at least had some experience building a complete programming language product. What started out as an interpreter ended up as a transpiler that outputs C, and I learned a lot along the way.

## Building the Interpreter

My secondary goal with working through Crafting Interpreters was to improve my Rust abilities, as I keep trying to write compilers in Rust (which is a hard way to learn it). I figured porting the code from Crafting Interpreters, which is already architected for me and working well, would help me learn Rust better than going in blind on my own project.

So the first thing I did was work through the first half of the book, writing an interpreter in Java, but in Rust. I decided to call this project "jaylox" as it is based on "jlox" but written in Rust (and, as we will see, eventually quite different).

Within a few weeks I had completed everything I wanted to from the first chapter! The main thing that was missing was garbage collection, which I think would be annoying at best to implement in Rust. Everything was instead simply reference-counted, and although this meant leaks were possible, oh well.

This is, perhaps, the first lesson learned:

## It's OK to have `Rc<RefCell<T>>` types

When writing Rust I previously have never wanted to write a type such as:

{% highlight rust %}
Rc<RefCell<Environment>>
{% endhighlight %}

I primarily program in C, and writing this when I should in theory be able to just use a raw pointer feels wrong. I mean, to mutably access the Environment requires me to `borrow()` the RefCell, which involves some additional arithmetic operations, and then the reference counting itself has overhead when I need to clone the Rc.

But... that's all OK. Sure, it could be more efficient. But it is still the most natural way (that I've seen, at least) to encode these semantics in Rust, and it really does payoff.

Once I started actually writing the transpiler (which outputs C) and the associated C runtime, I found I had many many memory related bugs in my C code--but I had absolutely none in my Rust code (besides the aformentioned reference cycles). The simple ability to write Rust code and not have any of these kinds of bugs is so incredibly refreshing, even if there is a performance overhead.

I'm still not totally satisfied with how object graphs work in Rust, as if I were to implement a GC for my interpreter I really don't know how I would do it ergonomically (i.e. without a bunch of "index" types like `u64` that are basically just fake raw pointers), but I am overall happier with Rust than I used to be.


