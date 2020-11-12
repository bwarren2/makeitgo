---
title: "Twitter and Venmo Pt 2"
date: 2020-08-27T23:34:02-04:00
draft: false
tags: ['case_study', 'followup']
summary: "Twitter's architecture, and extending my Venmo interview."
---

The feedback on [Venmo]({{< ref "venmo.md" >}}) was
> "pretty OK!  Your interviewer would probably steer you to the feeds portion, as that is the most interesting.  Look up how twitter is architected and take inspiration from that".

So, a brief detour into Twitter.  This is cribbed from [High Scalability](http://highscalability.com/blog/2013/7/8/the-architecture-twitter-uses-to-deal-with-150m-active-users.html).

First key insight:
> "Users care about tweets, but the text of the tweet is almost irrelevant to most of Twitter's infrastructure."

Most of Twitter's infrastructure is about shuffling IDs around sharded storage.  I read through the linked piece and drew up my understanding of their architecture; I **strongly** recommend this exercise.  If you can't draw something, you don't understand it, and if you can draw it, you probably understand it.

![Twitter](/img/diagrams/Twitter.png)

Basically, Twitter is a timeline manufacturing service, and it is mostly an implementation detail what goes in the timeline/that "Tweets" are involved.

On Tweet, a POST is sent to the load balancer + HTTP response tier (presumably the load balancer is doubled, but I only drew one for simplicity).  That tier creates a tweet in the Tweet service that matches ID to content, I presume that the HTTP tier stuck the tweet in the tweeter's timeline, then the problem of propagation is passed off to Flock.  (I presume the HTTP tier 201's at this point, and the tweeter's request is complete.)

Flock is the social graph portion that knows who follows who; I presume it is partially implemented as a datastore of User => [Followers], so on tweet the system knows who to update.

However Flock works implementationally, it is responsible for doing a bunch of "push tweet into timeline" actions, that put the tweet ID in the timelines of people following the person who tweeted.  The lists are 800 tweets long.

When a user gets their timeline, they get to hit an in-memory list of 800 tweets, and the service hydrates the tweets off the tweet store.

Pretty clever, actually.

### Venmo Feeds

So how would Venmo feeds of friend transactions work?  The same way as above.  There was nothing special about Tweets; we were just using a list of Person=>Listeners and pushing IDs into a list to by hydrated later.

### Other things I learned

Having to translate a system report into a diagram is super useful.  It calls out the "basic molecules" of systems design, and makes you think about how to represent what you understand.  I want to start a Menagerie subseries on this blog of the core types of structures I see.  The duplication of Redis consistent hashing boxes above, for example, is intentional; they look the same because fundamentally they are the same.
