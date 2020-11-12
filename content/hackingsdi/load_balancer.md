---
title: "Puzzle Piece: Load Balancer"
date: 2020-08-31T15:46:49-04:00
draft: false
summary: "Context and heuristics, with numbers, for building with a load balancer."
tags: ['puzzle_pieces', 'theory']
---

As you do more of these problems, recurring patterns emerge; how to shard NoSQL data, what kinds of sites fit in a single RDS box, etc.  This series will try to call out those patterns and pieces specifically, starting from low-level parts.  Today we cover the humble load balancer.

# TLDR

Use an AWS ALB in multi-AZ and you have infinite requests/second scalability and no Single Point of Failure.  If you expect surgy traffic, ex a zonal failover, more work needed.  Load balancers are almost never the problem.

# Options for load balancing:

1. Register multiple IPs on the same DNS record and round-robin it.
1. Delegate to subdomains on different IPs, ex for geographic spread.
1. Have server-side load balancing set up.

# What is Server-Side Load Balancing?

A reverse proxy sits in between traffic coming from the broader internet and your machine.


<div class="mermaid">
graph TD
  A[Client] -->|Sends Request| B(Internet)
  B --> C{Load Balancer}
  C -->D[App Server 1]
  C -->E[App Server 2]
  C -->F[App Server 3]

</div>
<script async src="https://unpkg.com/mermaid@8.7.0/dist/mermaid.min.js"></script>

The load balancer commonly provides termination services, ex for SSL, or compression/decompression (ex with gzip).  Rather than serving HTTP requests itself, a SSLB farms out requests to a pool of app servers that do the actual work.  Then, those servers send the HTTP response back through the LB, which sends back to the client.  The request originator might get stashed in a new HTTP header sent to the backend, ex HTTP-FORWARDED-FOR.

# In-Market Options

## AWS

AWS provides 2 real options for load balancing (and one legacy option not to be used, generally).  They are:

### Application Load Balancer

An ALB has scaling built in, so it scales to handling as many requests as you need.  However, it takes time to scale the ALB, so you need to pre-warm it if you expect a spike in traffic.  Operates on OSI 7.  [Announcement](https://aws.amazon.com/blogs/aws/new-aws-application-load-balancer/) in 2016.

### Network Load Balancer

An OSI 4 load balancer, an NLB can scale to "millions of requests per second".  Offers higher performance at the expense of required configuration/existing lower in OSI.  [Announcement in 2017](https://aws.amazon.com/blogs/aws/new-network-load-balancer-effortless-scaling-to-millions-of-requests-per-second/).

### Classic Load Balancer

Only recommended for EC2 Classic instances.  Operates on both levels 4 and 7.  Don't use this.

### Materials

 * [FAQs](https://aws.amazon.com/elasticloadbalancing/faqs/#:~:text=800%20new%20TCP%20connections%20per,and%20IP%20addresses%20as%20targets.) about AWS LBs
 * [AWS Marketing Page](https://aws.amazon.com/elasticloadbalancing/) for LBs
 * [CloudAcademy](https://cloudacademy.com/blog/application-load-balancer-vs-classic-load-balancer/) on LBs

## Apache and NGINX

Apache was started in 1995, and NGINX in 2004. [Wikipedia](https://en.wikipedia.org/wiki/Load_balancing_(computing)#Server-side_load_balancers).  Both are software that does load balancing, ie you must be arranging the machines yourself, unlike AWS solutions.

# Metrics + SDI Concerns

AWS ALB: infinite requests per second.  Not a single point of failure by itself, **if** you have presence in multiple AZs.  (Otherwise, 1 AZ going down, ex us-east-1, means you are down.)

NLB: Millions of requests per second.  Multi-AZ=> not SPoF.

NGINX: [400k-500K requests/second](https://gist.github.com/denji/8359866#:~:text=Generally%2C%20properly%20configured%20nginx%20can,without%20problem%20on%20slower%20machines.).  [See here for HA config](https://www.nginx.com/products/nginx/load-balancing/) to avoid SPoF.

Apache: Still looking for metrics.
