---
layout: post
title: Reflections on 2 Years of Golang
description: Reflections from using Go in production for 2 years at work
date: 2020-04-12
image: /static/images/gopher.png
---

Click [here](https://www.notion.so/wgyn/Reflections-on-2-Years-of-Go-dc8c35b2c64741b08be4505a71def382){:target="_blank"} to read from Notion.

* TOC
{:toc}

I've been working in Go for the past two years at Assembled ([https://www.assembled.com](https://www.assembled.com)), where it's been our primary backend language since the inception of the company. In my prior job, I worked in a mix of Ruby and Scala, and there was definitely an initial adjustment period. Overall, I've found that Go behaves essentially as-advertised: it's well-suited for professional work though annoying in some of its quirks.

## Some context

Assembled is a web application used by customer support teams to manage staffing and analytics. From an architecture perspective, it's a standard web application—a React frontend accesses a Go backend via an internal HTTP API, which in turn accesses a PostgreSQL database.

That said, there are a few specific applications that we've also had to optimize for:

- We maintain an external [API](https://docs.assembled.com)
- We generate time series forecasts from a Python-based machine learning service
- We perform fairly heavy batch processing, as we sync schedules from call center systems via SFTP
- We run fairly sophisticated optimization algorithms; my brother John shared more on this in an [HN comment](https://news.ycombinator.com/item?id=22584243)
- We deal with painful logic involving timezones and recurring events

## Go in production

We deliberately decided to build Assembled in Go over Python, Java, and Haskell (yeah...). The deciding factors came down to: static typing, simplicity, the standard library, and speed. In practice, everything I've seen maps surprisingly well to how Go presents itself in the [docs](https://golang.org/doc):

> Go is **expressive, concise, clean, and efficient**... Go **compiles quickly** to machine code yet has the convenience of garbage collection and the power of run-time reflection. It's a **fast, statically typed, compiled** language that feels like a dynamically typed, interpreted language



### Easy, albeit tedious, for new engineers

Go's simplicity has allowed new engineers to quickly spin up on our codebase, many of whom had never used the language before (this includes myself). I suspect Go's design has forced us to write simpler and more explicit code than we otherwise would have. That said, the simplicity does lead to repetitiveness, as has been well documented. The snippet `if err != nil` appears 2,919 times in our codebase as of writing.

### The standard library does what's needed

Go's standard library is a shining beacon. Packages like `bytes`, `encoding/json`, `database/sql`, `net/http`, and `time` are comprehensive and well-designed. In Python, for example, it's telling that third-party libraries (to be clear, good ones!) are the de-facto defaults for HTTP.

Go was clearly designed with production code in mind. Most strikingly, Assembled had a period of serious performance issues (knock on wood) that we were able to debug with `net/pprof` and `runtime/pprof`. These were super powerful and easy to enable via HTTP handlers, as below. My one nit would be that the best guide I found to interpret the output was buried in a [blog post](https://blog.golang.org/pprof).

``` Go
  superAdminMux.HandleFunc("/debug/pprof/heap", pprof.Handler("heap").ServeHTTP)
  superAdminMux.HandleFunc("/debug/pprof/profile", pprof.Profile)
```

### Fast, easy builds affect everything

The best part of Go is that you can easily run `go build` and reliably expect a working executable with very little wait. Java still makes compilation painful without IntelliJ or Eclipse, and let's not even start with Ruby or Python.

Fast, easy builds experience have made a number of downstream tasks easy: 

- Our deploy command is essentially `git pull` followed by `go install`
- Continuous integration (CI), well... let's just say Go is not the problem

The main difficulty has been local development with a file watch and rebuild loop. We have multiple build targets (e.g. application backend and API) and use [https://github.com/cespare/reflex](https://github.com/cespare/reflex), which required some work to [play nice](https://github.com/cespare/reflex/issues/6) with Mac OS X.

### Standard formatting and documentation

Have I mentioned yet that Go was clearly built for professionals?

- `gofmt` handles indentation and alignment; I use the [vim-go](https://github.com/fatih/vim-go) plugin, which automatically applies it when you save a .go file
- Here's an [eloquent explanation](https://blog.golang.org/godoc) of how Go documentation is different; I most enjoy the standard look, the public repository at [https://godoc.org/](https://godoc.org/), and the fact that it runs locally and extracts project-specific code (quickly)

### Gopher ❤️

So cute in [all](https://raw.githubusercontent.com/golang/dep/master/docs/assets/DigbyShadows.png) [its](https://res.cloudinary.com/bizzaboprod/image/upload/c_crop,g_custom,f_auto/v1580236921/qt1k2jk58ragj3ox4gwt.png) [incarnations](https://github.com/fatih/vim-go/blob/master/assets/vim-go.png?raw=true). Here's a fun read on the Go gopher's origins: [https://blog.golang.org/gopher](https://blog.golang.org/gopher).

![The Go Gopher]({{ site.url }}/static/images/gopher.png)

## Pain points

Go is not without its annoying quirks. Many of these have already been well documented elsewhere, but I include them just for the sake of completeness and to vent a little.

### No official package manager story (until recently)

As of Go 1.14 (Feb 2020), Go modules have been anointed ready for production use. Before then it was a wild west—we landed on `dep` but haven't had a chance to migrate to modules. `dep` is/was an admirable effort but it's also very slow. A common suggestion is to check dependencies into your repository (in e.g. a `/vendor` folder), which is perhaps not crazy in a production setting.

### GOPATH is confusing

The `GOPATH` directory is supposed to magically contain all code. I think (speculation based on the [wiki](https://github.com/golang/go/wiki/GOPATH)) it had something to do with making it easy to fetch from remote repositories e.g. `go get github.com/my/repo`. That's elegant in theory but really confusing in practice, because if you don't put your code in the right place, nothing works. This left me with a really negative first impression of Go.

Now I just have the below in my `.profile` on my work machine:

~~~ Shell
  export GOPATH=$HOME/go
  export PATH="$GOPATH/bin:$PATH"

  cd $GOPATH/src/github.com/assembledhq/assembled
~~~

### Errors are hard to introspect

Most people hammer Go on the verbosity of error handling, but it's also difficult to work with. In Go 1.13 (Oct. 2019), [great methods](https://blog.golang.org/go1.13-errors) for wrapping, unwrapping, and comparing were added, but we're unfortunately behind the curve on adoption.

There's also a specific pain in working with pre-1.13 code that doesn't wrap errors. For example, in Google's [own API bindings](https://github.com/googleapis/google-api-go-client/blob/52f0532eadbcc6f6b82d6f5edf66e610d10bfde6/internal/gensupport/send.go#L35), the underlying HTTP error for a request does not wrapped and is thus not inspectable as a `googleapi.RetrieveError`, the public error interface, or even the low-level `url.Error`. The only option is to string match, which we do to catch like `invalid_grant` for an OAuth error.

Compare, for example, to how [error handling](https://docs.scala-lang.org/overviews/scala-book/functional-error-handling.html) follows from [pattern matching](https://docs.scala-lang.org/tour/pattern-matching.html) in Scala.

### Nil versus zero values

It's tedious to model values that are empty or omitted versus intentionally set to a zero value. See, for example, [this migration](https://github.com/stripe/stripe-go/issues/560) in Stripe's Go bindings. In our code, we often return a pointer and error e.g. `(*string, error)`. This kind of breaks type safety by introducing the possibility of dereferencing nil pointers. You can check `if res == nil` as well as `if err != nil` but the compiler can't save you from forgetfulness or laziness.

### Lack of object-oriented expressiveness

Go suggests interfaces and type embedding to replicate useful object-oriented behavior that comes naturally in other languages. These tools turn out to be super limiting and, in various cases, we've accidentally worked around the type system. This leads to a temptatio

This one's been covered at length in the Go community. Here's a really good summary: [https://blog.golang.org/why-generics](https://blog.golang.org/why-generics).

## Conclusion

Initially it took me a bit of time to warm up to Go. There were some confusing parts to getting started, as I described with GOPATH. Coming from languages like Ruby and Scala also meant that a shift (or two) in mindset was required. In two years of working in the language, though, I've come to really enjoy its simplicity and philosophy of explicitness. 

At Assembled the company, the language is really well suited for our use case, a mostly standard web application. I think highly of the ecosystem—it feels like we're working with thoughtfully designed and well maintained tools. As a result, there's baseline less effort required to provide a stable service while rapidly making changes to the codebase.

<p>&nbsp;</p>
**Thanks** to Anil Vaitla, John Wang, Kaytlin Louton, and Kyle Conroy for sharing thoughts.
