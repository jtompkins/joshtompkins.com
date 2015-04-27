---
layout: post
title:  "How I Learned to Stop Returning Null and Use the Null Object Pattern"
photo: /assets/img/headers/subway-seats.jpg
photographer-url: https://unsplash.com/matthewwiebe
photographer: Matthew Wiebe
location: Gainesville, Florida
excerpt: A modest request to stop returning f**king null everywhere. A presentation of potential alternatives.
---

I *hate* checking for null.

It feels important to start out with that, because avoiding null checks requires discipline and effort. The end result, though, is cleaner, simpler, better looking code.

Fair warning: the solution involves design patterns. Get your astronaut hat ready.

!!note - expand on this opening

## The Revenge of the Void

Failing to check for null is often seen as a beginner's mistake, probably because beginners often make it.

{% highlight c# %}

public class Account {
    public int Id { get; set;}
    public string Name { get; set;}
}

public class AccountRepository {
    public IEnumerable<Account> GetByIds(IEnumerable<int> ids) {
        // lots of database code
    }
}

var ids = new int[] { 1, 2, 3 };

var accounts = (new AccountRepository()).GetByIds(ids);

for (var account in accounts)
    Console.WriteLine(account.Name);

{% endhighlight %}

See that code? *All* of my code used to look like that. It's simple, easy to understand, and the intention of the code is obvious. It's good code.

Until, of course, an `Id` gets passed in that doesn't exist in your data source. Then you get an exception. Because we're developers, and programming is terrible, that exception is probably going to happen in production, so enjoy that.

Null references always crop up in situations that "can't happen". Until they happen.

Enough of that and your boss will eventually make you a poster like this:

![Programming](/assets/img/posts/2015-4-26-stop-returning-null/programming-poster.jpg)

So fine. Null checks all around. Eventually, the fear of a null reference in production drives you into a very specific and finely-tuned paranoia. Your code starts to look like this:

{% highlight c# %}

var ids = new int[] { 1, 2, 3 };

var accounts = (new AccountRepository()).GetByIds(ids);

if (accounts == null)
    return;

for (var account in accounts) {
    if (account == null)
        continue;

    Console.WriteLine(account.Name);
}

{% endhighlight %}

...and so on.

Here's the thing about putting null checks everywhere in your code: it *feels* like you're writing good code, but you're not. *You're not.*

Null checks are *viral*. They start in one area of the code and eventually leech into all of it. All that extra code has a cost. Eventually you will introduce a bug into code intended to prevent bugs.

I can tell you from experience, recognizing the irony of that fact will not make the feeling of incompetence any better.

What's worse - all these null checks aren't addressing the real problem. They're just treating a symptom, and that symptom is the fact that you're *returning null from things*.

Let's explore some ways to avoid that.

## Suddenly, a Shortage of Nothing

There are three categories of values you'll need to watch out for:

* "Value" types, like `Int32` or `Decimal`, which always have a value,
* `String`, which is technically a value type but still can potentially be null, and
* Reference types, like `Object` and its many descendants, all of which can be null.

Value types are easy - they always contain a value and so are safe from null references.

!!note - discuss nullable types

`String` can be null. Here's how you avoid null references when returning `String`s: don't return null. Return `String.Empty` instead. `.Empty` as a constant representation of a non-existent value is important, so keep it in mind. We'll wander back here later.

Reference types can all be null, and it's mostly on their behalf that we're here.

Before we launch into a solution for the null reference problem, it's worth mentioning that there's one class of reference types for which there's an easy fix: `Enumerable<T>`. The .NET framework provides [`Enumerable.Empty`](https://msdn.microsoft.com/en-us/library/vstudio/bb341042(v=vs.100).aspx) which is both simple and an elegant solution to our problem. Friends don't let friends return a null `Enumerable`.

## An Introduction to the Null Object Pattern

The [Null Object Pattern](https://sourcemaking.com/design_patterns/null_object) is, at its heart, about defining a type that represents *nothing*. This sounds more complicated than it is, so let's take a look at some code:

{% highlight c# %}

public class Account {
    public int Id { get; set;}
    public string Name { get; set;}
}

public class EmptyAccount : Account {
    public Account() {
        Id = 0;
        Name = String.Empty;
    }
}

{% endhighlight %}

That's it. That's the whole pattern. `EmptyAccount` has two characteristics: it defines reasonable (**non-null**) defaults for all of its properties, and, critically, it responds to the **same API** as its parent class, `Account`.

I like to name my null types `Empty` to match `String.Empty` and `Enumerable.Empty`, but you can call yours anything you want. Because `EmptyAccount` extends `Account`, it'll work in any code that returns or consumes an `Account`.

With `EmptyAccount` in place, we're back to our original code:

{% highlight c# %}
var ids = new int[] { 1, 2, 3 };

var accounts = (new AccountRepository()).GetByIds(ids);

for (var account in accounts)
    Console.WriteLine(account.Name);

{% endhighlight %}

Note the beautiful absence of null checks. Speaking of absence...

## Evidence of Absence

There will eventually come a time when you need to know that a *nothing*, rather than a *something*, was returned, and you won't have the crutch of null to rely upon.

The easy way is to examine the type of your variable with `is`:

{% highlight c# %}
var acct = new AccountRepository.Get(id);

if (acct is EmptyAccount) {
    //...
}

{% endhighlight %}

This approach has an advantage over a standard null check - it makes your intentions clear. You can encounter a null value for a lot of reasons. Checking for `EmptyAccount` is a check for a specific kind of *nothing*.

Of course, nothing in programming is free. Comparisons against a type are slower than normal object comparisons, so let's explore another approach that uses a simple comparison check.

First, make `EmptyAccount` into a [singleton](http://csharpindepth.com/articles/general/singleton.aspx), and then expose the single instance of `EmptyAccount` through a static property on `Account:`

{% highlight c# %}
//singleton code here
{% endhighlight %}

Once this structure is established, returning an `EmptyAccount` is as easy as returning `Account.Empty`. As a bonus, accessing our null object is now consistent with other empty constants in the framework.

Checking for the absence of a value is now straightforward:

{% highlight c# %}
// comparing against Account.Empty
{% endhighlight %}

The default equality implementation in C# for Object is a reference comparison. Since there's only one instance of `EmptyAccount`, comparing two references to `Account.Empty` will always evaluate to `true`.

There is one potential problem lurking here - if the parent type overrides `.Equals`, comparisons to your empty singleton can become unreliable.

{% highlight c# %}
//equality overrides here
{% endhighlight %}

It's probably best to avoid the singleton approach if you're working in an environment that routinely overrides equality for your types. You should probably also try to avoid environments that override equality, but that's just one man's opinion.

You might be thinking at this point that you can just override the equality comparison on your null type, perhaps by enforcing a reference check:

{% highlight c# %}
//overriding equals on emptycontact
{% endhighlight %}

This solution might seem good on the surface, but in practice it will introduce subtle, hard-to-reproduce bugs into your code, mostly because it will make the order of the arguments in an equality comparison suddenly matter:

{% highlight c# %}
//equality comparisons that aren't
{% endhighlight %}

Seriously, don't do this. Down this path lies madness.

!!note - come up with a closing paragraph