---
layout: post
title: "Better Open Source"
description: "How I make open source projects successful and how you can too."
tags: [code, community, open source]
image:
  feature: mozilla-is-my-dinosaur.jpg
share: true
---

I was listening to the [Cracked Podcast][] today, and they mentioned that
any personal successful enough that you know about also knows about you.
In other words: they realized they had to market themselves a certain way and
present themselves a certain way to have people pay attention to them. Read
the diaries of even a supposed misanthrope like Kurt Cobain and you see he was
incredibly concious of his band's behaviour and image--despite their motto
seemingly being one of "we don't give a damn".

Caring about the way your open source project looks to the world will make
all the difference when it comes to getting contributions from casual users
and not just people with a vested interest in the library.

## Where it all began

I've been working on [localForage][] for the past few months, and I've picked
up some new skills that I didn't expect to pick up. I figured working on the
library would teach me more about persistent storage in JavaScript, but it
turns out I've learned a lot about creating better open source projects, from
managing issues and people to making sure the library is as inviting to new
contributors as is possible. I don't think localForage is that amazing of a
concept, nor do I think it's so revolutionary that it took off simply because
it's The Most Amazing Thing Ever™. Rather, I think localForage took off
because it's an **inviting**, well-documented, and well-maintained open source
project that encourages **automated tests**, **community ownership**, and
discourages **nitpicking** and **insider knowledge**.

My friend [sole][] [pointed out a lot of things][sole-talk] [our team][]
could do to create better, more successful open source projects. In this
context, I think success is not only lots of users *of* our software, but
contributors *to* our software. The same things sole was advocating for were
clearly things that fostered [contributions][] to localForage.

So what can you do to write an open source project that will not only garner
people's interests, but also their contributions? Here are the things that have
worked for me, without fail:

### No Insider Knowledge

Document *everything*. This one sounds easy and obvious, but it's not as
simple as a README with a few rushed paragraphs. You need to make sure people
know what your library does without having to read all the documentation.
Resist the temptation to name or cleverly describe something you want people
to use. The tagline of your project (and it should have a tagline) is the
entry point for people along with its name--it's also your one-sentence
elevator pitch. localForage's use to simply advertise itself as:

 > Offline storage, improved.

While that might sound "cool", it fails to simpy tell a glancing developer if
they should use it, if it solves their problems, or even what it does. The
new tagline in the app is:

 > localForage is a JavaScript library that improves the offline experience
   of your web app by using asynchronous storage (via IndexedDB or WebSQL
   where available) with a simple, localStorage-like API.

Even that's a bit wordy, but it's **much** more informative.

If you're writing anything with a public API, maintain a changelog.

<!--
Cracked Podcast is either:
http://www.cracked.com/podcast/8-hidden-causes-behind-modern-historys-biggest-changes/
OR
http://www.cracked.com/podcast/pop-or-soda-why-region-more-important-than-you-think/

Check which one!
-->

[Cracked Podcast]: http://www.cracked.com/podcast/8-hidden-causes-behind-modern-historys-biggest-changes/
[contributions]: https://github.com/mozilla/localForage/graphs/contributors
[localForage]: http://mozilla.github.io/localForage/
[sole]: http://soledadpenades.com/
[sole-talk]: http://soledadpenades.com/files/t/our-projects/
[our team]: irc://irc.mozilla.org:6697/#apps
