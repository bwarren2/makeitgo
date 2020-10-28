---
title: "Vscode"
date: 2020-10-27T22:56:52-04:00
draft: false
summary: "My VSCode setup"
---

# My Setup

Here are the tools I use in my VSCode setup, in case they help someone else.

For context, I primarily write/deal with:

* Python (Django)
* Javascript (Vue)
* Docker files
* Cloudformation/AWS tech stuff
* Gherkin files
* Go
* Hugo (for this blog)
* JSON

# Code Navigation

## Bracket Pair Colorizer (1.0.61)

Useful for matching densely nexted brackets/parens by changing colors

## indent-rainbow (7.4.0)

Makes everything in the same alignment the same color of indentation, and marks in red things that are off-standard-indentation.

## Indenticator (0.7.0)

Highlights current indent level

## vscode-json (1.5.0)

For beautifying/uglifying JSON

# Code Formatting

## `black`

Write crappy Python code and transform it into good stuff

## Prettier (5.7.1)

Same idea, but for JS

# Lifestyle

## filesize (2.1.4)

Show current file size in toolbar

## Gherkin Table Formatter (1.0.3)

Hand formatting tables sucks, and this does it for you.

## EditorConfig for VS Code (0.15.1)

Support for the Editorconfig standard

# Language-specific

## Python (2020.9.large)

Required for good Python experience.

## Go (0.18.0)

Go snippets and macros.

## Djaniero (1.4.2)

Django snippets

## Vetur (0.28.0)

Tooling for Vue.

## ESLint (2.1.13)

JS Linting

## Hugo Snippets (0.4.1)

Convenient snippets for things like highlighting when writing in the Hugo blog.

## Docker (1.7.0)

For navigating all the container infra.

## Hadolint (0.3.0)

This one is _excellent_.  It lints your Dockerfiles for antipatterns and suggests the more optimal way to do it.  Write amateurish Dockerfiles and follow its advice to make good ones.

# AWS

## AWS Toolkit (1.15.0)

A really excellent addon.  Puts a heads-up view of your AWS stack into your editor.

## Cloudformation Linter (0.15.1)

Find problems in your Cloudformation yaml ASAP.
