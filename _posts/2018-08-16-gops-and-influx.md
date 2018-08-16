---
layout: post
title: gops & InfluxDB
categories: [golang]
tags: [golang]
fullview: false
excerpt_separator: <!--more-->
comments: true
---

What can I say, I'm paranoid about reliability and things working how they were designed to.

Memory leaks, CPU spikes and race conditions upset me so I try my absolute best to know what my code is doing, when it should do it and how it should recover. In reaction to seeing increased memory usage as an example, I want to know what is going on and how to solve it.

<!--more-->

[![Everything Is AWESOME]({{ site.url }}/assets/media/gopsvideostatic.png)](https://www.youtube.com/watch?v=OCRr8qpnPlY "Gops & InfluxDB")

For a while, I knew of gops but didn't really do a lot with it. The same is true of pprof. This year I can say I'm as comfortable with both as the Go language itself thanks to building a couple of long lived data collection and transformation services.

This super short post is about a patch which modifies `gops` so that it can export JSON data from the embedded agent which lives in your application, to the `gops` client which then publishes the data to InfluxDB.

## Gops

Gops is a tool that attaches to your code via an embedded agent that interacts with the runtime giving you information on CPU and memory activity. It's neat, tidy and extremely easy to use. Below is a short list of capabilities taken from patched `gops` app.

```bash
    stack       		Prints the stack trace.
    gc          		Runs the garbage collector and blocks until successful.
    setgc	        	Sets the garbage collection target percentage.
    memstats    		Prints the allocation and garbage collection stats.
    memstatexport		Exports to InfluxDB.
    version     		Prints the Go version used to build the program.
    stats       		Prints the vital runtime stats.
```
In order to get the most out of `gops`, the client has to attach to your running program. If your program is short lived, then there probably isn't much point.

```go
import (
    "blah"
    "github.com/google/gops/agent"
    "blah"
)

func main() {
    //blah...
    if err := agent.Listen(agent.Options{}); err != nil {
		log.Fatal(err)
    }
    //blah...
}

```

Here's how to use the gops client to connect to the gops agent which is running in your code.

```bash
gops memstatexport <pid>|<addr> <influxdbhost:port> <dbname>
```

Where...

| Argument | Description |
| :-- | :-- |
|```<pid>\|<addr>```| PID of the instantiated go program, or the address where the programming is running.
|```<influxdbhost:port>```| The InfluxDB server IP address and port string|
|```<dbmame>``` | Is the database name for InfluxDB|
|||

Neat eh? I'm a big believer in re-use and instead of forking gops and doing my own thing with a webserver or something like, I chose to extend the client and agent in the lightest way possible so I could publish data to InfluxDB and then use Chronograf to view it. The mods didn't take much time (30 mins or so) thanks to the well written code and what's more, I solved my own issue over night. The real sad admittance is to falling asleep whilst watching the graphs on my laptop screen.

## Installation

Follow the instructions at [github.com/arsonistgopher/gops](https://github.com/arsonistgopher/gops)

Other pre-requisites include InfluxDB with unsigned 64 bit integer support and installing Chronograf and attaching it to the InfluxDB data source. I used Docker for Chronograf and build InfluxDB natively on my Mac. All instructions for running this are on the README in the repo.

## Agent Mods

The agent now exports memory statistics in JSON format in addition to natural language formats.

## Client mods

The client calls the RPC for exporting JSON serialized statistics and sends them on to InfluxDB.

## Close

For now I have just added JSON support for memory stats. If you create a diff in Git of the pre-patched vs patched version, it will become very obvious of how to extend this kind of pattern for other capabilities that you want in InfluxDB. The logic on using `gops` for this is because it exists, works well and was super simple to extend. The value-chain is being reused and through simple enhancements, a lot of additional value was extracted.