---
layout: post
title: Fun with Iota
categories: [golang]
tags: [golang]
fullview: false
excerpt_separator: <!--more-->
comments: true
---

A super short word on [Iota](https://github.com/golang/go/wiki/Iota#summary) with Golang.

Sometime back I was creating a UDP application that presented a basic JSON-API. This application was split in to multiple packages and I needed to use consistent JSON operation verb codes in each.

<!--more-->

Iota is used with constant declarations. It starts at zero when used and can populate other sequentially listed constants without any other expressions. It's useful and elegant hence I used it.

I fell into a trap that even Admiral Ackbar would have noticed a mile off.

Here's some simple code that shows the constants I created for the JSON-API.

{% highlight go %}

package main

import "fmt"

const (

	// Create constants using 'iota' and start from 1 (iota starts from zero)
	GET = iota + 1
	SET
	DEL
)

func main() {
	// Things check out as expected
	fmt.Println("GET: ", GET)
	fmt.Println("SET: ", SET)
	fmt.Println("DEL: ", DEL)

	// Output
	// -------
	// GET:  1
	// SET:  2
	// DEL:  3
	//
}


{% endhighlight %}

Output is as expected here. Great. The next task was to add another few constants for alarm levels and handling. For the sake of brevity in this example, I'll add just one.

{% highlight go %}

package main

import "fmt"

const (

	// Set alarm consts
	ALARM_LEVEL = 3

	// Create constants using 'iota' and start from 1 (iota starts from zero)
	GET = iota + 1
	SET
	DEL
)

func main() {
	// Things check out as expected
	fmt.Println("GET: ", GET)
	fmt.Println("SET: ", SET)
	fmt.Println("DEL: ", DEL)
	fmt.Println("ALARM_LEVEL: ", ALARM_LEVEL)

	// Output
	// -------
	// GET:  2
	// SET:  3
	// DEL:  4
	// ALARM_LEVEL:  3
	//
}

{% endhighlight %}

What's going on here? Why are my constants offset? Turns out, what I glanced over and didn't read properly was that iota is reset to zero every time the const keyword is used. A bit of an 'ahhh' moment. By inserting this new const I changed the absolute position of iota from the const keyword.

#### The Burn Down

So, I think two approaches exist for future Dave to take note of:

- Put my iota consts at the top of const declaration block

- Have two blocks of const.

I for one think the two block approach looks neater and no guess work for anyone else who hasn't took note of this small but important iota useage note.

Question to the audience: I haven't found an issue with linters moaning or complaining about two const blocks. Is it likely to be an issue? Is this considered idiomatic? It's certainly readable and removes any guess work.

Here's a visual of the two const block approach.

{% highlight go %}

package main

import "fmt"

// This block is just for the JSON-API
const (
	GET = iota + 1
	SET
	DEL
)

// This block is for other consts not relying on iota
const (
	ALARM_LEVEL = 3
)

func main() {
	// Things check out as expected
	fmt.Println("GET: ", GET)
	fmt.Println("SET: ", SET)
	fmt.Println("DEL: ", DEL)
	fmt.Println("ALARM_LEVEL: ", ALARM_LEVEL)

	// Output
	// -------
	// GET:  1
	// SET:  2
	// DEL:  3
	// ALARM_LEVEL:  3
	//
}


{% endhighlight %}

Thoughts please!
