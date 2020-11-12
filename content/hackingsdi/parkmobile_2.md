---
title: "Parkmobile 2, Feature Boogaloo"
date: 2020-08-22T22:03:26-04:00
draft: false
tags: ['case_study', 'followup']
summary: "Extending my 1st try on ParkMobile with guidance."
---


One of my much-more-knowledgeable friends gave me feedback on [last time]({{< ref "first_try.md" >}}), and challenges to do more.

>Good first try! Some thoughts:

> At the beginning, write down all of the things a user can do in the app. You mentioned a couple things, but don’t forget: a user can find the zones closest to him and the system sends alerts (push, email, SMS) to the user as time elapses

> The two I mentioned might be more interesting problems to solve and the interviewer might ask you to talk about those instead.

> The interviewer won’t care about “admin adds stuff” or “user creates account”. They’ll want to know the interesting APIs and they’ll want to know the input and output of each one.

> They’ll also be interested in the tables and what fields you’re storing.

> Last thing, they’ll want you to talk about each technology, the alternatives, the trade-offs, and why you are picking whichever one.

> **Try planning the other two features I mentioned as your next exercise: searching for nearby zones and notifications.**

So here we go.

### Nearby zones

Users want to be able to make an API hit with their current location and get a list of the nearest N zones.

```
# API shape:

zones_near_location(api_key, latitude, longitude) -> List[Zone]
```

#### Clarifying requirements

First questions: what does "nearby" mean?  What are zones shaped like?

I am going to assume that:

1. zones don't meaningfully overlap;
1. zones are small-ish, ie not defined by the points New York, Los Angeles, Duluth;
1. are regularly shaped, ie not basically a very narrow `+` ;
1. Zones aren't defined extremely granularly such that a ton of points define them;

I will be designing for zones that are basically rectangles that might span a long city block.

In terms of zone searching, we can make a few different guesses and assumptions to triangulate on something good.

#### Bad Option: Split on Service Territory

An initial naive guess would be "Parkmobile is most useful in cities, define every city-region of Parkmobile service as a hashmap of district => list of zone, and have the app tier filter the list based on distance from the user".  This solution will have seam problems when regions of Parkmobile service get big enough to be next to eachother/need to be partitioned due to size.

If we are doing something small, dividing the workload by hashing values on service territory or maintaining a single DB and doing lookups on adjacent zones as well would be fine.  I assume we are building a global service, so we don't get to avoid the "places that are next to eachother" problem, and this option does not work.

**We are designing a global Parkmobile, that is available _everywhere_.**

#### Bad Option: Measure Distance To Zone Center

It might be tempting to reduce a zone to a single point and compare distances to those to get an ordered list of zone closeness, but you can easily imagine the user being near the top of a long zone or a bit farther away from a smaller zone to produce degenerate wrong answers.

#### Choice: Measure Distance To All Perimeter Points

TLDR: we are going to make a system for finding closest-by-latitude and a very similar system for closest-by-longitude and intersect their answers.

Let's imagine a system of `point ID => point data` mapping just for latitude, where Zones are defined by a perimeter of Points.  This might be implemented as a DynamoDB table with point ID as the hash key, and latitude as the sort key, so we can say things similar to "get the values from the table where latitude greater than our reference latitude order by latitude ascending", and analogously for descending to get a bunch of points that are near our reference point in either direction.

(Implementationally so I am not just citing products, part of it is a pool of key:value servers standing behind a consistent hashing load balancer to store the data. I am not quite sure how the sort works, but I suspect it is a distributed B-tree index using similar shard-on-hash key-value store.  As for alternatives, it's possible to roll one's own hashing and key value clusters with something like Redis, but I appreciate not having to maintain versions and security patches etc by using a managed provider like AWS.)

After making the "ascending points and descending points" queries, we traverse the join of those two lists, we can get the zones those values correspond to in order of proximity to our target point.  Imagining a duplicate system for longitude, our app tier would be responsible for matching up the latitude and longitude for the same point, calculating the distance from the reference point to that point, and assembling a list of the top N zones.  If a zone has been seen before in the sort, its points can be skipped.

If zones are defined by up to six points, we may need to query up to 6x the number of zones we are looking for to be sure we got enough zones.  We will need to do this for both the ascending and descending queries, and for latitude and longitude, so it could be up to 24x as many point lookups as zones we are looking for.  Practically speaking, though, users only want <= 10 nearby zones, so we will need to query 240 points to reliably get 10 zones.

#### How Much Data?

I looked up the precision and storage of latitude and longitude, and the 4th decimal is worth 11 meters.  We need 4 bytes for latitude and 4 bytes for longitude, or 8 bytes * 240 ~= 2KB just for the keys.  Another 8 bytes for a long point ID, assuming we want 1 meter resolution on the entire planet with a trillion square meters on it, means we need  another 2KB for the point ID, and I will assume another 2KB for the zone IDs.  We then look up the zone data for the 10 zones we get, which will be small.  So we have ~6-8KB of data egress for our storage to serve the "zones near me" query.


### Reservation alerts

Users want notifications when their reservation is about to run out.  We will support reminders at 15 minutes and 3 minutes, because in my recollection this is not configurable, and the utility of setting when the notification is sent is low.

```
# API shape:

None.  This is an automatic feature when reservations are made.
```

#### Basic Option: Scan For What Needs Notifications

Last time we said there were about 7 million reservations per day.  That's close to a query we could run in a periodic task every minute or so for what reservations need updates.  This might be done in response to minutely or 30-secondly scheduled Cloudwatch events.  To make the problem harder, I will assume the data is big enough that I can't manage it with one query and a sort.

#### Choice: Enqueue Check-And-Send Events

For arbitrary scaling, hash the reservation on zone to a pool of machines running queues.  In a naive implementation, there might be 2*60*24 = 2880 queues per machine, corresponding to all the 30-second intervals of the day.  Kicked off by scheduled CloudWatch events, lambdas consume from the queues for the relevant 30s interval (and the past 1-2 if we are paranoid), check if the reservation is still active (because it might have been cancelled since creation,) and send notifications via AWS SNS.  Queues ack late because we think it is less bad to send 2 notifications than to send no notification at all.

#### How Much Data?

8 bytes (per reservation ID if we are using Longs) get written once and read twice (on the 15 minute send and the 3 minute send).  A reservation is also read on each send to check if the reservation is active, so ~32-50B of IO happen per reservation due to notifications.  Using our original 7 million reservations per day number, that would be 350MB of IO for notifications daily.


# Status Update

I am going to try keeping track of my hours spent on this project.  1 hour on the first try and ~1 hour on this draft means 2 hours so far.

Frankly, I still feel ennervated when I think about the systems design inteview, so I haven't fully processed things emotionally yet.  But we keep going.
