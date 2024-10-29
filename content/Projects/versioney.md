---
title: versioney
date: 2020-06-01
tags: dev
---

I'm currently writing a SSG, like the majority of developers do at some point in their life.
Writing a SSG is fun and easy, and can be a good way of learning a new language, or implementing weird and quirky features that you thought about.

## Rationale
In my case, I found out that the way I update my personal site clashes with the classic chronological ordering of posts you usually see in a blog.

I have talked about that in an old post that I recently deleted, where I tried to pinpoint my vision of a suitable ordering.
The basic idea is to have both classic date of creation based ordering, along with a date of modification one, and to style recently edited parts of the document differently.

Now some time has passed since then, and I have been working on [versioney](https://versioney.gitlab.io).

versioney aims to be a spiritual successor to tibl.
tibl is/was a client-side blog thing that rendered markdown using client-side javascript libraries such as [Showdown](http://showdownjs.com/).
Client-side rendering has its perks: it makes posting new content very easy, since you just have to push or upload your content to your repository/server and that's it.
However, no JS means no rendering, and delegating the work to the client can cause UX problems since you rely solely on them to render things properly.

I chose to build versioney because implementing the aforementioned vision was tricky. You would need to rely on third parties to explore the online repository, or use other weird techniques.

## Concept
The way I see it, versioney needs to be, in that order:

1. A basic useable static site generator...
1. ...that supports displaying updates..
1. ...and orders posts by "last updated" rather than "post date".

I like the principle of [dogfooding](https://en.wikipedia.org/wiki/Eating_your_own_dog_food), so I spent some time building the backbone of the SSG (process the `src` directory, generate html files and bundle everything in a `dst` folder) so that I could use versioney right away and tune it to my needs.
Now that everything _kinda_ works, I have begun implementing basic versioning support, by being able to provide style changes for content that was recently updated.

## Getting the changes
Right now, I use `git blame` as the way to get updates from content:

For every file that is marked as a "versioned" one, I run a blame and see which changes have been made recently. These changes will have some html tags added before and after them, tags that have to be specified in the blog's config file.
Currently, the tags can contain a `{}`, that will be replaced by the _age_ of the modification. You can then style more than the latest change, bu specifying another paramter in the blog's config file.

The `git` section of the Vyfile from a site looks like that:

```toml
[git]
    # these two tokens will be added in the files where
    # the changes are detected
    start_token = '<div markdown="1" class="age-{}">{{date}}'
    end_token = '</div>'

    # memory sets how many changes we style
    memory = 3

    # these metadata are accessible to the templates under git.metadata.foo
    metadata = [
        "author",
        "committed_datetime",
        "message",
        "name_rev",
        "summary",
    ]
```

![example of a versioned section.](https://ujj.space/static/img/versioning-example.png)


The versioning code is quite messy for now, but works for me.
I would like to change the way updates are fetched, replacing relying on blame by relying on commit history.
I dislike the way that the tags have to contain the `{}` string, and I'll change that to `{age}`, for example.
This would enable versioney to embed something else than an integer representing the age of the change.
