---
layout: post
title:  "Flux from the Ground Up, Part 1: Project Setup"
photo: /assets/img/headers/fire-sky.jpg
photographer-url: https://unsplash.com/brenomachado
photographer: Breno Machado
location: Gainesville, Florida
excerpt: Starting a new Javascript project is harder than you might think.
---

Lately I've been experimenting with React. I really like it - it's a completely different approach to building an application than the MVC/MVVM methodology that dominates modern Javascript development.

Unfortunately, building up a working project from the ground up is much harder than it really should be. The ecosystem is in constant flux, so documentation, especially around starting up a project, is pretty uneven. What documentation *is* out there becomes out of date quickly, too. It's hard to know what tools to use, and once you've picked out your tools, it's hard to figure out how to get them all to work together smoothly.

Since I'm going through this crucible right now, I'm going to write up the tooling choices I make and the steps I take to get my tools working, in the hope that someone else might have an easier time later.

## The Project - A Notification Timeline for the TradePMR API

My company's product is powered by an API, and one of the things the API can deliver is a list of "notifications" - alerts from our product, letting our users know something happened. We're going to build a small, mobile-focused application to consume those notifications and present them in timeline form. It's going to look like this:

![Notification Mockup](/assets/img/posts/2015-9-6-real-world-flux/notifications-app.png)

Well, it'll eventually look like that. Probably.

To build this thing, I'm going to pull in the current state-of-the-art in React tooling: React itself, Redux (for the Flux layer), ImmutableJS (for data management), Webpack (for dependency processing and packaging), and Mocha/Should (for testing). I'd like to write as much code as possible in ES6. All of the styles will be written in SASS.

I've never used Redux, ImmutableJS, Webpack, or Mocha before, so this should be pretty interesting.

## Installing dependencies

Before I can start writing code, I'm going to need to install some dependencies. All of my tooling requires NodeJS, so I'll get that installed first. I'm using a Mac for this project, but only the Node installation process tends to be platform-specific.

The easiest way to get Node onto my system is probably the official installer.
Node moves quickly though, and I'd like to make it easy to keep my installed version a) easily removable and b) easily upgradeable. From what I've read, the best way to meet both of those requirements is to install Node using NVM, the Node Version Manager, which provides an easy CLI for installing new Node versions (and removing old ones).

Installing NVM is as easy as executing a command from the OS X Terminal:

{% highlight bash %}
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.26.1/install.sh | bash
{% endhighlight %}

*If you're not familiar with terminal or command line for your platform, now would be a good time to do some extracurricular reading - a lot of Javascript tooling doesn't have a GUI.**

Installing NVM doesn't actually install any versions of Node, so I'll need to do that next. These days, the most advanced and feature-rich version of Node is actually the io.js fork, so let's install that:

{% highlight bash %}
nvm install iojs-3.3.0
{% endhighlight %}

NVM allows for multiple Node installations, so before I can use the io.js I just installed, I need to tell NVM that I want to use it as my default Node version:

{% highlight bash %}
nvm use iojs-3.3.0
{% endhighlight %}

Okay, so now I've got Node installed and I can use the node package manager NPM to install the rest of my dependencies. There are a bunch of them:

{% highlight bash %}

npm install -g webpack webpack-dev-server mocha

{% endhighlight %}

That command tells NPM to install all of the tooling I need globally, so I can use them from anywhere.

## Defining the project structure

With the tooling in place, I can actually start working on my project. I'm a big believer in source control, so I'm also going to initialize a Git repository to hold my code:

{% highlight bash %}
mkdir notifications
git init
{% endhighlight %}

This project is eventually going to have a bunch of project-specific dependencies, which I'm going to get from NPM, so I'm going to let NPM create a `project.json` file for me:

{% highlight bash %}
npm init
{% endhighlight %}

`project.json` is used for a lot of things: listing dependencies, defining scripts for building and testing the project, and probably more stuff I don't know about yet. `npm init` makes sure the file gets created correctly. I'm not sure how to answer some of the questions I'm asked - in those cases I usually accept the defaults and hope for the best.

Okay, the basic stuff is done. Time to start working out where to put all the files I'll be creating. There are a *lot* of opinions on how a Javascript project should be structured, and there doesn't seem to be any clear consensus on the "right" way to do it.

For my projects, I usually stick with a four-part structure:

* `/` holds all of my configuration files, READMEs, and build scripts
* `/app` holds all of the application code
* `/css` holds all of the SASS files
* `/test` holds my tests. This one is a convention for a wide range of Javascript test runners.

Eventually, I'll also end up with `/dist`, which will hold the production-ready source produced by Webpack.

Once I create all of those directories, my project looks like this:

![Project Structure]()

## Configuring Webpack

Webpack is the newest trend in client-side development, but the official website does a pretty poor job explaining what it *actually does*. Basically, Webpack takes all of the artifacts that make up your project - javascript code, CSS files, HTML, even images and fonts - and converts them into *modules*, which you can combine in interesting ways. When you're ready to actually run your project, Webpack can take all of your modules and compile them down into optimized bundles, ready for production use.

Right now, I just want Webpack to combine all of my Javascript files into a single file I can drop onto a web page.

Remember when I installed Webpack globally earlier? That makes it convenient to run Webpack builds from the command line, but I also want to make sure that anyone else who might work on this project in the future can run builds without having to do that, so I'm also going to install Webpack locally:

{% highlight bash %}

npm install webpack --save-dev

{% endhighlight %}

Now that Webpack is installed both globally and locally, I need to start configuring it. That's done with a file called `webpack.conf.js`, so I'm going create that file and put the minimum required config into it:

{% highlight javascript %}

module.exports = {
  entry: __dirname + '/app/index.js',

  output: {
    path: __dirname + '/dist',
    filename: './app.js'
  }
}

{% endhighlight %}

In order to build my app, Webpack needs to know two things:

1. Where my app execution starts, which Webpack calls the *entry point*.
1. Where the compiled version of my app should be written - Webpack calls this the *output*.

In this case, I'm defining the entry point to be a file called `index.js` in the `/app` directory, and the compiled output to be a file called `app.js`, which Webpack will write to a directory called `/dist`. Don't worry about that `__dirname` - it's a magic variable, created by Node, that contains the path to the project directory.

Before I go any further, I'm going to verify that my configuration works. I create the entry point file in `/app/index.js`, and put some JS code in it:

{% highlight javascript %}

console.log('hello, webpack!');

{% endhighlight %}

Now I can run a Webpack build and confirm that the compiled file shows up in `/dist/app.js`:

{% highlight bash %}

webpack

{% endhighlight %}

### Handling ES2015 Compilation

Webpack knows about the basic structure of my project and can process my Javascript, but by default, Webpack can only handle ES5. Before I can go any further, I need to teach it how to handle ES6 files. You do this by defining a *loader* in your `webpack.conf.js`:

{% highlight javascript %}

module.exports = {
  entry: __dirname + '/app/index.js',

  output: {
    path: __dirname + '/dist',
    filename: './app.js'
  },

  module: {
    loaders: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: 'babel-loader'
      }
    ]
  },

  resolve: {
    extensions: ['', '.js']
  }
}

{% endhighlight %}

A Webpack *loader* allows you to add code to the Webpack processing pipeline. In this case, I want to tell Webpack that all of the files ending in `.js` in my project should be processed by the `babel-loader`, which converts ES6 code into ES5 code using the Babel transpiler. I also tell Webpack to exclude all the files in `/node_modules`, because compiling that stuff would take forever and probably break things.

I also added a resolver for `.js` files, which will let me import files into my ES6 modules without including the file extension. That's not *strictly* necessary, but makes the import syntax a little cleaner.

Let's change `index.js` to include some ES6 code to make sure everything's working:

{% highlight javascript %}

let hello = () => {
    console.log('hello, webpack!');
};

hello();

{% endhighlight %}

At this point we're starting to write a little bit of code, so let's create an HTML file in the `/app` where we can run the Javascript produced by Webpack:

{% highlight html %}

<!DOCTYPE html>

<html lang="en">
  <head>
    <meta charset="utf-8">

    <title>Notifications</title>
  </head>

  <body>
      <script src="app.js"></script>
  </body>
</html>

{% endhighlight %}

This file should end up in the `/dist` folder during a build, but right now, Webpack only includes Javascript files in the build. We can fix that by adding a loader for HTML files:

{% highlight javascript %}

module.exports = {
  entry: {
    javascript: __dirname + '/app/index.js',
    html: __dirname + "/app/index.html"
  },

  output: {
    path: __dirname + '/dist',
    filename: './app.js'
  },

  module: {
    loaders: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: 'babel-loader'
      },

      {
        test: /\.html$/,
        loader: "file?name=[name].[ext]",
      }
    ]
  },

  resolve: {
    extensions: ['', '.js']
  }
};

{% endhighlight %}

Running the Webpack build now should move the `index.html` file into the `/dist` folder. Javascript running in a file opened directly by a web browser is pretty heavily sandboxed, so I've got to find a way to serve my files from an actual web server.

### Live-reload and webpack-dev-server

Luckily, Webpack has a development web server we can use called `webpack-dev-server`. Much like Webpack itself, I'll need to install it both globally and locally:

{% highlight bash %}

npm install webpack-dev-server --save-dev

{% endhighlight %}

When I installed this package, I got a bunch of really scary-looking error messages in my terminal. I don't know why or how to fix them, but they didn't seem to matter - running the server works as expected.

Once the server is installed, launch it from the terminal and then navigate to `http://0.0.0.0:8080`:

{% highlight bash %}

webpack-dev-server

{% endhighlight %}

Most of the documentation suggests that you should be able to view your site at `localhost:8080`, but for whatever reason, this didn't work for me out of the box. I had to explicitly tell the server to serve from `localhost`:

{% highlight bash %}

webpack-dev-server --host localhost --port 8080

{% endhighlight %}

Typing that command over and over is a bit of a drag. Luckly, NPM supports running scripts defined in `package.config`, so I added that command to the `start` script:

{% highlight javascript %}

"scripts": {
  "start": "./node_modules/.bin/webpack-dev-server --host localhost --port 8080"
}

{% endhighlight %}

Since I'm scripting out launching the server, I'm using the locally installed `webpack-dev-server` package, just in case it's not installed globally.

Now I can launch the web server with `npm start`.

`webpack-dev-server` can do more than just serve your files - it can also live reload your code as you change it. It does this by compiling your code on the fly and serving it from memory via a virtual 'public' path. You'll need to configure this path in your `webpack.conf.js`:

{% highlight javascript %}

output: {
  path: __dirname + '/dist',
  filename: './app.js',
  publicPath: '/public/'
}

{% endhighlight %}

Once you've added this, make sure your `index.html` file loads your code from a relative path:

{% highlight html %}

<script src="app.js"></script>

{% endhighlight %}

This step is important because in live reload mode, `webpack-dev-server` never writes your files to disk.

When you launch the server, you need to tell it to run in 'hot' mode by adding `--hot --inline` to the command:

{% highlight javascript %}

"scripts": {
  "start": "./node_modules/.bin/webpack-dev-server --hot --inline --host localhost --port 8080"
}

{% endhighlight %}

Now when you run the server, instead of navigating to `http://localhost:8080`, navigate to `http://localhost:8080/webpack-dev-server/` instead. Changing your code while the server is running should reload the content automatically. If you don't add `webpack-dev-server` to your URL, *the live reload doesn't work*, so be mindful of that when testing.

## Setting up the testing environment

Now that we have a functional build process and a way to serve the site, let's get a test harness in place. For this project, I'm going to be doing BDD-style testing with Mocha and the Should assertion framework, both of which need to be installed locally:

{% highlight bash %}

npm install mocha should --save-dev

{% endhighlight %}

Writing a Mocha test is as simple as dropping a `test.js` file into the `/test` directory:

{% highlight javascript %}

var should = require('should');

describe('A test framework', function() {
   it('should be able to run a test', function() {
    1.should.equal(1);
   });
});

{% endhighlight %}

Run the test with the `mocha` command. It should pass.

### A brief aside for OS X and iTerm2 users

If you're running iTerm2 on OS X with the popular *Solarized Dark* theme installed, you might notice something strange - the description of the test doesn't show up in the Mocha output:

![Missing Descriptions]()

That's happening because Mocha outputs the test description in a non-standard terminal color that's the same color as the Solarized Dark background - or very close to it. There's some debate over whether this is a [Solarized bug or a Mocha bug](https://github.com/mochajs/mocha/issues/802).

It's fixable by either switching themes (I've found the [Tomorrow Night Eighties](https://github.com/chriskempson/tomorrow-theme) theme works well), or increasing the contrast in iTerm2's settings:

![Increasing Contrast]()

### Dealing with ES2015 (again)

Much like Webpack, Mocha doesn't know how to deal with ES6 files out of the box. Also like Webpack though, Mocha allows for pluggable 'compilers', which can allow us to write our tests in ES6.

First, we need to install Babel locally:

{% highlight bash %}

npm install babel --save-dev

{% endhighlight %}

Now we can launch Mocha, specifying that we want to use the Babel transpiler:

{% highlight bash %}

mocha --compilers js:babel/register

{% endhighlight %}

Remembering that `--compilers` switch every time you want to run your tests is a bit of a drag. Lucky for us, Mocha has a way to specify standard command-line arguments. If you create a file called `mocha.opts` in your `/test` directory, you can tell Mocha which options you want it to always include:

```
--compilers js:babel/register
--recursive
```

Note the `--recursive` switch - as the number of tests in the project grows, I'm going to want to split them up into subfolders. Mocha doesn't search those folders unless you include the recursive switch.

As an alternative to the `mocha.opts` file, you can also add Mocha to the `test` command in your `package.json` file:

{% highlight javascript %}

"scripts": {
  "test": "./node_modules/.bin/mocha --compilers js:babel/register --recursive",
  "start": "./node_modules/.bin/webpack-dev-server --hot --inline --host localhost --port 8080"
}

{% endhighlight %}

Now you should be able to run your tests with `npm test`.

## SO MUCH CONFIGURATION

That's nearly 2,500 words and I still haven't written any app-specific code. Remember when I said starting a Javascript project is harder than it needs to be? Now you understand why.

We'll get around to actually writing some React code in the next post.