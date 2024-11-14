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

To make this more concrete, let's imagine I have 10 sentences that I got from the internet, and that I want to remove crap and keep english sentences only:

```rs
let sentences = vec![
"Welcome to our website! Here you’ll find everything you need to know about fashion, lifestyle, health, and beauty... Click here to subscribe to our newsletter and never miss an update!",
"¡Descubre los mejores destinos de viaje para este verano! Los precios de los vuelos están sujetos a cambios, así que apúrate y reserva ahora. Más detalles en nuestra página principal.",
"Na pewno nie chcesz przegapić tych 10 najnowszych trendów technologicznych, które zmienią sposób, w jaki pracujesz i bawisz się w 2024 roku!",
"Don’t miss out on our amazing weekend sale – up to 50% off selected items in our electronics category! Offer valid only while supplies last. Terms and conditions apply.",
"Lorem ipsum dolor sit amet, consectetur adipiscing elit. Check out our extensive library of tutorials for programming, web design, and more!",
"Découvrez notre nouvelle gamme de produits bio ! Profitez d’une réduction de 10% avec le code ‘BIO2024’ – conditions générales s’appliquent.",
"Looking for a new pet? We have everything from kittens to exotic reptiles! Use our filters to find your perfect companion today!",
"Здравствуйте! Добро пожаловать на наш сайт. Здесь вы найдете все последние новости и обновления. Подпишитесь, чтобы не пропустить ничего важного.",
"Anmeldung abgeschlossen! Vielen Dank für Ihre Anfrage. Wir bearbeiten diese so schnell wie möglich, bitte prüfen Sie Ihren E-Mail-Posteingang für weitere Informationen.",
"Follow us on social media and stay connected! For terms and privacy information, please read our policies or contact customer support at support@ourcompany.com. Don’t forget to like, comment, and subscribe!",
"Welcome! 🍪 We use cookies for a better experience – Accept or Decline?",
"Latest News: 3 mins read | COVID-19 updates... learn more *>",
"<div class='content'>This offer expires soon – act now! <p>Terms apply</p>",
"¿Cómo llegar? - map coordinates: (34.0522° N, 118.2437° W) – 🚗 Directions",
"Unsubscribe here | © 2024 Company, Inc. All rights reserved.",
"Bienvenue sur notre site! CLIQUEZ ICI pour accepter tous les cookies et continuer",
"<a href='/products/sale'>SALE</a>: up to 70% off selected items! Only until Oct 15th.",
"404 Error: Page not found. <br> Go to <a href='/home'>Home</a>",
"Review score: 4.5/5 ★☆☆☆☆ - Highly recommended! (12345 reviews)",
"Your cart (3 items): subtotal $145.99 checkout > | <a href='/help'>Need help?</a>"];
```

Let's define some functions/structs to help us with that:

```rs
// exclusion reason 
#[derive(Debug)]
struct Reason(String);
```

We use a [Newtype](https://rust-unofficial.github.io/patterns/patterns/behavioural/newtype.html) pattern here.


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

Now, all of these filters share a common behaviour (and a common signature): filtering stuff, taking `&str` as input and returning `Option<Result>`.
So we can define a trait to group those behaviours and make adding a new filter easy:

```rs
trait Filter {
    fn filter(&self, item: &str) -> Option<Reason>;
}
```

Now, implementing the `Filter` trait for `LanguageFilter` is easy, as the `langid` method already has everything we need:

```rs
impl Filter for LanguageFilter {
    fn filter(&self, item: &str) -> Option<Reason> {
        self.langid(item)
    }
}
```

Now, how could we implement this for our simple static functions?

We could make them methods, and then implement the trait as we did for `LanguageFilter`:

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
Something that can be displayed implements `[Display](https://doc.rust-lang.org/std/fmt/trait.Display.html)`, so we can write:

```
trait Capitalize {
    fn capitalize(&self) -> 
}
```
```rs
trait Flt {
    fn flt(&self, item: &str) -> bool;
}

struct A;
impl Flt for A {
    fn flt(&self, item: &str) -> bool {
        true
    }
}

struct B;
impl Flt for B {
    fn flt(&self, item: &str) -> bool {
        false
    }
}

impl<T> Flt for T where T: Fn(&str) -> bool {
    fn flt(&self, item: &str) -> bool {
        self(item)
    }
}
impl Flt for Vec<Box<dyn Flt>> {
    fn flt(&self, item: &str) -> bool {
        self.iter().any(|f| !f.flt(item))
    }
}
fn main() {
    let filters: Vec<Box<dyn Flt>> = vec![
        Box::from(A{}),
        Box::from(B{}),
        Box::from(|x: &str| if x.len() > 10 {true} else {false})
    ];
    
    dbg!(filters.flt("hello"));
}
```