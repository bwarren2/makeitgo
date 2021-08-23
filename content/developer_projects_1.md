---
title: "Developer Projects, Pt 1"
date: 2021-08-22T22:36:34-04:00
draft: false
---

A friend of mine recently asked for a list of projects he could do to expand his skillset/try out other areas of development.  This seemed like a thing worth sharing, so here is the list!

# Projects to Expand Developer Horizons

## Static Site: Blog

It's useful to know how static sites work, and how to build and deploy them quickly, because they are simple yet powerful.  Need a marketing site for your new thing to gauge interest?  A dev blog?  A personal KPI tracker?  A static site has you covered.

### What To Build

Use Hugo and deploy a developer blog on Netlify.  Publish a few entries.  Chronicling your projects/progress with a dev blog is a great commitment mechanism, and a good way to share/archive what you learn!  Write in Markdown.  Pick a hugo theme you like.  Netlify has an excellent-by-default push-to-publish workflow.

### Skills Developed

 * Git/Version Control
 * Markdown 
 * Netlify Deployment

## Data Analysis: Postgres

### What To Build

Spin up a postgres instance on your local machine.  Interact with it with `psql`.  Make a database, download an interesting dataset (I love to use county-level voting data in presidential elections for this), and store it in some tables in your database.  Use `pandas` and Jupyter Notebooks to explore your data.  Use Plotly and Mapbox to make some pretty graphics.   As an example, see [here](https://whatactuallyhappenedin2016.com/).  Note: you can publish your findings on a static site!

### Skills Developed

 * Python
 * Pandas
 * Jupyter Notebooks
 * Data Visualization
 * Basic SQL

## Dynamic Site: Django

Static sites are great, but what about dynamic ones, where the same uri (like `/home/`) looks different to different users?  You need a web framework.

### What To Build

Set up [Django](https://www.djangoproject.com/).  Use Postgres for a backend.  Make some kind of basic user-focused flow, like a site that lets people upload pictures of their lunch, make friends with other users, and look at everyone else's food pics.  (Or just doing the Django tutorial and extending it is fine.)  Navigating between pages will cause full page reloads, but don't worry about that for now.  Deploy what you are building on Heroku; starter everything should be fine.  Use bootstrap for CSS.  Use Pyenv to manage your virtual environment.  Write unit and integration tests.

Note how within the Model system Django uses to model data, there is an abstraction placed over your storage.  You still need to know the nitty-gritty of how your storage works (thus the project above), but it's nice to not have to worry about implementation details for basic things/write in expressive language.

### Skills Developed
 * Web frameworks
 * HTML
 * MVC/MVT architecture
 * Data Modeling
 * Pyenv/package management

## Django Pt 2: REST

You made templates and rendered data!  So cool!  Now, we'll split up responsibilities to enable that no-page-reload flow from major sites you know.

### What To Build

Add the `django-rest-framework` package, do the tutorial in a new project, then add RESTful endpoints in your existing Django project.  Read all the DRF docs; they are excellent, and model best practices.  For reference, consider the [cdrf docs](https://www.cdrf.co/).  Use something like drf-yasg or drf-swagger to generate Swagger docs.  You want something looking like [this](https://petstore.swagger.io/).  Consider [`drf-nested-routers`](https://github.com/alanjds/drf-nested-routers) if you need them.


### Skills Developed

 * REST design
 * API documentation

## Vue Frontend

You have an API; great!  Now let's put a frontend on it.  Use [Vue](https://vuejs.org/).  It's accessible, functional, and has excellent documentation and community.  (Much like Django.  Can you see my preferences?)    

### What To Build

First, mock up what you are thinking of building in a tool like [Whimsical](https://whimsical.com/).  Then, implement your flow with Vue.  Use the APIs you made above, if possible.

### Skills Developed

 * Javascript
 * Frontend deve
 * UI mockups

## More Coming Soon!

There is much more to try!  For example:

 * Strongly typed languages
 * CLI tools
 * Cloud Deployments
 * NoSQL Datastores
 * Lambda/FaaS Systems
 * Distributed Systems