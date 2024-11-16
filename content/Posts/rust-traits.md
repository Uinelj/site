---
title: Rust traits and functions
---

These days I write less Rust code than I used to, and one of the features that I miss the most are traits. 
Traits are ubiquitous in Rust, and are often described as "interfaces with superpowers". 

While I could write something about what a trait is and how to use it,
I'd prefer linking the fantastic resource that is [The Rust Book](https://doc.rust-lang.org/book/)'s chapter on
[traits](https://doc.rust-lang.org/book/ch10-02-traits.html),
and show you an interesting, albeit relatively simplistic consequence of their flexibility.

This post assumes some level of familiarity with Rust, but I tried to add explanations for non trivial stuff.

## A little bit of context

I have a list of filters that tell me if I should keep or discard a given item. 
Those filters are run in order, with usually faster filters up in the chain, and more heavy ones down the chain.

In the following example, we're dealing with sentences crawled from the internet, and we want clean, English-only sentences.

Let's define some functions/structs to help us with that:

```rs
#[derive(Debug)]
struct Reason(String);
```

Filters will return `Option<Reason>{:rs}`, which means `None{:rs}` if the sentences gets through, and `Some(reason){:rs}` if not, with `reason{:rs}` telling us why the sentence got filtered out.

> [!info]- `Option<T>` in Rust
> 
>`Option<T>{:rs}` is this enum:
>
> ```rs
> enum Option<T> {
>    Some(T),
>    None
>}
> ```
>
> It is widely used to tell that a value might not exist. 
> Imagine a `list.first(){:rs}` method. What would it return if the list was empty?
>
> Rather than either returning a null value or raising an exception, `first(){:rs}` would return `None{:rs}` here.


Some static filtering functions:
```rs

/// Excludes text that is too short
fn length_filter(text: &str) -> Option<Reason> {
    if text.len() > 50 {
        None 
    } else {

        // .into() converts &str to String
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
    if text.chars().filter(|c| c.is_alphabetic()).count() < noise_thresh {
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
            Some("not english".into())
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

## Defining our `Filter` trait

Now, all of these filters share a common behaviour (and a common signature): filtering stuff, taking `&str{:rs}` as input and returning `Option<Reason>{:rs}`.
So we can define a trait to express that shared behaviour:

```rs
trait Filter {
    fn filter(&self, item: &str) -> Option<Reason>;
}
```

Now, implementing the `Filter{:rs}` trait for `LanguageFilter{:rs}` is easy, as the `langid` method already has everything we need:

```rs

// note that you can have multiple impl blocks for your struct,
// and implementing a trait is done on another impl block aswell :)
impl Filter for LanguageFilter {
    fn filter(&self, item: &str) -> Option<Reason> {
        self.langid(item)
    }
}
```

Now, how could we implement this for our simple static functions?
Traits can be implemented on a lot of stuff: structs, primitive types, references, tuples..
But not on functions. Or not directly:

```rs
fn foo() {}

impl Filter for foo {} 
```

```rs
error[E0573]: expected type, found function `foo`
  --> src/main.rs:77:17
   |
77 | impl Filter for foo {}
   |                 ^^^ not a type
```

One workaround we could use is to wrap those functions into a struct, and then implement the trait as we did for `LanguageFilter{:rs}`:

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
But there is a better way. Traits in Rust are everywhere, and are quite flexible.

## Traits rules

Basically:
- You can implement your trait on foreign types[^1]: `impl Thingy for &str`
- You can implement foreign traits on your types (samesies): `impl Display for YourStruct`
- You (kinda[^2]) cannot implement foreign traits on foreign types (this is known as the [orphan rule](https://doc.rust-lang.org/book/ch10-02-traits.html#implementing-a-trait-on-a-type)) There's a good reason for that: Without the rule, two crates could implement the same trait for the same type, and Rust wouldn’t know which implementation to use.[^3]

[^1]: types defined outside of the crate.
[^2]: Nothing stops you from wrapping the type into a [Newtype] and implementing the foreign trait on it: `struct MyType(ForeignType){:rs}`
[^3]: Sentence is copied from the aforementioned link. It's a good explanation dontcha think?

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


Now, with that in mind, we need another piece of information: the [`Fn` traits](https://doc.rust-lang.org/book/ch13-01-closures.html#moving-captured-values-out-of-closures-and-the-fn-traits).


`Fn` traits are automatically implemented for functions. A function that has the signature `fn foo(bar: &str) -> i32{:rs}` has a type associated that implements the `Fn(&str) -> i32{:rs}` trait.

This has an interesting consequence: Where we have a generic type `T{:rs}` we can restrict it to functions with a given signature:

```rs
// we can put functions that take a &str and returns a i32 in here!
struct FunctionHolder<T>
where
    T: Fn(&str) -> i32, // this is where we add the constraint on T.
                        // this is called a trait bound
{
    function: T,
}
```

Our filtering functions implement the `Fn(&str) -> Option<Reason>{:rs}` trait:

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

And then, we can call `html.greet(){:rs}`! It's completely useless though.

What's less useless now is that we can implement `Filter{:rs}` on our set of filtering functions!


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


## So what?

With all of that in mind, we can then implement our `Filter` trait on a wide array of different things, and use them interchangeably!

As an example, we can now have a `Vec` containing all of our filters: 

```rs

    // We use Box here because we're actually storing trait objects.
    // It's not that important here. I mean it's an important topic 
    // but for another time maybe!
    let filters: Vec<Box<dyn Filter>> = vec![
        Box::from(length_filter),
        Box::from(noise),
        Box::from(html),
        Box::from(LanguageFilter {model: ()}),

        // we can even put a closure that takes &str and returns Option<Reason>!
        // This is due to the "automatic" implementation of the trait on a whole set of functions,
        Box::from(|x: &str| if x.len() > 10 {None} else {Some(Reason("Too short!".into()))})
    ];
```

As a last step, we can then _also_ implement `Filter` on a collection of filters:

```rs
impl Filter for Vec<Box<dyn Filter>> {
    fn filter(&self, item: &str) -> Option<Reason> {
        self.iter()                      // get an iterator over our filters
            .map(|flt| flt.filter(item)) // map it to an iterator of Option<Reason>
            .find(|res| res.is_some())   // short-circuit on the first non-None result
            .flatten()                   // Since find returns Option<Option<Result>> we 
                                         //remove one level of indirection here.
    }
}
```


>[!note]- Why do we need `Box` here?
>
>`Vec` accepts a unique generic type `T`, which means you can't have a `Vec` that contains values of different types this way.
>To circumvent this issue we can rely on [Trait objects](). When you use trait bounds, Rust will guess the concrete types you're using and 
>create appropriate non generic implementations for those concrete types. Trait objects will _not_ do the same and will do the resolution
>at runtime. While this might have some overhead at runtime it's usually negligible.
>
>Now, if we try to use `Vec<dyn Filter>`, Rust won't be happy:
>
>```rs
>error[E0277]: the size for values of type `dyn Filter` cannot be known at compilation time
>   --> src/main.rs:96:36
>    |
>96  |     let filters: Vec<dyn Filter> = vec![];
>    |                                    ^^^^^^ doesn't have a size known at compile-time
>    |
>    = help: the trait `Sized` is not implemented for `dyn Filter`
>note: required by an implicit `Sized` bound in `Vec`
>```
>
>TODO: explain why we need to know size at compile time. See https://users.rust-lang.org/t/why-does-rust-need-to-know-the-size-of-types-at-compile-time/67356/2
>
>To fix this, let's wrap the `dyn Filter{:rs}` into `Box{:rs}`, which is a fat pointer. 
>This will get us a fixed size item (since it's a pointer + some metadata) that points to a dynamic memory location stored on the heap.


Then, running `filters.filter(item){:rs}` would call all of our filters sequentially!

Next up:
- Explain trait and trait types != generic stuff
- implementing annotate for `T: Iterator<Item=Option<Reason>>`.
- avoiding Box by manually building iterators of filters.
    - is it a good idea? 
    - profile vs. static Vec
