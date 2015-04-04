---
layout: post
title:  "An Experiment in Vanity, Part Two: Choosing Technology"
photo: /assets/img/headers/mac-laptop.jpg
photographer-url: https://unsplash.com/firmbee
photographer: William Iven
location: Gainesville, Florida
excerpt: Wherein I embark on a new learning adventure and, while stroking my ego, very quickly get in over my head.
---

## Choosing Some Technology

First, I need to choose a language. This one is pretty easy &mdash; I've been looking for an excuse to build something in Ruby for a long time.

I think there will be two major Ruby components:

* A worker, which will be responsible for harvesting data from my various data sources on a regular schedule.
* The site itself, which will serve the dashboard UI.

The worker will be a basic Ruby program, triggered by a CRON job. The server will be written in Rails, since that's the standard web framework in the Ruby world and I want to learn how to use that, too.

Now that the language question is settled, I'll need to figure out how I'm going to be storing all of this data. Each repository will probably offer data with very different schemas, and I'll want to add new data sources over time without completely rebuilding my data layout.

Since I don't know ahead of time how my data will structured, and even once I figure it out, the structure will probably be changing over time, I'm going to choose MongoDB to hold my data.

I don't know all that much about Mongo, which is kind of the point. Going with something like Postgres would probably be more practical, but practicality isn't really a requirement of this project anyway.

Moving right along.

This is a web project, so the front end will be HTML, CSS, and Javascript. I've been really facinated with React lately, so let's build the client-side app using that. At this point I'm pretty undecided between writing in ES6 (using the excellent Babel transpiler) or going with Coffeescript, which will probably require less of a cognative shift coming from writing the Ruby backend.

There's a lot of code to write between now and the UI, so I'll just save that decision for later.

## Hosting

Let's talk about hosting this thing. The obvious option here is Heroku - they have excellent support for Ruby sites like this, and pushing code here requires a minimum of Linux configuration knowledge.

A *minimum* of Linux knowledge? That doesn't really sound like what we're going for here, does it?

I think I want to host this thing on a Linux VPS. Fortunately, the options in that space are plentiful and the cost is low. Additionally, the data I'll be hosting isn't exactly important to keep secret, so if I mess up on the server configuration and data gets leaked, nothing of any serious import will be lost.

Of more serious concern is that I know essentially nothing about Linux server configuration. Thankfully, I found Servers for Hackers a while back while messing with a Node project. That should give me a good starting point.

Hosting this project on a VPS gives me an idea: I'll need to configure the server anyway, so why not try building this project in a Vagrant VM? Now I don't have to configure this thing twice - once on my local machine, and once on the server.

## A Long-Awaited Conclusion

That's a lot of new tech to try to pick up all at once. Am I in over my head here?

Yes.

Yes I am.

Thankfully, this thing can be built in small steps. Step one will be to get Vagrant installed, Ruby set up in the VM, and see if I can't get the worker to pull down some sample data from somewhere.