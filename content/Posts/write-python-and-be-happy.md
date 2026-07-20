---
title: Writing Python without losing your mind
tags: [coding, python, human-authored]
---

I've been writing Python for a long time now. I think it's the language I wrote most during my career. 
A lot of things have changed since then (esp. me meeting a crab that I like very much), both in my personal coding adventure
and in the Python language/ecosystem. 

Here are some resources/things that I use to make the whole Python writing experience better. 

1. Stop even relying on your system's Python interpreter.

Just use `uv` and let it do its thing. You'll be happy to never have to check again why python 3.11.x is not in your system's packages or whatever. It makes working on any python version quite easier. And you get the added benefit of having something fast :) 

2. Type all the things

Right now there's a lot of ways you can type stuff _while_ keeping the flexibility of python. Anonymous sum types (like `str | None`), generics (`def foo[T](bar: str) -> Baz`), constraints on generics (`def foo[T: str](bar...)`).

Another one that is less known but can still be interesting, esp. when adding typing on methods you don't really own is `Protocol`.
This is more or less typed duck typing, as it enables you to specify what methods you're expecting on the input of a function.

Here's an example:

```py
from typing import Protocol


class Greeter(Protocol):
    def greet(self, name: str) -> str: ...


def say_hello(g: Greeter) -> None:
    print(g.greet("Alice"))


class Friendly:
    def greet(self, name: str) -> str:
        return f"Hello, {name}!"


class Rude:
    def greet(self, name: str) -> str:
        return f"Go away, {name}."


say_hello(Friendly())
say_hello(Rude())      
```

As you can see here, `Friendly` and `Rude` are acceptable as inputs for `say_hello`, while being implicitly compatible by conforming to the specified `Protocol`.

3. Use `pydantic`

Using pydantic's BaseModels is like using supercharged dataclasses: validation, serialization, deserialization, it does it all and makes your code way more robust by failing as early as possible (at data creation).

4. Some blog posts I love

There's two blog posts that helped me find out all those less known (and more recent) things you can find on modern Python:

- [Writing Python Like It's Rust](https://kobzol.github.io/rust/python/2023/05/20/writing-python-like-its-rust.html)
- [14 Advanced Python Features](https://blog.edward-li.com/tech/advanced-python-features/)