---
title: "Taming A Wild Django Project, Part 1"
date: 2020-10-03T16:08:48-04:00
draft: false
summary: "How I whipped a project into shape on the code side"
---

> “You go to market with the code you have, not the code you might want or wish to have at a later time.” - Donald Rumsfeld, probably

I was hired on a Django codebase that was clearly written by people learning both Django and Python as they went. This is part of how I worked to whip the project into shape.

There are two big categories of solution for major wins:

1.  fixing lack of knowledge;
1.  fixing social structures.

The first category is simpler and more fun for tech people, but often less impactful. Effort in the second category is often required to buy credibility for change in the first. For now we'll talk about the tech side. Here's what worked for me.

# Use `django_extensions`, get model diagrams

> A picture is worth a thousand words.

Big, unruly code projects are often hard to understand and crufty. Make people's lives easier by exposing data relationships. With [`graph_models`](https://django-extensions.readthedocs.io/en/latest/graph_models.html?highlight=graph_models), you can share what the actual tables and relationships are with nontechnical/nonbackend people, and keep things at least partially up to date automatically. Being able to point out to your PM or Director where things fit in the data model, or why their proposed change is easy/hard, makes things clearer to them. Rely more on truth and less on trust.

# Use `sphinx` to save time

Unruly projects often don't have good documentation, because they need to devote so much time to managing a difficult codebase. Documentation is always important but never urgent. Resist the trap of not documenting and thereby ensuring more time wasting later, by documenting your own work in Sphinx as you go. Lie about how long tickets will take, if you have to, to get the extra few hours to write good docs. Add the doc to github/gitlab pages in your CI/CD pipeline, and in this case it is better to ask forgiveness than permission. There are several [kinds of documentation](https://en.wikipedia.org/wiki/Software_documentation). At least write the architecture and technical commentary. What information would a person want from the senior, experienced developer on this part of the code if they had 5-10 minutes to get the download from them? Write that down.

# Use `django-rest-framework` for Model-oriented architecture

DRF is super powerful, but some developers might not complete the tutorial and develop along the paved roads the framework lays out. Time-crunched developers especially might come to the library with preconceptions of how APIs work and poach the things that match their expectations instead of challenge them. Use ModelViewSets, ModelSerializers, permissions, and RESTful practices. Don't override HTTP method handlers unless you have a really, really good reason (and even if you think you do, you probably don't).

# Use `drf-yasg` for autodoc

When you use good DRF practice, you can add one library and a CI/CD job to get excellent, Stripe-style automatic API doc _for free_. All that time that used to be sunk into developers asking you personally how things work is gone, as is the time lost to confusion about what something does. Do this part well, and you can write code, it makes doc, and people never ask you trivial questions about it again; you get to reinvest _all that time_. Annotate DRF actions with different potential response values with `swagger_auto_schema`, and set examples with `examples = <dict>` in your ModelSerializer class Meta. Use swagger2openapi + redoc-cli to get fancy Redoc.

# `black` your codebase...

I asked a very senior developer I respected what his major coding breakthroughs were, and this one stuck out: "I use prettier or other autoformatters so I can write shitty code and have it turned into good code". All the cycles you spend on interpreting code that is unevenly or unusually formatted are wasted; you can spend them better and get more done. Automate line breaks, spacing, and other typesetting administrivia by letting [`black`](https://black.readthedocs.io/) format your code, with one exception.

# ... but also `isort` your imports

Imports are one place where `black` might make your code less readable if you have already been coding for a while with bad style. If you have a bunch of `from foo import (A, B, C, D, E, ..., AA, AB, ...)` (it happens!) then you might add several screens of just imports per file. So how can you make imports the province of `isort` and everything else `black`'s domain? Well, isort lets you:

1. add comments ahead of named sections of imports;
2. add a specific import to every python file;
3. remove a named import from every python file;
4. put imports in order by section

So, in your `.isort.cfg` (or whatever you are using) use a `black` profile and whatever sections you would normally use, but add FIRST and LAST sections and

{{< highlight toml "linenos=table" >}}

    known_first=first
    known_first=last
    import_heading_last = fmt: on
    import_heading_first = fmt: off

{{< / highlight >}}

Then add all the import blocks:

{{< highlight bash "linenos=table" >}}

    isort -a "from first import sentinel" *.py
    isort -a "from first import sentinel" */*.py
    ... # Just make sure to get every .py you care about.

    isort -a "from last import sentinel" *.py
    isort -a "from last import sentinel" */*.py
    ... # Just make sure to get every .py you care about.

{{< / highlight >}}

Then remove all the actual imports:

{{< highlight bash "linenos=table" >}}

    for file in $(find -type f -name "*.py"); do echo $file; sed -i "/from first import sentinel/d" $file; done
    for file in $(find -type f -name "\*.py"); do echo $file; sed -i "/from last import sentinel/d" $file; done

{{< / highlight >}}

Then delete Any null blocks shaped like:

{{< highlight python "linenos=table" >}}

    # fmt: off

    # fmt: on

{{< / highlight >}}

Boom! You just wrapped all of your isort-visible non-nested imports in "black don't format" blocks. Set up your imports ordering to your liking and don't worry about about juggling unwieldly import rules again; isort will remove duplicates, merge similar imports, and order them for you.

# Lint your imports with `import-linter`

Especially if your code is oddly factored, you might want guardrails to avoid the path toward circular imports. Use [`import-linter`](https://github.com/seddonym/import-linter) to specify contracts your code must follow on what modules can import from what other modules, perhaps with exceptions for existing debt you write tickets to pay down. This is also a good one for building social capital with architects that want reinforcement to their product. Drop it into CI/CD and give more structure to floppy code.

# Find dead code with `pyflakes`

Mostly I am concerned with removing dead imports here, because that is the lowest hanging fruit to simplify things. Once you have categorically fixed a class of linting error, you can add that lint check to your CI/CD (or require devs to turn on `pyflakes`/`flake8` in their editors!) and never have this kind of problem again.

# Remove dead imports with `autoflake`

Looking for a good solution to remove those dead imports? [`autoflake`](https://github.com/myint/autoflake) will remove standard library stuff by default, but you can progressively rerun for all your other imports to remove all the other dead weight.

# `git-bisect` is your friend!

If you don't have tests in a large unwieldy project, then first and foremost you need to fix that. Once you have them, you can save some sanity doing these upgrades with [`git-bisect`](https://git-scm.com/docs/git-bisect). Given a command that returns success or failure, `git-bisect` can find the exact commit that caused the command to go from success to failure, ie it can find the commit that broke your tests. Did one of those imports we removed before actually have side effects that the system secretly relied on? Make changes as small as possible (commits are cheap! Do one for every file or line of a big removal if you must) and bisect your way to fixing your tests.

# The crazy stuff: `libCST`

Most of the wins so far have been easy stuff. Run some tools, add some rules, get some free wins that come with better style. [`libCST`](https://github.com/Instagram/LibCST) is the tool of choice for custom, complex, powerful changes to your codebase. Developed by Instagram to manage their enormous Django project, they have a lovely writeup of their use case [here](https://instagram-engineering.com/static-analysis-at-scale-an-instagram-story-8f498ab71a0c). The import problem from [last time]({{< ref "python_asts.md" >}}) is fairly trivial with `libCST`.

# Aaaand this post is too long

Next time, we deal with the harder but more impactful side of taming a wild Django project: the social side.
