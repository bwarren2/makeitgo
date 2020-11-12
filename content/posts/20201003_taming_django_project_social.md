---
title: "Taming A Wild Django Project, Part 2"
date: 2020-10-03T16:14:48-04:00
draft: false
summary: "Maneuvering in social space to improve the project"
---

As described [last time]({{< ref "20201003_taming_django_project" >}}), there are two major solution areas for big wins when taming a wild django project. This time, we discuss the social side.

Code is just one part of the business system. It is the most visible, obvious, and measurable, but it's a byproduct, a symptom of the people and structures that produced it. When a Django project is wild, some of the problems stem from lack of knowledge or time in the manufacturing, but part of it probably stems from institutional forces: who has power or authority, and the patterns of interaction that form the workplace.[^power]

> "Happy families are all alike; every unhappy family is unhappy in its own way." - Tolstoy

We can imagine various stories that all arrive at the same place:

1. As the project grew, failures in handoffs begat the scar tissue of Process to CYA. This slowed down the system, resulting in more division of the work to speed things up, increasing handoffs and GOTO 1.
2. What started as a small project grew into a larger team, and the reporting structure organized around chains of command up to the original founders instead of centralizing in the team. This let large ideological holy wars run rampant as developers in the same project had orthogonal requirements to their bosses, who themselves sat on different sides of the wars.
3. Lacking knowledge of practice and history, decision-makers thrust into a management position reinvent Waterfall. Surely, it will work better this time. It makes so much sense! The bullwhip effect is not real.

> [Traumatropism](https://www.merriam-webster.com/dictionary/traumatropism): a modification of the orientation of an organ (as a plant root) as a result of wounding.[^traumatropism]

These dysfunctions in the between-humans parts of the business (which is most of the business!) ripple into decision about planning and scope, and that shows up in code. Here's how to deal with it.

[^power]: After all, who made those resourcing calls? Who set those timelines? Who decided to build the project _this_ way? All those things come from people, and are tainted by their dysfunctions.
[^traumatropism]: [Kevlin talks about this phenomenon as well](https://youtu.be/-nWhH-4wWBU?t=2024).
