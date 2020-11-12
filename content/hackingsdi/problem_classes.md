---
title: "Problem Classification"
date: 2020-09-07T00:11:30-04:00
draft: true
summary: 'Or, ''being less bad is not the same as "good"'''
---

A friend gave me feedback on the intent of this blog, noting that SDIs are more useful than the traditional coding problems that are also fixtures of interviews. He observed:

> Honestly I think the system design interview questions are more important and less hazing than coding/algorithm questions. Those tend to be things that aren't close to anything I've seen most engineers do. But I think engineers do system design a lot, and it becomes increasingly important over time. I don't care too much about whether someone can implement a backtracking search algorithm neatly and correctly in 30 minutes, but I care a lot about whether someone can take a somewhat ambiguously defined problem, make it less ambiguous, and propose a reasonable solution that I believe they can implement. That said, a real design review will involve all kinds of writing and such that doesn't show up in an hour long interview. //shrugs//

which is true! System design is an interesting problem, and there is merit to being able to do it in a software engineering role. (Contrast with a programming role where the spec is given to you.)

However, I maintain that system design interviews are:

1. unnecessarily underresourced,
2. a fundamentally miscategorized problem.

The easiest contrast: there is literally a book called Cracking the Code Interview, and recommended by everyone who knows things about preparing for code interviews. This means that for code interviews, there is much higher likelihood that the candidate will have the resource directly recommended to them (still not great, relies on social networks) or randomly fall onto it by Googling. The problem of code interview prep is well resourced (CTCI, Leetcode), the resources are well publicized (popular knowledge/CTCI has 2500 Amazon reviews), and the approach is discoverable. (Am I the millionth person to write about CTCI? Quite possibly.)

_The SDI is not similarly well-resourced, well-socialized, or highly discoverable_.[^1] **Maybe** [this book](https://www.amazon.com/System-Design-Interview-insiders-Second/dp/B08CMF2CQF/ref=tmm_pap_swatch_0) is good? I don't know, but I bought it, so we'll find out. Where CTCI has 2500 reviews, at the time of writing this book has 35; it just doesn't have the same popular awareness.

But _even if_ there were such a prolific and useful resource, the nature of the systems design interview puts it into the wrong problem class.

# A homebrewed theory of problem classification

TLDR: there are different classes of problems that have similar sensitivities, some problems are immoral/counterproductive to put in certain classes.

We can carve out different problem classes by measuring how changing inputs changes the certainty that a person can solve the problem. I'll reason by degenerate, illustrative example before building up to more realistic examples.

If a job interviewer includes a mandatory step where the applicant must guess the number the interviewer is thinking of, the entire problem is low certainty of completion no matter what you change; spending more time prepping, getting insider knowledge of how the interview works, etc will not help you. If the interview involves a secret handshake, adding knowledge changes the problem from low certainty of success to high certainty of success. Similarly if the interview requires certain status or wealth markers, like going to an Ivy: add the wealth to increase certainty of success. In the GRE example from the [first post]({{< ref "failure.md" >}}), adding lots of time, some resources (a prep book), and knowledge that the prep book exists changed my outcome from low likelihood of success to high likelihood of success.

This is sounding like Calculus because it is. If `f` is the "certainty of success" function, we write `f(time, resources, knowledge, inclination) = certainty of success`, or `f(t,r,k,i) = c`. We can rewrite our examples above as:

"Guess a number" interview: `df/dt = df/dr = df/dk = df/di = 0, c is always low`. Changing any of the variables doesn't change the result.

"Secret handshake" interview: `df/dk = +++, c might range low without handshare, high with`. Changing the knowledge you have going in dramatically increases certainty of success.

"Go to Ivy" interview: `df/dr = +`. Depending on sensitivity, `c` might range medium to high.

GRE: `df/dt = +, df/dr = ++, df/dk = +++` with a bunch of cross terms. You need a lot more time than knowledge or resources, but you need them all. Based on my experience of the GRE, `c`

# Problem classification can vary by person

So why is this interesting? _For some problems these equations vary by person, and for some they don't_. For example, PhD-level theoretical mathematics is _**hard**_; everyone has positive coefficients on time, resources, knowledge, and inclination there, but some people have much larger coefficients. Picking the right person for that problem is important because the certainty of success equation varies greatly by candidate, rather than being fixed for the problem, and the fraction of sufficiently-good candidates in the pool is small: candidate selection matters and you aren't like to to nail it by chance.

By contrast, with the "guess a number"/"secret handshake"/"GRE" examples above, the success equation doesn't meaningfully vary by candidate. Especially when you are clued into spaced reinforcement strategies, the GRE becomes an exercise in time rather than talent.

# Wrong problem categorizations

TLDR: It is bad to have some problems in some categories. For example, a poll tax is voting sensitive to resources, and it is immoral (one kind of bad) to have the voting problem be sensitive to anything but a small amount of time. Other categorizations are also bad. I claim the SDI is one.

The "guess a number" interview is a silly example, but there are other members of its problem class. We can imagine a hiring panel that is so discriminatory it almost only accepts male candidates. For that panel, too, `df/dt = df/dr = df/dk = df/di = 0`, but certainty of success wildly varies by candidate. Unless maleness is a bona fide _requirement_ of the job[^2], this is immoral.

Other wrong categorizations tha "immoral" exist too. What if a particular hiring process costs $2-3K per applicant processed, but has a low success rate, and the problem is resource-&-knowledge-sensitive s.t. spending $50 dramatically increases odds of success? It's not _immoral_ to have the resource sensitivity in this problem, but it is _short-sighted, uneconomical, and self-defeating_ to not address the resource sensitivity. Maybe after clearing a certain hurdle, the hiring company just eats the \$50 to buy the candidate the resource and tells them to use it, or (less optimally) says "if you want to succeed you should buy this thing". Either way, it is counter to the hiring company's purposes to just muddle through this bad system without addressing the resource sensitivity.

(It is possible that the information on how to pass is propagated through social channels s.t. people of the same demographics tend to pass. I leave it to the reader to decide if this is immoral, or uneconomical, or some other bad.)

[^1]: Cracking the Code Interview, 6th edition has a chapter on SDI, but it is 9 pages.
[^1]: which is very rarely true!
