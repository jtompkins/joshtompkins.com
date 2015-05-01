---
layout: post
title:  "Stop Returning Null"
photo: /assets/img/headers/subway-seats.jpg
photographer-url: https://unsplash.com/matthewwiebe
photographer: Matthew Wiebe
location: Gainesville, Florida
excerpt: Stop returning null. Seriously, just stop it.
---

I *hate* checking for null.

Null checks make code worse, not better. They're like fire ants - they show up everywhere, and once you start seeing them, they're basically impossible to fully remove.

The good news is that you can write C# code that doesn't check for null and **is still safe from null reference errors**. It just takes discipline, a little extra code, and the will to make it happen.

Fair warning: banishing null checks from your code involves design patterns. Get your [astronaut hat](http://www.joelonsoftware.com/articles/fog0000000018.html) ready.

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

See that code? *All* of my code used to look like that. It's simple, easy to understand, and the intention of the code is obvious. It has all of the *characteristics* of good code - but in reality, it's a time bomb waiting to explode.

The good news is that you're a developer, and the odds are *never* [in your favor](https://www.youtube.com/watch?v=_s7qgNMqDJI), so when (not if) this bomb explodes, it's probably going to do it in production.

Enjoy that.

Novice developers often argue that they don't have to check for null, because reaching the null value execution path "can't happen". Say that enough and your boss will eventually make you a poster like this:

![Programming](/assets/img/posts/2015-5-1-stop-returning-null/programming-poster.jpg)

(I know because that poster is mine.)

Eventually, you learn your lesson. The fear of a null reference in production drives you into a very specific and finely-tuned brand of paranoia. Your code starts to look like this:

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

Here's the thing about putting null checks everywhere in your code: it *feels* like you're writing good code, but you're not. **You're not.**

Stop and let that sink in for a minute.

Null checks are *viral*. They start in one area of the code and eventually leech into all of it. All that extra code has a cost. Eventually you will introduce a bug into code intended to prevent bugs.

I can tell you from experience, recognizing the irony of that fact will not make the feeling of incompetence you'll have any easier to tolerate.

What's worse - all these null checks aren't addressing the real problem. They're just treating a symptom, and that symptom is the fact that you're *returning null from things*.

Let's explore some ways to avoid that.

## Suddenly, a Shortage of Nothing

There are three categories of values you'll need to watch out for:

* Value types, like `Int32` or `Decimal`
* `String`, which is technically a value type but still can potentially be null
* Reference types, like `Object` and its many descendants

Value types are easy - they always contain a value and so are safe from null references.

`String` can be null. Here's how you avoid null references when returning `String`s: don't return null. Return `String.Empty` instead. `.Empty` as a constant representation of a non-existent value is important, so keep it in mind. We'll wander back to it later.

Reference types can *all* be null, and it's mostly on their behalf that we're here.

It's worth mentioning that there's one class of reference type for which there's an easy way to avoid returning null values: `Enumerable<T>`. The .NET framework provides [`Enumerable.Empty`](https://msdn.microsoft.com/en-us/library/vstudio/bb341042(v=vs.100).aspx), which returns an empty list, neatly avoiding the null problem.

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

That's it. That's the whole pattern. `EmptyAccount` has two characteristics: it defines reasonable (i.e., *non-null*) defaults for all of its properties, and, critically, it responds to the *same API* as its parent class, `Account`.

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

The easiest way to identify an empty value is to examine the type of your variable with the C# keyword `is`:

{% highlight c# %}
var acct = new AccountRepository.Get(id);

if (acct is EmptyAccount) {
    //...
}

{% endhighlight %}

This approach has an advantage over a standard null check - it makes your intentions clear. You can encounter a null value for a lot of reasons. Checking for `EmptyAccount` is a check for a specific kind of *nothing*.

Of course, nothing in programming is free. Comparisons against a type are (slightly) slower than normal object comparisons, so let's explore another approach that uses a simple comparison check.

First, make `EmptyAccount` into a [singleton](http://csharpindepth.com/articles/general/singleton.aspx), and then expose the single instance of `EmptyAccount` through a static property on `Account:`

{% highlight c# %}
public class EmptyAccount : Account {
    private static EmptyAccount _instance = null;
    private static readonly object _lock = new object();

    private EmptyAccount() {
        //...
    }

    public static EmptyAccount Empty {
        get {
            lock (_lock) {
                if (_instance == null) {
                    _instance = new EmptyAccount();
                }

                return _instance;
            }
        }
    }

    //...
}

public class Account {
    public static Account Empty {
        get {
            return EmptyAccount.Empty;
        }
    }

    //...
}
{% endhighlight %}

Once this structure is established, returning an `EmptyAccount` is as easy as returning `Account.Empty`. As a bonus, accessing our null object is now consistent with other empty constants in the .NET framework.

Checking for the absence of a value is now straightforward:

{% highlight c# %}
var id = -1; //let's assume that there's no id "-1"
var acct = (new AccountRepository()).GetById(id);

return acct == AccountEmpty; // true
{% endhighlight %}

The default equality implementation in C# for `Object` is a reference comparison. Since there's only one instance of `EmptyAccount`, comparing two references to `Account.Empty` will always evaluate to `true`.

There is one potential problem lurking here - if the parent type overrides `.Equals`, comparisons to your empty singleton can become unreliable.

{% highlight c# %}
public class Account {
    //...

    public override bool Equals(Object obj) {
        return this.Name == ((Account)obj).Name;
        // let's agree to ignore the potental NRE here
    }

    public Account() {
        Name = String.Empty;
    }
}

return (new Account() == Account.Empty); // true!
{% endhighlight %}

It's probably best to avoid the singleton approach if you're working in an environment that routinely overrides equality for your types. You should probably also try to avoid environments that override equality, but that's just one man's opinion.

You might be thinking at this point that you can just override the equality comparison on your null type, perhaps by forcing a reference check:

{% highlight c# %}
public class EmptyAccount {
    //...

    public override bool Equals(Object obj) {
        return Object.ReferenceEquals(this, obj);
    }
}
{% endhighlight %}

This solution might seem good on the surface, but in practice it will introduce subtle, hard-to-reproduce bugs into your code, since the order of the arguments in an equality comparison suddenly matter:

{% highlight c# %}
var acct = new Account();

var hmm = acct.Equals(Account.Empty); // true
var uh_oh = Account.Empty.Equals(acct); // false
{% endhighlight %}

Seriously, don't do this. Down this path lies madness.

## Let's All Stop Returning Null Together

Now you know how to avoid returning null, and, of course, knowing is half the battle.

![LASERS](/assets/img/posts/2015-5-1-stop-returning-null/knowing-is-half-the-battle.jpg)

(The other half is lasers.)

What you do now is up to you, but remember this: every avoided NRE makes the world a better place.

You want to make the world a better place, right?