---
layout: post
title:  "An Experiment in Vanity, Part One: Introducing the Dashboard"
photo: /assets/img/headers/apple-stuff.jpg
photographer-url: https://unsplash.com/firmbee
photographer: William Iven
location: Gainesville, Florida
excerpt: Wherein I embark on a new learning adventure and, while stroking my ego, very quickly get in over my head.
---

I spend a lot of time thinking about stagnation.

Technical knowledge has a shelf life. Eventually, it expires. Andrew Hunt and Dave Thomas discuss this in _The Pragmatic Programmer_:

> Your knowledge becomes out of date as new techniques, languages, and environments are developed. Changing market forces may render your experience obsolete or irrelevant. Given the speed at which Web-years fly by, this can happen pretty quickly.

I've spend the last nine years writing C# & Javascript professionally. I've gotten pretty good at it, and I work hard to stay on top of developments in the .NET space. I like to think that my code has kept up pretty well with the changing nature of .NET development.

Don't get me wrong &mdash; C# is a great language, and I like it a lot. I've even ventured out into the world of [.NET open source](https://github.com/jtompkins/Haberdasher)! There's a lot of room to grow, and the upcoming changes in C# 6.0 are really exciting. The language is getting more dynamic and more expressive with every release. I'm looking forward to writing C# code for many years to come, but today, C# is the only language I really know well. Nine years is a long time to stick to working in one language. What else am I missing because I'm not experimenting with other technologies?

*(Javascript doesn't count. Everyone knows Javascript.)*

There's one way to find out. It's time to try something new.

## A Personal Dashboard

Learning a new set of technologies is easier said than done. In my experience, the best way to learn something new is to just try it out, so I went looking for a side project.

A couple of days into the search, I found this Tweet:

<blockquote class="twitter-tweet" lang="en"><p>It&#39;s underestimated how much value you can provide simply by intelligently gluing together existing APIs and services.</p>&mdash; Einar Vollset (@einarvollset) <a href="https://twitter.com/einarvollset/status/584052437549326336">April 3, 2015</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

Hmm.

A few weeks ago, I started wearing a FitBit and tracking the number of steps I take during the day. Does FitBit have an API where I can access that data? As it turns out, [they do.](https://wiki.fitbit.com/display/API/Fitbit+API;jsessionid=4CEE021FDC7D30745FD5FE21CF0451AE)

FitBit data is easy to visualize, so putting it into a chart seems like an obvious option. [I'm not the first person to figure this out.](http://blog.soff.es/fat/)

Okay. Let's build a dashboard.

I'm passively pushing data into a few other places, too - commits to GitHub, calorie information into LifeSum, articles to Instapaper. Once I've got the basics down, I can incorporate that data into my dashboard, too.

## Dashboards are a UI Playground

*(With [apologies to John Gruber](http://daringfireball.net/2009/04/twitter_clients_playground))*

Once I decided on a personal dashboard, I started looking around for inspiration. As usual, the internet is a rich source of prior art: dashboards crop up everywhere once you start looking for them.

Examples range from the science fictional:

![The Avengers Helicarrier](/assets/img/posts/2015-4-4-an-experiment-in-vanity/Avengers_Fury_Monitor_Screen_Graphics_slapcomp_jayse_hansen.jpg)

*Image credit: [Jayse Hansen](http://jayse.tv/v2/?portfolio=avengers-helicarrier-glass-screens)*{: class="caption" }

...to the realistic:

![Panic Status Board](/assets/img/posts/2015-4-4-an-experiment-in-vanity/statusboard.jpg)

*Image credit: [Panic](http://www.panic.com/blog/the-panic-status-board/)*{: class="caption" }

Of course, the canonical example of excellent personal analytics has to be [Nicholas Feltron](http://feltron.com/):

![Feltron Annual Report I](/assets/img/posts/2015-4-4-an-experiment-in-vanity/FAR12_04.jpg)

![Feltron Annual Report II](/assets/img/posts/2015-4-4-an-experiment-in-vanity/FAR12_08.jpg)

*Image credit: [Nick Feltron](http://feltron.com/FAR12.html)*{: class="caption" }

Do I expect my project to look as good as that? No, I don't. Do I expect to end up with something in the same zip code?

I hope so.

## Next Steps

I've got a destination in mind, and I have a vague idea of what the end result should look like. In my next post, I'll figure out what I'm going to use to build my dashboard.