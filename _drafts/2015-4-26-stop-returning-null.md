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

Checking for null makes good code ugly. Unfortunately, *not* checking for null is worse.

For years, I didn't check for null. At first, I simply didn't know better. Later, I knew better (or at least, I *thought* I knew better), but still didn't check for null sometimes because I didn't like how it made my code look.

I'd often avoid checking for null in cases where a null value "couldn't happen". This is funny, because null values *always* happen, usually in the places where you were most sure they couldn't.

You can picture the results. My code was full of Null Reference Exceptions. Littered with them.

My boss at the time eventually got so frustrated by my crash-happy code and my unsatisfactory explainations that he made me a poster to help me remember to check for null:

![Programming](/assets/img/posts/2015-4-26-stop-returning-null/programming-poster.jpg)

Lesson learned.

Today, potential null references are one of the first things I look for during code reviews. You would be stunned to learn how often they show up, even among experienced programmers who should probably know better.

Why do we avoid checking for null? Because it makes our code look worse.

!!why do we have to check for null everywhere? what's the consequences?

Programming is, at its heart, a form of art. The best programmers are artists, driven to make work that reflects their innate good taste. Littering your code with null checks violates their sensibilities. What's worse, null checks are viral - once they appear in one place in your code, they start to appear *everywhere*.

Imagine you've created the following type:

{% highlight c# %}
class Contact {
    public string Name { get; set;}
    public string Email { get; set;}
}
{% endhighlight %}

Pretty much as simple as it gets. You want to retrieve some `Contact`s from the database. How do you do it? If you've been at this for a while, you might extract data retrieval out into some sort of repository:

{% highlight c# %}
class ContactRepository {
    public IEnumerable<Contact> GetAll() {}
}
{% endhighlight %}

Your repository returns a list of `Contact`s. Let's say you want to display a list of email addresses. You might write code like this:

{% highlight c# %}
var repo = new ContactRepository();

var emails = repo.GetAll()
                 .Select(c => c.Email);

for (var email in emails)
    Console.WriteLine(email);

{% endhighlight %}

Pretty straightforward, right? What happens if one of the contacts doesn't have an email address? Your code crashes.

Maybe you've been around the block a few times, and you saw the potential NRE right away. You know better than to write code like that, right?

{% highlight c# %}
var repo = new ContactRepository();

var emails = repo.GetAll()
                 .Select(c => c.Email);

for (var email in emails) {
    if (!String.IsNullOrEmpty(email))
        Console.WriteLine(email);
}

{% endhighlight %}

There, that's better. Now just remember to do that every time you pull a list of email addresses out of your database, and you'll be safe, right? Of course you'll remember! This isn't your first rodeo.

!!note expand here on why a developer does this

What about the next developer to use this code? What about the developer after that?

The fact is, any code that returns null is dropping a time bomb into your code base. The question isn't *if* it'll go off - the question is *when*.

!!note in here somewhere about how putting null checks into your code *feels* like it's better code, but actually isn't.

### Working around return values that aren't Object

### An introduction to the Null Object Pattern

There's an alternative to littering your code with null checks, or just sticking your head in the sand and hoping for the best. It's called the **[Null Object Pattern](http://en.wikipedia.org/wiki/Null_Object_pattern)**, and it's about to change your life for the better.

!!note here about the relative obscurity of this pattern in C# code and how it's more common in other communities like RUby

As patterns go, this one is relatively simple. What we want is to define a *null object* - a version of a known type that defines sensible defaults and - critically - responds to the same API as the existing type.

It's easy to define a type like this in C#:

{% highlight c# %}

public class EmptyContact : Contact {
    public EmptyContact() {
        Name = String.Empty;
        Email = String.Empty;
    }
}

{% endhighlight %}

The exact name of the type is up to you, but I like to stick with the "Empty" convention established by `String.Empty` and `Enumerable.Empty<T>()`.

`EmptyContact` extends `Contact`, so it responds to the same API and can be returned from and consumed by any method that expects a `Contact`.

Now we're back to our original database code:

{% highlight c# %}
var repo = new ContactRepository();

var emails = repo.GetAll()
                 .Select(c => c.Email);

for (var email in emails)
    Console.WriteLine(email);

{% endhighlight %}

### Evidence of Absence

What if you need to detect the situation where nothing was returned? That's the most common use of a `null` return value and there will be times when you need to know that nothing came back.

The easiest way is to examine the type of your variable:

{% highlight c# %}
var contact = new EmptyContact();

if (contact is EmptyContact) {
    //handle this case here
}

{% endhighlight %}

As a pattern, this has a an advantage over a basic null check: *it makes your intentions* clear. You're not checking for null to avoid an NRE, you're checking to see if *nothing came back* from your data source.

Nothing in programming is free, though, and there's a disadvantage here, too: checking the type at runtime with *is* is slower than a simple comparison check.

We can get around this by making a default `EmptyContact` we can compare values against. The easiest way to do this is to make `EmptyContact` a `Singleton`:

{% highlight c# %}
//singleton code here
{% endhighlight %}

Once you've ensured that there will only ever be one `EmptyContact`, simply return the singleton whenever you need to. For consistency with other objects in the .NET framework, we can create a `.Empty` static property on `Contact` for easy access to the `EmptyContact` singleton:

{% highlight c# %}
//empty property on contact
{% endhighlight %}

Once this structure is established, code that consumes your `Contact` type doesn't necessarily have to know about `EmptyContact` any more. All references can come through the `Contact.Empty` accessor.

{% highlight c# %}
//sample equals checking code here
{% endhighlight %}

!!note here explaining the default equality check in C# for objects and how all of this works

There's a trap lurking here, though - what if `Contact` overrides `.Equals` and `==`?

{% highlight c# %}
//equality overrides here
{% endhighlight %}

In the example above, the equality operator has been overriden to check the value of `.Name`. Contacts with the same name are considered the same contact.

In this case, any `Contact` with an empty `Name` property would be considered equal to `EmptyContact`, which obviously presents a problem.

If you're writing code with objects that tend to override `Equals`, you should probably stick to `is` when checking for `EmptyContact`.

Before we move on: you might be thinking of trying to get around this problem by overriding `Equals` on `EmptyContact`:

{% highlight c# %}
//overriding equals on emptycontact
{% endhighlight %}

This is one of those solutions that seems good on the surface, but in reality can introduce subtle, hard-to-reproduce bugs into your code. In this case, overriding `Equals` suddenly makes the ordering of the arguments to `==` matter:

{% highlight c# %}

var contact = new Contact();
var empty = Contact.Empty();

Console.WriteLine(empty == contact); //false
Console.WriteLine(contact == empty); //true!
{% endhighlight %}

Don't do this.

Using the Null Object Pattern can make your code safer, simpler, and more intentional. Use it often!
