---
title: "Why I Chose Go and Why I'm Not Going Back"
date: 2026-06-04
draft: false
tags: ["go", "backend", "cloud"]
---

# Why I Chose Go and Why I'm Not Going Back

_The accidental start, a cloud rabbit hole, and what I've built since_

---

I didn't choose Go. Go kind of chose me.

It started with a job application. I had applied for a backend developer position, two years of backend experience behind me, and the take-home task was to build a Notification Service. The catch: they wanted it in Go. I had never written a single line of it. I had to Google the syntax to write a main function.

What happened over the next few days changed how I think about backend engineering.

<div style="text-align: center;">

![Go Gopher](/images/go-banner.png)

</div>

---

## The Assignment That Started Everything

There's something clarifying about learning a language under deadline pressure. You can't afford rabbit holes. You just need to make something work.

What surprised me wasn't that I _could_ build the notification service. It was how quickly I could. Go's standard library had everything I needed: HTTP, JSON, concurrency primitives, all consistent and well-documented. No framework to fight.

The goroutine model hit me especially hard. I needed to fan out notifications to multiple channels concurrently (email, push, in-app) without one slow channel blocking the others. I spun up goroutines, used a channel to collect results, and the code read like what I was _thinking_. That almost never happens when you're picking up something new.

I didn't want to stop writing Go after that.

---

## Why the Numbers Make Sense

I wasn't just feeling something. The data backs it up.

Go has around **5.8 million developers** worldwide as of 2024, up from 3.8 million in 2020. In GitHub's 2024 stats, it ranked as the **third fastest-growing language**, behind only Python and TypeScript. According to the Stack Overflow Developer Survey, **14.4% of professional developers** now use Go as a primary language.

The satisfaction numbers are even more telling: **93% of Go developers** reported being satisfied with the language in the 2024 Go Developer Survey. That's not a language people are tolerating. That's a language people are actively choosing to stay in.

And on the market side, median Go developer salaries sit around **$75,000 to $135,000 globally**, with senior roles in the US regularly reaching $160,000+. The demand is real, and the supply of experienced Go developers hasn't caught up yet.

---

## The Google Cloud Rabbit Hole

That assignment sent me somewhere I hadn't planned. I enrolled in Google Cloud labs, partly curious, partly sensing I was onto something. The deeper I went into cloud infrastructure, the more Go appeared. Not as an option. As the default.

**Kubernetes. Terraform. Docker. Prometheus.** The tooling that the entire cloud-native world runs on was built in Go. The language hadn't just become popular in this space. It had shaped it.

The reason became obvious quickly. Go compiles to a single static binary with no runtime and no dependency hell. For cloud deployments where containers need to start fast and run lean, that's enormously valuable. Cloudflare's 2024 data showed Go accounting for **12% of all API calls**, up from 8.4% the year before, overtaking Node.js for automated API traffic.

My interest stopped being accidental after that. I was choosing Go.

---

## Building FailSafe

I wanted to push on Go's concurrency in a serious way. That led to **FailSafe**, a chaos engineering and resilience testing platform I built as an academic project.

FailSafe needed to orchestrate concurrent fault-injection experiments across distributed systems while continuously observing and measuring what was happening. Goroutines made the parallelism feel natural. The `context` package, which propagates deadlines and cancellation signals through your entire call stack, was something I hadn't seen handled this cleanly anywhere before. When you're running timed experiments that need to terminate gracefully and clean up after themselves, `context` is the difference between correct behaviour and subtle bugs that only surface under load.

I'll do a full deep-dive on FailSafe in the next post. But building it showed me what Go is actually for: systems where performance and concurrency aren't nice-to-have, they're the point.

---

## Why I'm Staying

Go feels _considered_. The compiler catches mistakes early. The standard library handles more than you expect. When something goes wrong, the stack trace points to your code with no framework magic to trace through.

Over 80% of Go developers learned it _after_ starting their professional careers, according to the 2025 Go Developer Survey. It's not a beginner's language. It's a language experienced engineers actively seek out. I'm one of them now.

---

## Where to Start If You're Curious

If this resonated and you want to try Go yourself, here's what actually works:

- **[A Tour of Go](https://go.dev/tour/)** - The official interactive tutorial. Write and run Go in your browser. Start here, no setup needed.
- **[Go by Example](https://gobyexample.com/)** - Annotated code snippets for every concept. I still use it as a reference.
- **[Effective Go](https://go.dev/doc/effective_go)** - Official guide to writing idiomatic Go. Read this once you have the basics.
- **[The Go Programming Language](https://www.gopl.io/)** (book, paid) - The definitive text. Written by the people who know the language best.
- **[Boot.dev](https://www.boot.dev/)** - Interactive Go course if you prefer structured, guided learning.

The fastest way to actually learn it: pick a small service you'd normally build in your usual stack. Build it in Go instead. The friction is lower than you think.

---

_Next up: a full technical deep-dive into FailSafe, what it does, how Go's concurrency made it possible, and what I learned building it._

---

_Follow this blog to catch that one when it drops._
