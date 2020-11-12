---
title: "Bitly"
date: 2020-09-06T22:45:43-04:00
draft: false
summary: "Building a URLShortener"
---

Another day, another problem. So how would we implement a URL shortener like Bitly?

Here, clarifying questions are very important. A basic URL shortener just converts a short value into a URL redirect, ex

```
GET newbitly.com/somehash -> 301 MOVED someother.com
```

But there are a couple obvious extension directions. What if the client wants password protection on URLs? How would we support vanity URLs? In our first implementation we will ignore those requirements, then extend.

# Clarification

I would normally ask the interviewer for their scope rather than making stuff up, but here we go.

Shortness is a key value of a URL shortener, so let's presume that the hashes are no more than 8 characters long. Using just uppercase, lowercase, and numbers, this gives us 62\*\*8 possible addresses for 2 \* 10^14 supportable addresses; this is about 200 trillion addresses. We can just add in two more characters to make it an even 64 options.

Let's plan for 3 million writes per day, or a billion a year. We can easily scale that up given our 200T address pool.

What is the maximum URL length we will support? Research says IE has a limit of 2KB, that seems like a good approximation.

# Entities

```
URLRedirect (~2KB)
    id 8B string
    owner_api_key 16B string
    to_url 2KB string
```

Storing these URLRedirects is easy. They are about 2KB in possible sie, and we intend to store 1 billion a year, so we need about 2TB of storage a year if all the redirects are as long as possible.

This should be stored in a NoSQL table with a hash key on id.

# API Calls

```
/api/redirect/
{
    api_key: <value>, to_url: <url>
} -> 201 {to_url: <url>, redirect_link: <url>}
```

**I'll talk about how we make new URLRedirects after I describe the system working as though they exist, because it is simpler**.

Once we have some URLRedirects made, our HTTP servicing is conceptually simple in the "three tier app" pattern:

<div class="mermaid">
graph TD
	LB[Load Balancer] --> AS1
	LB[Load Balancer] --> AS2
	AS1[App Server 1] --> DDB[DynamoDB]
	AS2[App Server 2] --> DDB[DynamoDB]
</div><script async src="https://unpkg.com/mermaid@8.7.0/dist/mermaid.min.js"></script>

With as many app servers as we require to serve all the incoming requests. We might instead go full serverless and use:

<div class="mermaid">
graph TD
	APIG[API Gateway] --> L[Lambda Tier]
	L --> DDB[DynamoDB]
</div>

I like this one more because the scaling is taken care of for us. When there is a huge spike, like if Elon Musk shared a `newbitly` link promising free bitcoin, the cold-start time for Lambda is around 500ms; not ideal, but acceptable.

API Gateway also offers us between .5 and 237GB of caching, which can reduce load on our system for links like Elon's. 200GB of ~2KB objects is about 100M values, or a solid month's worth of top URLRedirects. This should dramatically reduce Lambda dispatch and DDB read capacity costs.

# New URLRedirects

So how do we actually create new redirects? We need a key-management service that can offer up a new available key whenever a Lambda gets a POST to create. The machines in the KMS cluster should be load balanced and check out new keys from their shared DB whenever they start up or their inventories are exhausted. Whenever they check out keys, they need to mark the keys as used in their shared DB. If a machine goes down randomly without offering keys, that's fine; we have tons of headroom

<div class="mermaid">
graph TD
	APIG[API Gateway] --> L[Lambda Tier]
	L --> DDB[DynamoDB]
    L-->KMSLB[KMS Load Balancer]
    KMSLB[KMS Load Balancer]-->KMS1[Keyserver 1]
    KMSLB[KMS Load Balancer]-->KMS2[Keyserver 2]
    KMS2-->DB[KMS DB]
    KMS1-->DB[KMS DB]
</div>

The entities for the KMS DB might be as simple as

```
MaxID
    id int

Transaction
    start_id
    appserver_ip
    checkout_ts
```

with Keyservers basically running "UPDATE MaxID set id = id + \$CHECKOUTSIZE RETURNING id". Because our 8 character hashes map to ints, Keyservers can know what range of ints they got and convert them. (A string is a number in base 64 in our system, so convert the ints you got into base 64.)

# Extensions

We did it! We can create new redirects, and we can serve redirects. APIGateway, Lambda, and DynamoDB should scale well, because we can avoid the hot-partition problem with caching on APIGateway.

How about the extensions? Adding a password is easy; add a field to URLRedirect to store it, send the user a login page, when they submit with the correct password redirect them.

If we want vanity urls, we need to amend the Lambda tier. When the API payload specifies `vanity=true` and provides a proposed vanity url, the Lambda should _not_ hit the KMS, and instead do a conditional PutItem on the DDB table directly and either 201 or 400 the response.

# Status

This problem is much easier than the EBay version, and it felt good to write it out. Progress!
