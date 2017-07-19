---
layout: post
title: Golang magic, package level vars, init, Init and global state
categories: [golang]
tags: [golang]
fullview: false
excerpt_separator: <!--more-->
comments: true
---

#### Pre-Intro

This article is a re-focussed re-write on the matters at hand. This article was triggered by Peter Bourgon's posting of this [http://peter.bourgon.org/blog/2017/06/09/theory-of-modern-go.html](http://peter.bourgon.org/blog/2017/06/09/theory-of-modern-go.html). I hopped on the bandwagon pretty quick to explore the subject matter but failed in round one to come up with something 100% concrete, so dry docked the article to work on it. Welcome to version 2.0.

<!--more-->

#### Intro

Code that is created with magic, bad habits and no regard for anyone else will result in unreadable and unmaintainable code. At this point, you are writing code that reads like a spell and you begin to believe you are Gandalf the Grey, taking pride in how clever and powerful you are. However, anyone else who looks at your code will think it's time for your magic to be taken away.

Writing good code doesn’t mean you have to be Gandalf the White, but it does mean you have to be very careful about when and how you use magic. It means you must put integrity, readability and simplicity first. Write code that removes any doubt to what the code is doing. Peter described how global state is magic and casts spells on your code. This is what I want to discuss.

#### What does Peter mean by magic?

I am the first to admit to using package level vars and `init()` in some very constrained use cases like singletons. Bill Kennedy points out configuration and logging are delegates for singletons if a code base is small and there is no requirement to cleanup state on shutdown. But if a code base is growing and the team that works on it is growing, these singletons will become a pain point.

I will first talk about what Peter means by magic and how I've understood it. After all, arguments are mostly subjective as we don't share the same brain and it's one person's view being projected (albeit a knowledgeable, informed and influential person with a complex but valid view).

Here's the magic dictionary entry.

> magic
> ˈmadʒɪk/Submit
> noun
>
> The power of apparently influencing events by using mysterious or supernatural forces.
synonyms:   sorcery, witchcraft, wizardry, necromancy, enchantment, spell working, incantation, the supernatural, occultism, the occult, black > magic, the black arts, devilry, divination, malediction, voodoo, hoodoo, sympathetic magic, white magic, witching, witchery;

It’s funny at first glance how magic and code have similarities.

Ok, so what does Peter describe as magic? Here's an excerpt from the blog post listed above:

>    magic is bad; global state is magic → no package level vars; no func init

>    The single best property of Go is that it is basically non-magical. With very few exceptions, a straight-line reading of Go code leaves no ambiguity about definitions, dependency relationships, or runtime behaviour. This makes Go relatively easy to read, which in turn makes it relatively easy to maintain, which is the single highest virtue of industrial programming.

Peter says a number of things (paraphrased-ish):

*   No package level vars
*   No function init()
*   Non-magical, straight line reading code
*   Lack of ambiguity about relationships or dependencies
*   Runtime behaviour has to be clear
*   The above leads to maintainability
*   VERY FEW EXCEPTIONS < *note, the word exception*

#### The magic of package level vars and init()

In article 1.0, I originally said _"Not using `init()` means you have to think about how something is instantiated and whether it’s unique and if it should be globally unique"_. This doesn't change if you use package level vars or init() for singletons. As a package creator, you have to think very carefully about initialization but how does your choices affect the package consumer’s ability to understand how things are initialized and in what order?

When the code base is small or when there is no need to clean things up on shutdown, a singleton could be used without the loss of initialization readability. Look at this code which initializes the Go standard library default logger in the `main` package using an `init()` function.




```
package main

import "log"

// init is called before main. We are using init to customize logging output.
func init() {
    log.SetFlags(log.LstdFlags | log.Lmicroseconds | log.Lshortfile)
}

// main is the entry point for the application.
func main() {
```

The initialization of the log package is clear  and since the call to `init()` is happening in the main package, it’s not hidden from view. The log package doesn’t require a clean up call when the program is being shut down so the initialization and use of the default logger is clean. All of this said, let’s be clear, we are mutating global state and removing some runtime control capability instead of passing functions and methods a logger var/object, which provides us with more control to manipulate the logger var/object as it’s passed around. You could also ask “what’s the point of having an init()?”. Wyatt Johnson points out that some packages have a ‘before’ function, which is called before every action. This could set things like logging level and format.



Here is another example of how a singleton and the use of `init()` could be an exception.

```
import "database/sql"
import _ "github.com/go-sql-driver/mysql"
```

In the code above an import with the use of the blank identifier allows the compiler to find and execute the `init()` function declared inside the `mysql` package. The blank identifier is a good readability marker that an `init()` function is getting called. The `init()` function inside the `mysql` package is being used to register the driver to the `sql` package.

```
func init() {
    sql.Register("mysql", &MySQLDriver{})
}
```

Here is the code for the `sql.Register` function:

```
var (
    driversMu sync.RWMutex
    drivers   = make(map[string]driver.Driver)
)


// Register makes a database driver available by the provided name.
// If Register is called twice with the same name or if driver is nil,
// it panics.
func Register(name string, driver driver.Driver) {
    driversMu.Lock()
    defer driversMu.Unlock()
    if driver == nil {
        panic("sql: Register driver is nil")
    }
    if _, dup := drivers[name]; dup {
        panic("sql: Register called twice for driver " + name)
    }
    drivers[name] = driver
}
```

You can see the `sql` package uses a package level `map` that stores the actual driver implementation by name.

The initialization of the `sql` package is interesting because of the use of the blank identifier on the import and what follows. I would expect the call to `sql.Open` to happen in the main package and there are no calls that are required to be made from the `sql` package when the program is shutting down.

I think this example is interesting and very contentious. Is it magic? Yes. Is it a problem? I’m not sure. I think it’s quite neat the way the dependency is imported and it doesn’t affect anything else. However, for newcomers as pointed out by Kale, this could be really confusing and could be even rated as one of the worst anti-patterns displayed by the stdlib. Yay and d’oh.

#### Summary

With the two exceptions above, we can see what is being  performed within the packages. All of the package setup happens in the `main` package. This is an important point. If you have to do this, keep it in main.

My initial jump to a conclusion in v1.0 of the article was ill aimed but a good lesson. If in doubt, present reason as an argument and seek council from the community. You can also check out how other singletons are created as guidance.

Keeping magic out of code is a part of keeping code readable, maintainable and lacking ambiguity.

The article from Peter is at the very core of writing ~~good~~ great Golang code. It should be used as a base rule if you’re not already doing so. If you have to do something, do it from main so it’s immediately noticeable.



Whilst this article was under construction, Dave Cheney also wrote [this](https://dave.cheney.net/2017/06/11/go-without-package-scoped-variables) article on the subject which is as always, pretty darned good.

This has been a huge learning experience and I hope this has been an interesting read.

#### Special Thanks

[Bill Kennedy](https://twitter.com/goinggodotnet) has endured an enormous amount of questioning from me whilst building this 2.0 version. Thank you Bill for your input, reviews and keeping me honest.

[Kale Blankenship](https://twitter.com/vcabbage) reached out from version 1.0 and questioned some badly described thinking and has since gone on to peer review this article. Thank you Kale.

[Wyatt Johnson](https://twitter.com/wyattjoh), thank you too for stepping in with your great commentary.



