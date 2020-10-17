---
title: "Fargate"
date: 2020-10-05T20:39:11-04:00
draft: false
summary: "Notes from starting with Fargate"
---

Fargate sounds like exactly my kind of thing: "Here's a container, here's a scalign strategy, go do the infra". Infrastructure is immensely configurable, and most of the states are wrong, so why waste time on a fiddly process with Right Answers?

Getting a Cluster and Task Definition up in the console is easy enough, but I want more: I want CLI control. Unfortunately, that has proven elusive.

> "There should be one-- and preferably only one --obvious way to do it." - `$ python -c "import this" | head -n 15 | tail -n 1`

In the traditional AWS spirit, there are at least three ways of doing things:

1. `aws-cli` via `aws ecs ...`
1. `ecs-cli` via `ecs-cli ...`
1. `copilot` via `copilot ...`

I have played with aws-cli a long time ago, but spent tonight on `ecs-cli` and `copilot`.

# [ECS CLI](https://github.com/aws/amazon-ecs-cli)

After install, you really want to run some config steps:

{{< highlight bash  >}}
ecs-cli configure profile --profile-name=<myprofile> --access-key=<mykey> --secret-key=<supersecret>
{{< /highlight >}}
{{< highlight bash  >}}
ecs-cli configure --region=us-east-1 --cluster=pollsapi
{{< /highlight >}}

Make a cluster

{{< highlight bash  >}}
aws ecr create-repository --repository-name pollsapi-app --profile <myprofile> --region us-east-1
{{< /highlight >}}

Then the Push commands from ECR for that repo:

{{< highlight bash  >}}
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <anumber>.dkr.ecr.us-east-1.amazonaws.com
docker tag pollsapi-app:latest <anumber>.dkr.ecr.us-east-1.amazonaws.com/pollsapi-app:latest
docker push <anumber>.dkr.ecr.us-east-1.amazonaws.com/pollsapi-app:latest
{{< /highlight >}}

Then start the service up:

{{< highlight bash  >}}
ecs-cli up --capability-iam --create-log-groups
{{< /highlight >}}

But after taking down the broken service I could not spawn it again:

{{< highlight bash  >}}
(pollsapi) [pollsapi](master)\$ ecs-cli compose up
WARN[0000] Skipping unsupported YAML option for service... option name=build service name=web
INFO[0000] Using ECS task definition TaskDefinition="pollsapi:1"
ERRO[0000] Error running tasks error="InvalidParameterException: No Container Instances were found in your cluster." task definition=0xc000382530
FATA[0000] InvalidParameterException: No Container Instances were found in your cluster.
{{< /highlight >}}

Redirected toward docs that use the console, I gave up and went to the new hotness: `copilot`.

# [Copilot CLI](https://github.com/aws/copilot-cli)

Copilot looks like it is designed by people who really care about developer happiness. Their docs are good, their ergonomics are pleasant; I really, really want this to be the solution that works for me, at least on smaller projects.

Set the right profile:

{{< highlight bash  >}}
aws configure --profile <myprofile>
{{< /highlight >}}

{{< highlight bash  >}}

(pollsapi) [building-api-django](master)\$ copilot init
Welcome to the Copilot CLI! We're going to walk you through some questions
to help you get set up with an application on ECS. An application is a collection of
containerized services that operate together.

Application name: pollsapi
Service type: Load Balanced Web Service
Service name: pollsapi-app
Dockerfile: pollsapi/Dockerfile
no EXPOSE statements in Dockerfile pollsapi/Dockerfile
Port: 80
Ok great, we'll set up a Load Balanced Web Service named pollsapi-app in application pollsapi listening on port 80.

✔ Created the infrastructure to manage services under application pollsapi.

✔ Wrote the manifest for service pollsapi-app at copilot/pollsapi-app/manifest.yml
Your manifest contains configurations like your container size and port (:80).

✔ Created ECR repositories for service pollsapi-app.

All right, you're all set for local development.
Deploy: Yes

✔ Created the infrastructure for the test environment.

- Virtual private cloud on 2 availability zones to hold your services [Complete]
- Virtual private cloud on 2 availability zones to hold your services [Complete]
  - Internet gateway to connect the network to the internet [Complete]
  - Public subnets for internet facing services [Complete]
  - Private subnets for services that can't be reached from the internet [Complete]
  - Routing tables for services to talk with each other [Complete]
- ECS Cluster to hold your services [Complete]
- Application load balancer to distribute traffic [Complete]
  ✔ Linked account <mynumber> and region us-east-1 to application pollsapi.

✔ Created environment test in region us-east-1 under application pollsapi.
Sending build context to Docker daemon 46.59kB
Step 1/9 : FROM python:3
---> 3189819ced3e
Step 2/9 : ENV PYTHONUNBUFFERED 1
---> Using cache
---> fd06036dafac
Step 3/9 : RUN mkdir /code
---> Using cache
---> 80d2407f4513
Step 4/9 : WORKDIR /code
---> Using cache
---> 0b9a54e1231c
Step 5/9 : COPY requirements.txt /code
---> Using cache
---> e35e5d6c4442
Step 6/9 : RUN pip install -r requirements.txt
---> Using cache
---> 8a2e63abdcd8
Step 7/9 : COPY . /code/
---> Using cache
---> e58175170b27
Step 8/9 : ENTRYPOINT ["python", "manage.py"]
---> Using cache
---> 45e3ee15c11d
Step 9/9 : CMD ["runserver", "0.0.0.0:8000"]
---> Using cache
---> 5465d39c9dd9
Successfully built 5465d39c9dd9
Successfully tagged <mynumber>.dkr.ecr.us-east-1.amazonaws.com/pollsapi/pollsapi-app:63efd56
Login Succeeded
The push refers to repository [<mynumber>.dkr.ecr.us-east-1.amazonaws.com/pollsapi/pollsapi-app]
e6cbf376efc9: Pushed
2c21345efcae: Pushed
c26e299ac505: Pushed
cdbc1b4e4c3a: Pushed
fb624d7424e8: Pushed
91eb7baa6eee: Pushed
7067471c4c31: Pushed
b69b7976c2ab: Pushed
cfb4ccdac258: Pushed
46a297e68c47: Pushed
cf47dfabe081: Pushed
c53d956ebfec: Pushed
6086e1b289d9: Pushed
63efd56: digest: sha256:4107e0a2bab87dfaa985b45e423c4fa1a475442fc1e55e2ed7b6752926ca7b80 size: 3051

✔ Deployed pollsapi-app, you can access it at http://polls-Publi-14SISDTFCME4Q-1743346331.us-east-1.elb.amazonaws.com.
{{< /highlight >}}

Boom!

Initially I got some 503s because the service was failing the health check; I had an auth restriction on the / endpoint healthcheck used by default. Adding

{{< highlight python  >}}
path(r"", lambda x: HttpResponse("Ok")),
{{< /highlight >}}

to my Django urls fixed that.

Bringing down the service when it was busted was as simple as

{{< highlight python  >}}
copilot svc delete
{{< /highlight >}}

and I could recreate it with

{{< highlight python  >}}
copilot svc init
copilot svc deploy --name pollsapi --env test
{{< /highlight >}}

and check the status with

{{< highlight python  >}}
copilot svc status
{{< /highlight >}}

Still not sure how to update a service in-place. But I think there is potential here!

Read more [here](https://aws.amazon.com/blogs/containers/introducing-aws-copilot/).
