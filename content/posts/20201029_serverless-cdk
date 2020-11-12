---
title: "Serverless Cdk"
date: 2020-10-29T00:03:58-04:00
draft: false
summary: "Up and running with CDK and Serverless!"
---

This is just a stub of a post, but: I got some of the CDK Serverless setup done!

It is just a python implementation of the tutorial project [here](https://docs.aws.amazon.com/cdk/latest/guide/serverless_example.html).

Things learned:

Adding the local env installs for autocompletes: very important. Looks like

    "python.autoComplete.extraPaths": [
        "${workspaceFolder}/.env/lib/python3.7/site-packages"
    ]
in .vscode.

A new cloudwatch group is spawned per deployment, ie you need to cycle through groups to keep up with changes. You do get print debugging that way, though.

Wish I knew of a better way to test; might just need to declare some mock objects. It feels like there should be a library for this?

CDK is super expressive.  It is much more intuitive to write than Cloudformation and knowing the ins and outs of 10 different services.
