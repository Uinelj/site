---
title: Rust Traits
---

These days I write less Rust code than I used to, and one of the features that I miss the most are traits. 
Traits are ubiquitous in Rust, and are often described are "interfaces with superpowers". 

While I could write something about what a trait is and how to use it, I'd prefer linking the fantastic resource that is [The Rust Book](https://doc.rust-lang.org/book/)'s chapter on [traits](https://doc.rust-lang.org/book/ch10-02-traits.html).

Here I'd like to take a concrete example of where traits helped me. 

## Problem description:

I have an ordered set of filters that tell me if I should keep or discard a given item. 
Those filters are run in order, with usually faster filters up in the chain, and more heavy ones down.

In the following, we're dealing with sentences crawled from the internet, and we want clean, English-only sentences.

Let's define some functions/structs to help us with that:

```rs
#[derive(Debug)]
struct Reason(String);
```

We use a [Newtype](https://rust-unofficial.github.io/patterns/patterns/behavioural/newtype.html) pattern here.
Filters will return `Option<Reason>{:rs}`, which means `None{:rs}` if the sentences gets through, and `Some(reason)` if not, with `reason` telling us why the sentence got filtered out.


Some static filtering functions:
```rs

/// Excludes text that is too short
fn length_filter(text: &str) -> Option<Reason> {
    if text.len() > 50 {
        None
    } else {
        Some("too short".into())
    }
}


/// Excludes text that might be HTML
fn html(text: &str) -> Option<Reason> {
    if text.starts_with("<"){
        Some("is html".into())
    } else {
        None
    }
}


/// Excludes text that has >50% chars that are non alphabetic
fn noise(text: &str) -> Option<Reason> {
    let noise_thresh = text.len() / 2;
    if text.chars().filter(|c| !c.is_alphabetic()).count() > noise_thresh {
        Some("is noisy".into())
    } else {
        None
    }
}
```

And a model based language filter that needs some initialization.

```rs
struct LanguageFilter {
    // ...model
}

impl LanguageFilter {
    fn predict(&self, text: &str) -> (String, f32) {
        // ...logic to guess the language 
        ("en".into(), 0.9)
    }
    
    /// Excludes text that is not english with 0.9 confidence.
    pub fn langid(&self, text: &str) -> Option<Reason> {
        let (label, conf) = self.predict(text);
        if label != "en" || conf < 0.9 {
            Some("langid".into())
        } else {
            None
        }
    }
}
```

Ok! Now that we have some filtering functions we can test them and use them:

```rs
let text = "<h1>hello world!</h1>";
if let Some(reason) = html(text) {
    println!("text discarded: {:?}", reason);
} else {
    println!("text kept");
}
```

We'd get `text discarded: Reason("is html")` here, all good!
First step done. 

Now, all of these filters share a common behaviour (and a common signature): filtering stuff, taking `&str{:rs}` as input and returning `Option<Reason>{:rs}`.
So we can define a trait to group those behaviours and make adding a new filter easy:

```rs
trait Filter {
    fn filter(&self, item: &str) -> Option<Reason>;
}
```

Now, implementing the `Filter{:rs}` trait for `LanguageFilter{:rs}` is easy, as the `langid` method already has everything we need:

```rs
impl Filter for LanguageFilter {
    fn filter(&self, item: &str) -> Option<Reason> {
        self.langid(item)
    }
}
```

Now, how could we implement this for our simple static functions?

We could make them methods, and then implement the trait as we did for `LanguageFilter{:rs}`:

```rs

struct LengthFilter{}

impl Filter for LengthFilter {
    fn filter(&self, text: &str) -> Option<Reason>
        if text.len() > 50 {
        None
    } else {
        Some("too short".into())
    }
}
```

This would imply creating a new empty struct for each new filter, or grouping them into a single struct.

But there is a better way. Traits in Rust are everywhere, and are quite flexible. Basically:
- You can implement your trait on foreign types (be it stdlib or types from other crates)
- You can implement foreign traits on your types (samesies)
- You (kinda) cannot implement foreign traits on foreign types (this is known as the [orphan rule](https://doc.rust-lang.org/book/ch10-02-traits.html#implementing-a-trait-on-a-type))


We can also define traits on a generic type `T`, and have trait constraints on `T`.
As an example, let's imagine we'd like to add a `capitalize` method to everything that can be displayed.
Something that can be displayed implements [`Display`](https://doc.rust-lang.org/std/fmt/trait.Display.html), so we can write:

```rs
trait Capitalize {
    fn capitalize(&self) -> String;
}

impl<T: Display> Capitalize for T {
    fn capitalize(&self) -> String {

        // .to_string() is provided by the Display trait
        let s = self.to_string();

        s.chars().map(|c| c.to_uppercase().to_string()).collect()
    }
}
```

Here, `T: Display{:rs}` can be read as *Any type, provided it implements `Display`*.


With that now in mind, we need another piece of information: the [`Fn` trait(s)](https://doc.rust-lang.org/book/ch13-01-closures.html#moving-captured-values-out-of-closures-and-the-fn-traits).

What is interesting for us here is that functions can be passed where generic types implementing `Fn` could pass.

```rs
// this function
// can be used where T: Fn(&str) -> Option<Reason> is bound.
fn html(text: &str) -> Option<Reason> {
    if text.starts_with("<"){
        Some("is html".into())
    } else {
        None
    }
}
```

To convince ourselves of that:

```rs
trait Greeter {
    fn greet(&self) -> String;
}

impl<T> Greeter for T
where
    T: Fn(&str) -> Option<Reason>,
{
    fn greet(&self) -> String {
        "hello from filtering functions 👍".into()
    }
}
```

And then, we can call `html.greet(){:rs}` 🦀! 

Implementing `Filter{:rs}` on `Fn(&str) -> Option<Reason>{:rs}` is then relatively simple:

```rs
impl<T> Filter for T
where
    T: Fn(&str) -> Option<Reason>,
{
    fn filter(&self, item: &str) -> Option<Reason> {
        self(item) // calls the function on item
    }
}
```

Then we get an equivalence between `func(item){:rs}` and `func.filter(item){:rs}`.
I wonder if this gets optimized away? Perhaps with an `#[inline]{:rs}` before.

## So what?

With all of that in mind, we can then implement our `Filter` trait on a wide array of different things, and use them interchangeably!

As an example, we can now have a `Vec` containing all of our filters: 

```rs
    let filters: Vec<Box<dyn Filter>> = vec![
        Box::from(length_filter),
        Box::from(noise),
        Box::from(html),
        Box::from(LanguageFilter {model: ()}),

        // we can even put a closure there, provided it meets the Fn trait!
        Box::from(|x: &str| if x.len() > 10 {None} else {Some(Reason("Too short!".into()))})
    ];
```

As a last step, we can then _also_ implement `Filter` on a collection of filters:

```rs
impl Filter for &[Box<dyn Filter>] {
    fn flt(&self, item: &str) -> Option<Reason> {
        self.iter()
            .map(|flt| flt.flt(item))
            .find(|res| res.is_some())
            .flatten()
    }
}
```


Then, running `filters.as_slice().filter(item){:rs}` would call all of our filters sequentially!

Next up: Collecting all filter results rather than only the first one, and doing weird (and possibly bad) things with `iter::once`.