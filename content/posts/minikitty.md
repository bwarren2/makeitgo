---
title: "Minikitty"
date: 2020-10-19T20:10:07-04:00
draft: false
summary: Deploying an ECS service that uses RDS via Copilot
---

This is a quickstart to getting an ECS service up with an RDS instance using AWS Copilot.  The project is in very active development, so this may get out of date quickly, but we will provision the DB via an addon.

# Setting up a skeleton Django project

## Make a git repo

This is actually required for some inference Copilot does.

{{< highlight bash>}}
[SideProjects]$ mkdir minikitty
[SideProjects]$ cd minikitty/
[minikitty]$ git init
Initialized empty Git repository in /Users/bwarren/SideProjects/minikitty/.git/
[minikitty](master)$ touch README.md
[minikitty](master)$ echo "Spinning something up with Copilot" >> README.md
[minikitty](master)$ git add README.md
[minikitty](master)$ git commit -m "Initial commit"
{{< /highlight >}}

## Docker requirements

Add a `Dockerfile`.

{{< highlight docker>}}
# Dockerfile
FROM python:3.8
WORKDIR /code/
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
{{< /highlight >}}

Add a `docker-compose.yml`

{{< highlight docker>}}
# docker-compose.yml
version: "3.8"

services:
  app:
    image: minikitty-server
    build:
      context: .
    container_name: minikitty-api
    volumes:
      - .:/code/
    ports:
      - "8000:8000"
{{< /highlight >}}

## Django setup

Add a requirements file.

{{< highlight bash>}}
# requirements.txt
Django==3.1.2
{{< /highlight >}}

Start a docker container to work in.

{{< highlight bash>}}
[minikitty](master)$ docker-compose run  --entrypoint bash  --rm --service-ports app
{{< /highlight >}}

You'll see the image get built like this:

{{< highlight bash>}}
Creating network "minikitty_default" with the default driver
Building app
Step 1/5 : FROM python:3.8
 ---> 4f7cd4269fa9
Step 2/5 : WORKDIR /code/
 ---> Using cache
 ---> 38d220abb92f
Step 3/5 : COPY requirements.txt .
 ---> 9519e0e8f880
Step 4/5 : RUN pip install -r requirements.txt
 ---> Running in 90f45fbe9984
Collecting Django
  Downloading Django-3.1.2-py3-none-any.whl (7.8 MB)
Collecting pytz
  Downloading pytz-2020.1-py2.py3-none-any.whl (510 kB)
Collecting sqlparse>=0.2.2
  Downloading sqlparse-0.4.1-py3-none-any.whl (42 kB)
Collecting asgiref~=3.2.10
  Downloading asgiref-3.2.10-py3-none-any.whl (19 kB)
Installing collected packages: pytz, sqlparse, asgiref, Django
Successfully installed Django-3.1.2 asgiref-3.2.10 pytz-2020.1 sqlparse-0.4.1
WARNING: You are using pip version 20.1; however, version 20.2.4 is available.
You should consider upgrading via the '/usr/local/bin/python -m pip install --upgrade pip' command.
Removing intermediate container 90f45fbe9984
 ---> e5d09f712ffc
Step 5/5 : COPY . .
 ---> 6e49e4db7597

Successfully built 6e49e4db7597
Successfully tagged minikitty-server:latest
WARNING: Image for service app was built because it did not already exist. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`.
Creating minikitty_app_run ... done
root@820db4d92b88:/code#
{{< /highlight >}}

Inside the container, use the Django admin command to start your project.

{{< highlight bash>}}
root@820db4d92b88:/code# django-admin startproject minikitty
{{< /highlight >}}

Leave the container.  Because we mounted the code as a volume in `docker-compose.yml`, the files generated inside the container appear in your host filesystem.

{{< highlight bash>}}
root@820db4d92b88:/code# exit
[minikitty](master)$ ls
Dockerfile		README.md		docker-compose.yml	minikitty		requirements.txt
{{< /highlight >}}

## Upgrading `Dockerfile`

Alter your Dockerfile to look like this.  It'll start a server by default.

{{< highlight docker>}}
# Dockerfile
FROM python:3.8
WORKDIR /code/
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
WORKDIR /code/minikitty/

ENTRYPOINT [ "python", "manage.py" ]
CMD [ "runserver", "0:8000" ]
{{< /highlight >}}

## Checkpoint: does it work?

Try bringing up your container.

{{< highlight bash>}}
[minikitty](master)$ docker-compose build app && docker-compose up
{{< /highlight >}}

You should see something like this:

{{< highlight bash>}}
Building app
Step 1/7 : FROM python:3.8
 ---> 4f7cd4269fa9
Step 2/7 : WORKDIR /code/
 ---> Using cache
 ---> 38d220abb92f
Step 3/7 : COPY requirements.txt .
 ---> Using cache
 ---> 55a143b82f1d
Step 4/7 : RUN pip install -r requirements.txt
 ---> Using cache
 ---> 839283b1f3f2
Step 5/7 : COPY . .
 ---> 2a8d128dc336
Step 6/7 : ENTRYPOINT [ "python", "minikitty/manage.py" ]
 ---> Running in 2d37b739faf8
Removing intermediate container 2d37b739faf8
 ---> df7ab041539e
Step 7/7 : CMD [ "runserver", "0:8000" ]
 ---> Running in 3f8992eb4c51
Removing intermediate container 3f8992eb4c51
 ---> 95f2623fd966

Successfully built 95f2623fd966
Successfully tagged minikitty-server:latest
Recreating minikitty-api ... done
Attaching to minikitty-api
minikitty-api | Watching for file changes with StatReloader
{{< /highlight >}}

Go to localhost:8000 and...

![It works](/img/tinycat/it_works.gif)

# Using Postgres

## Make a .env file

We need to put some env vars in local development.

{{< highlight bash>}}
# local.env

DJANGO_SETTINGS_MODULE=minikitty.settings

# Don't make __pycache__
PYTHONDONTWRITEBYTECODE=1

# Postgres connection vars
PGDATABASE=minidb
PGUSER=postgres
PGPASSWORD=mylocalpassword
PGHOST=db
PGPORT=5432
{{< /highlight >}}

Note the PG variables.

Now change the settings file to allow incoming traffinc and use the env vars for Postgres.

{{< highlight python>}}
# settings.py
# ...
import os
# ...
ALLOWED_HOSTS = ["*"]
# ...
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.postgresql_psycopg2",
        "NAME": os.getenv("PGDATABASE"),
        "USER": os.getenv("PGUSER"),
        "PASSWORD": os.getenv("PGPASSWORD"),
        "HOST": os.getenv("PGHOST"),
        "PORT": os.getenv("PGPORT"),
    }
}
{{< /highlight >}}

Add the postgres dependency in `requirements.txt`

{{< highlight bash>}}
# requirements.txt
Django==3.1.2
psycopg2==2.8.6
{{< /highlight >}}

And add a postgres container to `docker-compose.yml`

{{< highlight docker>}}
version: "3.8"

services:
  app:
    image: minikitty-server
    build:
      context: .
    container_name: minikitty-api
    volumes:
      - .:/code/
    ports:
      - "8000:8000"
    env_file:
      - local.env
    depends_on:
      - db

  db:
    image: postgres:13
    restart: always
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=mylocalpassword
      - POSTGRES_DB=minidb
    volumes:
      - ./docker/postgres:/docker-entrypoint-initdb.d
      - postgres-data:/var/lib/postgresql/data
    ports:
      - "0:5432"
    logging:
      # limit logs retained on host to 12.5MB
      driver: "json-file"
      options:
        max-size: "500k"
        max-file: "25"

volumes:
  postgres-data:
{{< /highlight >}}

Note: we added a volume for postgres to write in, added a dependency on the db to the app, and added the env variables to app.

## Checkpoint 2: Does it still work?

Start up both containers

{{< highlight bash>}}
[minikitty](master)$ docker-compose build app && docker-compose run  --entrypoint bash  --rm --service-ports app
{{< /highlight >}}

You should see something like this

{{< highlight bash>}}
Building app
Step 1/7 : FROM python:3.8
 ---> 4f7cd4269fa9
Step 2/7 : WORKDIR /code/
 ---> Using cache
 ---> 38d220abb92f
Step 3/7 : COPY requirements.txt .
 ---> Using cache
 ---> 866f89725196
Step 4/7 : RUN pip install -r requirements.txt
 ---> Using cache
 ---> 51a2f64bf934
Step 5/7 : COPY . .
 ---> Using cache
 ---> f9f0aeb88432
Step 6/7 : ENTRYPOINT [ "python", "manage.py" ]
 ---> Using cache
 ---> a8cc11c8de09
Step 7/7 : CMD [ "runserver", "0:8000" ]
 ---> Using cache
 ---> fc4a30761dc7

Successfully built fc4a30761dc7
Successfully tagged minikitty-server:latest
Creating network "minikitty_default" with the default driver
Creating volume "minikitty_postgres-data" with default driver
Creating minikitty_db_1 ... done
Creating minikitty_app_run ... done
{{< /highlight >}}

When you are dropped in, try running migrations:

{{< highlight bash>}}
root@d71f1fcdcb6f:/code/minikitty# python manage.py migrate
{{< /highlight >}}

You should see this:

{{< highlight bash>}}
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying admin.0003_logentry_add_action_flag_choices... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying auth.0009_alter_user_last_name_max_length... OK
  Applying auth.0010_alter_group_name_max_length... OK
  Applying auth.0011_update_proxy_permissions... OK
  Applying auth.0012_alter_user_first_name_max_length... OK
  Applying sessions.0001_initial... OK
{{< /highlight >}}

# Setting up a de minimis service

## Adding a healthcheck

ECS will need a way to know the container is up.  We add a healthcheck URL for it to hit.

{{< highlight python>}}
import os
from django.contrib import admin
from django.urls import path
from django.http import HttpResponse

urlpatterns = [
    path("admin/", admin.site.urls),
    path("healthcheck/", lambda x: HttpResponse("OK")),
]
{{< /highlight >}}

# Checkpoint 3: Filesystem structure

Your project should look roughly like this:

```
.
├── Dockerfile
├── README.md
├── copilot
│   └── api
│       ├── addons
│       │   └── rds.yml
│       └── manifest.yml
├── docker
│   └── postgres
├── docker-compose.yml
├── local.env
├── minikitty
│   ├── manage.py
│   └── minikitty
│       ├── __init__.py
│       ├── asgi.py
│       ├── settings.py
│       ├── urls.py
│       └── wsgi.py
└── requirements.txt
```

# Sending things to AWS

You might need to specify a profile to use for this part.  I did that like this.

{{< highlight bash>}}
[minikitty](master)$ export AWS_PROFILE=BenW
{{< /highlight >}}

## Initializing Copilot

[Copilot](https://aws.github.io/copilot-cli/) is a high level service management tool.  Install it, and when you are ready, initialize your app like this.

{{< highlight bash>}}
[minikitty](master)$ copilot init  --app minikitty                \
  --svc api                              \
  --svc-type 'Load Balanced Web Service' \
  --dockerfile './Dockerfile'            \
  --port 8000            \
  --deploy
> {{< /highlight >}}

You should see this:

{{< highlight bash>}}
Welcome to the Copilot CLI! We're going to walk you through some questions
to help you get set up with an application on ECS. An application is a collection of
containerized services that operate together.

Ok great, we'll set up a Load Balanced Web Service named api in application minikitty listening on port 8000.

✔ Created the infrastructure to manage services under application minikitty.

✔ Manifest file for service api already exists at copilot/api/manifest.yml, skipping writing it.
Your manifest contains configurations like your container size and port (:8000).

✔ Created ECR repositories for service api.
{{< /highlight >}}

Note: this is where the CLI will give you a prompt about deploying to a test environment.  DON'T do it!  We need to add something.

{{< highlight bash>}}
All right, you're all set for local development.
Deploy: No

No problem, you can deploy your service later:
- Run `copilot env init --name test --profile default --app minikitty` to create your staging environment.
- Update your manifest copilot/api/manifest.yml to change the defaults.
- Run `copilot svc deploy --name api --env test` to deploy your service to a test environment.
{{< /highlight >}}

Now, to add an RDS box to our environment, we need to specify this addon.  In `copilot/`, where there is currently a `manifest.yml`, add a folder named `addons`.  Within that folder, make a file names `rds.yml` with this Cloudformation:

{{< highlight yaml>}}
# You can use any of these parameters to create conditions or mappings in your template.
Parameters:
  App:
    Type: String
    Description: Your application's name.
  Env:
    Type: String
    Description: The environment name your service, job, or workflow is being deployed to.
  Name:
    Type: String
    Description: The name of the service, job, or workflow being deployed.

Resources:
  DBSubnetGroup:
    Type: AWS::RDS::DBSubnetGroup
    Properties:
      DBSubnetGroupDescription: !Sub "${Env} public subnets"
      DBSubnetGroupName: !Sub "${Env}-public-subnets"
      SubnetIds:
        - Fn::Select:
            - "0"
            - Fn::Split:
                - ","
                - Fn::ImportValue: !Sub "${App}-${Env}-PublicSubnets"
        - Fn::Select:
            - "1"
            - Fn::Split:
                - ","
                - Fn::ImportValue: !Sub "${App}-${Env}-PublicSubnets"

  MyRDSInstanceRotationSecret:
    Type: AWS::SecretsManager::Secret
    Properties:
      Description: "This is my rds instance secret"
      GenerateSecretString:
        SecretStringTemplate: '{"username": "postgres"}'
        GenerateStringKey: "password"
        PasswordLength: 16
        ExcludePunctuation: true

  SecretRDSInstanceAttachment:
    Type: AWS::SecretsManager::SecretTargetAttachment
    Properties:
      SecretId: !Ref MyRDSInstanceRotationSecret
      TargetId: !Ref RDSDBInstance1
      TargetType: AWS::RDS::DBInstance

  RDSDBInstance1:
    Properties:
      AllocatedStorage: "20"
      DBName: "tinydb"
      MasterUserPassword: !Sub "{{resolve:secretsmanager:${MyRDSInstanceRotationSecret}::password}}"
      MasterUsername: !Sub "{{resolve:secretsmanager:${MyRDSInstanceRotationSecret}::username}}"
      AvailabilityZone:
        Fn::Select:
          - "0"
          - Fn::GetAZs: { Ref: "AWS::Region" }
      DBInstanceClass: db.t2.micro
      DBSubnetGroupName:
        Ref: DBSubnetGroup
      Engine: postgres
      PubliclyAccessible: true
    Type: "AWS::RDS::DBInstance"

# All outputs are injected as environment variables.
Outputs:
  # The secret will be inject as an environment variable to your service! You'll need to parse the json.
  PGPASSWORD:
    Value: !Sub "{{resolve:secretsmanager:${MyRDSInstanceRotationSecret}::password}}"

  PGUSER:
    Value: !Sub "{{resolve:secretsmanager:${MyRDSInstanceRotationSecret}::username}}"

  PGHOST:
    Value:
      Fn::GetAtt: [RDSDBInstance1, Endpoint.Address]

  PGPORT:
    Value:
      Fn::GetAtt: [RDSDBInstance1, Endpoint.Port]

  PGDATABASE:
    Value: "minidb"
{{< /highlight >}}

Note: if you are naming your DB something different, be sure to set that in `PGDATABASE` and on the DBInstance!


Next, change the `manifest.yml`.  Add these attributes.  (Ignore ellipses.)

{{< highlight yaml>}}
...
http:
  # Requests to this path will be forwarded to your service.
  # To match all requests you can use the "/" path.
  path: '/'
  # You can specify a custom health check path. The default is "/"
  healthcheck: '/healthcheck/'
...
variables:                    # Pass environment variables as key value pairs.
 DJANGO_SETTINGS_MODULE: minikitty.settings
...
{{< /highlight >}}

Initialize the test environment for your app:

{{< highlight bash>}}
[minikitty](master)$ copilot env init --name test  --app minikitty
{{< /highlight >}}

You should see:

{{< highlight bash>}}
Which credentials would you like to use to create test? [profile BenW]
Would you like to use the default configuration for a new environment?
    - A new VPC with 2 AZs, 2 public subnets and 2 private subnets
    - A new ECS Cluster
    - New IAM Roles to manage services in your environment
 Yes, use default.
✔ Created the infrastructure for the test environment.
- Virtual private cloud on 2 availability zones to hold your services     [Complete]
- Virtual private cloud on 2 availability zones to hold your services     [Complete]
  - Internet gateway to connect the network to the internet               [Complete]
  - Public subnets for internet facing services                           [Complete]
  - Private subnets for services that can't be reached from the internet  [Complete]
  - Routing tables for services to talk with each other                   [Complete]
- ECS Cluster to hold your services                                       [Complete]
- Application load balancer to distribute traffic                         [Complete]
✔ Linked account 871089662319 and region us-east-1 to application minikitty.

✔ Created environment test in region us-east-1 under application minikitty.
{{< /highlight >}}

Now we can deploy our service:

{{< highlight bash>}}
[minikitty](master)$ copilot svc deploy
{{< /highlight >}}

You should see:

{{< highlight bash>}}
Sending build context to Docker daemon  121.3kB
Step 1/8 : FROM python:3.8
 ---> 4f7cd4269fa9
Step 2/8 : WORKDIR /code/
 ---> Using cache
 ---> 38d220abb92f
Step 3/8 : COPY requirements.txt .
 ---> Using cache
 ---> 81d154f89116
Step 4/8 : RUN pip install -r requirements.txt
 ---> Using cache
 ---> 3c265a8ca319
Step 5/8 : COPY . .
 ---> c0c2b0d36dac
Step 6/8 : WORKDIR /code/minik/
 ---> Running in e33e09240885
Removing intermediate container e33e09240885
 ---> a2dbb81e1376
Step 7/8 : ENTRYPOINT [ "python", "manage.py" ]
 ---> Running in b9f3fced310f
Removing intermediate container b9f3fced310f
 ---> 6e8a782af9c6
Step 8/8 : CMD [ "runserver", "0:8000" ]
 ---> Running in 602f3ec428ae
Removing intermediate container 602f3ec428ae
 ---> 8cfe233b4920
Successfully built 8cfe233b4920
Successfully tagged 871089662319.dkr.ecr.us-east-1.amazonaws.com/minik/api:be8c379
Login Succeeded
The push refers to repository [871089662319.dkr.ecr.us-east-1.amazonaws.com/minik/api]
db3a3512f051: Pushed
f0cf3f685125: Layer already exists
40741e43e9a0: Layer already exists
db24877f8528: Layer already exists
508c3f3b7a64: Layer already exists
7e453511681f: Layer already exists
b544d7bb9107: Layer already exists
baf481fca4b7: Layer already exists
3d3e92e98337: Layer already exists
8967306e673e: Layer already exists
9794a3b3ed45: Layer already exists
5f77a51ade6a: Layer already exists
e40d297cf5f8: Layer already exists
be8c379: digest: sha256:814bbe523ee8671f25af994c9ebc2535a402d13a40463c8d3e640b550f1cf65e size: 3052


✔ Deployed api, you can access it at http://minik-Publi-7KSWRWH1UFJU-2070867723.us-east-1.elb.amazonaws.com.
{{< /highlight >}}

Finally, and I am not sure yet why this is necessary: alter the security group that the RDS instance is in to allow traffic from any source.  When I don't do this, the ECS service will just continually provision and drain tasks.  The next container that comes up after you do this should join the load balancer.  Hit the URL provided above and:

![It works](/img/tinycat/up_on_aws.png)

# Next steps

This is just the start.  We need to run migrations, set up pipelines, actually make a prod-ready stack; it's a whole thing.  But, we can just keep redeploying the service as we iterate!
