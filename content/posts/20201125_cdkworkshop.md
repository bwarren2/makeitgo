---
title: "Visualizing a stack with CDK Workshop"
date: 2020-11-25T15:43:27-05:00
draft: false
summary: "AWS CDK is _lovely_"
---

AWS CDK is excellent, and might be my favorite AWS product ever.  I am using it to better understand other AWS services and how they are supposed to be set up, with the help of fancy graphics.

A sample graphic is this Lambda + Role setup:

![Lambda](/img/cdk_viz/step_1_lambda.png)

## Starting with Cloudformation

It replaces/upgrades Cloudformation (Cfn), which always felt very clunky to me.  Extremely configurable, Cfn makes anything possible, but almost all the configurations are wrong: maybe they have too-broad permissions, or are forgetting a key resource, etc.  Expressiveness is not Cfn's forte.  Instead of "I want a lambda behind an APIGateway endpoint", you need to know about (and create) every resource each of those services requires to integrate.  I hope you know what the best-practice setup of all those resources are, because nothing is going to tell you.

## Enter CDK

CDK actually lets you write "I want a lambda behind an APIGateway endpoint", and it _just works_.  Under the hood, CDK generates Cloudformation, so you are not technically locked in while CDK is under active development.

## Try CDK out

[The CDK Workshop](https://cdkworkshop.com/) is an excellent intro to get your hands dirty.  When you first start the sample app, your directory will look like
```
.
├── README.md
├── app.py
├── cdk.json
├── cdkwork2
│   ├── __init__.py
│   ├── cdkwork2.egg-info
│   │   ├── PKG-INFO
│   │   ├── SOURCES.txt
│   │   ├── dependency_links.txt
│   │   ├── requires.txt
│   │   └── top_level.txt
│   └── cdkwork2_stack.py
├── requirements.txt
├── setup.py
├── source.bat
└── tests
    ├── __init__.py
    └── unit
        ├── __init__.py
        └── test_cdkwork2_stack.py
```
and you get a Queue subscribed to a Topic.  The relevant Python is

{{< highlight python>}}
# app.py

class Cdkwork2Stack(core.Stack):

    def __init__(self, scope: core.Construct, construct_id: str, **kwargs) -> None:
        super().__init__(scope, construct_id, **kwargs)

        queue = sqs.Queue(
            self, "Cdkwork2Queue",
            visibility_timeout=core.Duration.seconds(300),
        )

        topic = sns.Topic(
            self, "Cdkwork2Topic"
        )

        topic.add_subscription(subs.SqsSubscription(queue))
{{< /highlight >}}

It looks like this:

![Sample app](/img/cdk_viz/sample_app.png)

Nodes are Resources in the generated Cloudformation, and arrows point to dependencies of those Resources.

You can see the generated YAML with `cdk synth`

## Evolving a setup

To really drive home how much work CDK is doing for us, watch what happens to go from "a Lambda" to "a Lambda behind APIGateway"

### Just a Lambda

For the first step in the workshop, we change our stack to have a single Lambda with a handler.  Our project adds a lambda directory with a file:

```
.
...
├── lambda
│   └── hello.py
...
```

And the stack becomes

{{< highlight python >}}
class Cdkwork2Stack(core.Stack):
    def __init__(self, scope: core.Construct, construct_id: str, **kwargs) -> None:
        super().__init__(scope, construct_id, **kwargs)

        my_lambda = _lambda.Function(
            self,
            "HelloHandler",
            runtime=_lambda.Runtime.PYTHON_3_7,
            code=_lambda.Code.asset("lambda"),
            handler="hello.handler",
        )
{{< /highlight >}}

And the generated Cloudformation only has two resources:

![Lambda](/img/cdk_viz/step_1_lambda.png)

### Add APIGateway

Exposing the Lambda to the internet means putting it behind APIGateway.  In CDK, that is easy:

{{<highlight python>}}
class Cdkwork2Stack(core.Stack):
    def __init__(self, scope: core.Construct, construct_id: str, **kwargs) -> None:
        super().__init__(scope, construct_id, **kwargs)

        my_lambda = _lambda.Function(
            self,
            "HelloHandler",
            runtime=_lambda.Runtime.PYTHON_3_7,
            code=_lambda.Code.asset("lambda"),
            handler="hello.handler",
        )

        apigw.LambdaRestApi(
            self,
            "Endpoint",
            handler=my_lambda,
        )
{{</highlight>}}

But in Cloudformation, it's _much_ more complex.

![APIGateway](/img/cdk_viz/step_2_apigateway.png)

Our original Lambda setup is in the top right; you can see that the hashes match.  Everything else came from those three lines of LambdaRestApi in the cdk app.  You can see the interactive version of this graph [here](http://www.ndexbio.org/viewer/networks/1f5303af-2f5d-11eb-9e72-0ac135e8bacf).

## Learning from the graph

"API with Lambda backing" is a clear and comprehensible idea; implementing it from first principles in Cfn is harder because you need to know what all the AWS nouns are.  CDK + graphics let us reason backwards from known good setups, and ask what all the pieces are for.

I did not know what an [APIGateway Account](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-apigateway-account.html) was before doing this research.  From what I read, it appears to be a magic resource expected by APIGateway to reference an ARN of an IAM Role to authorize APIGW to write logs to Cloudwatch.  It's weird that there is an implicit ordering; if you have never created an APIGW resource before, you need to make an Account depend on the first one you make via template.

Based on the generated Cloudformation from CDK (and the [CDK Examples](https://github.com/aws-samples/aws-cdk-examples) repo), I expect CDK to make it to be _much_ easier to learn how to build the things I want.  Even if I don't stick with CDK long term, getting decipherable best-practice examples is a huge win.
