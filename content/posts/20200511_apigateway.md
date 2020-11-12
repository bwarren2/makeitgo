---
title: AWS Lambda-Backed APIGateway w/ Cognito Authorizer
summary: A Hello-World Auth'd Setup
date: 2020-04-22 21:16
tags: aws, lambda, apigateway, cognito, howto
draft: false
---

# Growing Into AWS

My near-term goal is to grow into using AWS more, and [this project](https://github.com/bwarren2/sam-cognito) was a step in the right direction.  The objective was to get a basic hello-world endpoint with:

 * API Gateway in front
 * Lambda serving a hello-world response
 * Cognito doing auth

Simple, right?  Just make an endpoint accessible to the world, provide a basic response to know that it works, and limit who can interact with it.  This provides the scaffold for building more; add more routes backed by more functions, make the functions hit a datastore like Dynamo DB, and suddenly I have an arbitrarily-scalable CRUD endpoint.  Seems cool.  So how do we get there?

# Organizing Concept: AWS Is A Platypus

Each time I come back to AWS, I am struck by the total disharmony of design across components.  SAM lets you use a CLI to spin up a stack, but you need to use Cloudformation to take it down.  SAM appears to be a superset of Cloudformation, but CF is also supported and sometimes preferred??  In the AWS CLI, there is the `s3` set of commands, _and also_ the `s3api` set of commands, with substantial overlap.

    "There should be one-- and preferably only one --obvious way to do it."
    -- Source: `python -c "import this" | tail -n 7  | head -n 1`

It is very much like a platypus: a weird design that might seem familiar locally, but definitely isn't globally, and only really makes sense when you see all the pieces together.  So this is a very small setup that still does something nontrivial.

# Starting Small

## From-Zero Setup

[Make an AWS Account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/) if you don't have one.

[Make an admin user to work with](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started.html).

If you're confused about IAM, [try this](https://start.jcolemorrison.com/aws-iam-policies-in-a-nutshell/).

## Install CLIs (because of course you need two)

Get the [SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html).

Also [get the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html).

## Suggested Tooling

I wasted half an hour debugging a Cloudformation template problem before I installed a linter that pinpointed the problem for me.  Don't be like me: use `kddejong.vscode-cfn-lint` in the VSCode extensions from the start.  If you forget a dash and accidentally try to give a dict to an array property, this will tell you.

# The Steps

There are stages to this work:

1. Do the SAM hello-world example as a base
1. Add on the Cognito resources
1. Configure Cognito hosted auth
1. Make a user and get a login token
1. Test the token in API Gateway
1. Use it


## Do the SAM hello-world example

From [here](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-getting-started-hello-world.html).

### High level paraphrase of the article

`sam init` to start a project.  I use python 3.8, do what works for you.  You should have a folder structure kinda like this.  You won't have templategood.yaml; that's my own addition.

<img src="images/20200511/project template.png" alt="Basic project template" style="max-height:300px"/>

Install local testing requirements locally with `pip install pytest pytest-mock --user`.  Try running them with `pytest`, which should work under a normal default setup, but doesn't for me; I need to assert the PYTHONPATH like so:

<img src="images/20200511/running_tests.png" alt="Running pytest" style="max-height:300px"/>

OK!  You have something worth deploying.  Bundle up your code with

`sam build`

and deploy it with

`sam deploy --guided`

Almost all the defaults are correct.  My first time I needed to agree to create roles, IIRC, but don't have that problem on subsequent runs.

You should be able to watch the deploy complete, then see it in your Cloudwatch stack list with `aws cloudformation list-stacks`.

Try hitting it at the invocation URL the deployment gave you.  Remember the URL format should be `https://<host>/<environment>/hello` for the default template.  If you skip the trailing `hello`, it will fail and you will be confused.  This hit should succeed.

Great!  Now tear it down.  (The command is `aws cloudformation delete-stack --stack-name sam-app --region region`.  Note how we have to use a different CLI for this lifecycle task.)

## Adding Cognito Resources

Cognito has two kinds of auth under the same service, basically: User pools and Identity pools.  User pools are similar to the standard signup/login identity flow most places use.  Identity pools map the person logging in _to an IAM role_, giving you permissions management at the IAM level.  For now, just think about user pools; we're adding one of those.

In the `Resources` block of your template.yaml, add something like this:

```
  CognitoUserPool:
    Type: AWS::Cognito::UserPool
    Properties:
      UserPoolName: swimming-pool
      Policies:
        PasswordPolicy:
          MinimumLength: 8
      UsernameAttributes:
        - email
      Schema:
        - AttributeDataType: String
          Name: email
          Required: false
```

to declare the pool.  You can use whatever pool name you want.  We also need a user pool client, so add something like:

```
  MyCognitoUserPoolClient:
    Type: AWS::Cognito::UserPoolClient
    Properties:
      UserPoolId: !Ref CognitoUserPool
      ClientName: user-pool-client
      GenerateSecret: false
```

note the `!Ref`; it lets us refer to some other thing in our Cloudformation template to populate that variable, rather than hardcode a value.  This User Pool Client will be configured to use the AWS hosted login flow.

We also need to assign the Cognito User Pool as an authorizer on the endpoint we are building.  In the basic hello-world example, this API got built by implication of our work.  Here, we need to include it so we can add those `Authorizer` blocks.

```
  MyAPI:
    Type: AWS::Serverless::Api
    Properties:
      StageName: Prod
      Auth:
        DefaultAuthorizer: CognitoAuthorizer
        Authorizers:
          CognitoAuthorizer:
            UserPoolArn: !GetAtt CognitoUserPool.Arn
```

And finally, wire the function up to the API:

```
  HelloWorldFunction:
    Type: AWS::Serverless::Function # More info about Function Resource: https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md#awsserverlessfunction
    Properties:
      CodeUri: hello_world/
      Handler: app.lambda_handler
      Runtime: python3.8
      Events:
        HelloWorld:
          Type: Api # More info about API Event Source: https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md#api
          Properties:
            RestApiId: !Ref MyAPI
            Path: /hello
            Method: get
```

This block should look very similar to the template you started with.  Note the `RestApiId` property we added.

If you build and deploy this, it should spin up your Cognito resources as well, so do that.

`sam build`
`sam deploy`

## Adding Cognito Hosted Auth

Go to `https://console.aws.amazon.com/cognito` and `Manage User Pools`. Pick the pool you made, then go to `App Integration > Domain Name`.  This is where we will pick a domain to host the auth Amazon will hold for us.  Choose anything, but you won't be allowed to choose something someone already chose.

Next go to `App Integration > App Client Settings`.  Make it look like this:

<img src="images/20200511/integration.png" alt="Auth integration" style="max-height:300px"/>

Those two things together should unlock the Launch Hosted UI link at the bottom of the page.  Open that in a new tab, then go to your `General Settings > Users and Groups`.

## Create a User

Just fill in something basic for now as a test user.  Remember that the phone format in the US needs to be `+15555555555`; AWS will yell at you if you forget the `+1`.

## Login With That User

In the Hosted Auth tab you opened, login with that user.  The redirect URL will look like:

```
<Your Redirect URL>#id_token=areallyreallylongstringofnumbersandletters&access_token=adifferentreallyreallylongstringofnumbersandletters&expires_in=3600&token_type=Bearer
```

That ID token is what you want.  Validate it at [JWT.io](https://jwt.io); the decoded token should have Header, Payload, and Signature, and the signature should be valid (at the bottom below your token).

## Test The Token In APIGateway

In the AWS Console, go to you API Gateway, go to the authorizers (that should already exist because you changed the template), and hit that Test button.  Put the JWT (the id_token you validated) in, and you should get a validity confirmation.

<img src="images/20200511/authorizers.png" alt="Auth integration" style="max-height:300px"/>

## Use That Token!

In [Postman](https://www.postman.com/), hit your API Gateway (shaped like `https://<host>/<environment>/hello`) with a header of `Authorization` of your JWT token (no Bearer).  You should get the valid response!

# Room For Growth

This endpoint uses an Implicit Grant; it just gives the auth-worthy creds back to the user.  This is a risk, because those can be intercepted.  The internet, and the docs, imply that implicit grants are to be avoided if possible.

# Next Steps (/Exercises For The Reader)

1. Add more APIs
1. Use more intelligent security
1. Touch a datastore
