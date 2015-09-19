---
layout: post
title:  "Real-World React, Part 1: Introduction"
photo: /assets/img/headers/fire-sky.jpg
photographer-url: https://unsplash.com/brenomachado
photographer: Breno Machado
location: Gainesville, Florida
excerpt: Starting a new Javascript project is harder than you might think.
---

*This post is part of a series on **Real-World React & Flux**.*

Lately I've been experimenting with React at work. I like it a lot - expressing a user interface in terms of nested components is very natural, and rendering each component in terms of its state (while handing off the messy responsiblity of actually displaying the rendered HTML to the framework) makes building complicated interfaces easy.

*Flux*, the de-facto standard way to architect a React app, is pretty straightforward too, boiling down complex data flows to a single, one-dimensional flowchart:

![Flux Flowchart](https://facebook.github.io/flux/img/flux-simple-f8-diagram-1300w.png)

Unfortunately, *grasping* Flux is very different from actually *implementing* Flux. See, Flux isn't really a framework - it's more of a specification, one that (very loosely) describes how to lay out a React application. Since Facebook didn't release a canonical implementation, there are dozens of Flux libraries available. The quality of documentation for all of these libraries is, well, uneven, and until recently there hasn't been a clear consensus on which implementation is the "best" (even for a very loose definition of "best").

If you want to build a React app from scratch, picking a Flux library isn't the only decision you need to make - you'll need to choose a task runner, a testing framework, a transpiler (if you want to use ES6 - and you do), a CSS preprocessor, and on and on and on.

The most common advice you'll get when trying to start a new app is to use one of the [many](https://github.com/christianalfoni/flux-react-boilerplate), [many](https://github.com/ellbee/redux-boilerplate) ([many](https://github.com/erikras/react-redux-universal-hot-example)) application "boilerplate" projects available on Github. These projects make a lot of decisions for you, often pulling in all of the pieces you need and doing the configuration necessary to bind them all together.

Using a boilerplate project doesn't really work for me - I like knowing what all the pieces of my project are, why they were chosen, how they're configured, and what's likely to break if I change something. I like to think this is an admirable trait, but it's more likely it just creates more work for me.

C'est la vie.

More work or not, I'm going to try and build a React app, from scratch, using the current state-of-the-art in the React ecosystem. Wish me luck.

## The Project - A Notification Timeline for the TradePMR API

I work for a company named [TradePMR](http://www.tradepmr.com). We're a custodian and broker-dealer based in Gainesville, Florida, and we're trying hard to make waves in an industry dominated by giant, entrenched encumbents. One of the ways we do that is by investing heavily in technology - and then opening up that technology as much as we can. Our public [REST API](https://developers.tradepmr.com) exposes a number of services to our partners, and one important one is our *Notification API*, which provides a way for 3rd-party applications to feed actionable information to our users. In an upcoming release, we're going to open up read access to our system's notification streams, and as a way to test the new services, I'm going to build a small, mobile-oriented application to consume those notifications and present them in timeline form.

When I'm done, the app will (hopefully) look like this:

![Notification Mockup](/assets/img/posts/2015-9-19-real-world-react-1/Notifications-App.png)

Eventually. Probably.

Oh, one more thing: at TradePMR we like to give our projects code names (because it's [not that weird to have a code name](http://star-lord-hijacked-serenity.tumblr.com/post/105252667039/starrdork-its-cool-to-have-a-codename-its)), usually places around Gainesville. Even though this is a personal project, I'm still going to assign it a code name. Let's call it "[Millhopper](http://devilsmillhopper.com)".

## Choosing the Technology Stack

To build this application, I'm going to try to pull together a bunch of packages that represent the current state-of-the-art in React tooling (for this week, anyway):

### ES6 & Babel

ES6 is the future of Javascript - and the first release in a very long time to make significant improvements to the syntax of the language itself. New keywords like `let` and `const`, the new `class` syntax, arrow functions - all of these things are helping Javascript shed its old image of a bad language with a few "good parts".

Unfortunately, ES6 doesn't (yet) have significant adoption in web browsers, so if we want to use it, we'll have to use something called a *transpiler* - a piece of software that translates one programming language into another. The best of the best right now is [Babel](https://babeljs.io), so that's what I'll be using. Babel has great support for most ES6 featuers (and a few ES6+ features, too), and excellent tooling. As of this writing, it's pretty much the *only* choice for a transpiler outside of adopting an entire new language like [TypeScript](http://www.typescriptlang.org) or [CoffeeScript](http://coffeescript.org). Those are both fine choice too, of course (TypeScript especially has a lot of interesting features).

### Webpack

Even for the Javascript ecosystem, [Webpack](http://webpack.github.io) seems to have transitioned from relative obscurity to explosive popularity in record time. Webpack calls itself a Javascript *module bundler* - essentially, it's a piece of software that can package up all of the dependencies of a typical application (Javascript code, CSS, even images and fonts) and then expose them as modules that can be combined together and used in interesting ways. Thanks to a pluggable architecture, Webpack can also handle some duties usually delgated to a task runner, like compiling ES6 code, processing and minifying CSS, or optimizing images. I've found that while Webpack can do a lot, getting started is actually pretty easy, though the documentation is a little spotty.

### Redux

[Redux](http://rackt.github.io/redux/) represents a rethinking of the typical (inasmuch as "typical" exists in something so new) Flux architecture. Redux considers an application from a very functional-programming point of view, eliminating things like multiple singleton Stores in favor of a single source of immutable state. Of all the tech I'll be using for this project, Redux is the one I'm most looking forward to exploring.

### ImmutableJS

[ImmutableJS](https://facebook.github.io/immutable-js/) is a Facebook project that provides a number of immutable data structures that will hopefully make working with Redux's immutable state easier. As I write this, I'm not sure how necessary this library really is, but the ideas are interesting and the API looks solid, so I'm going to give it a shot.

### Mocha & Should

[Mocha](http://mochajs.org) is a Javascript testing framework. I'm chosing it for a couple of reasons - it's well documented, it has excellent support for tests written in ES6 (via the [Babel transpiler](https://babeljs.io)), and I like that Mocha allows me to choose my own assertion library. For this project, I'm going to use [Should](https://shouldjs.github.io), mostly because I like the English-like assertion syntax.

I've never used Redux, ImmutableJS, Webpack, or Mocha before, so this should be a pretty interesting project. In the [next post](/2015/09/21/real-world-react-2/), I'll be walking through project setup, which is really much more involved than it probably should be. I hope you'll join me there!

