---
title: "First Try"
date: 2020-08-21T23:51:06-04:00
draft: false
tags: ['case_study']
summary: "Designing ParkMobile as a case interview."
---

To get me calibrated, I was given a sample problem to work through in a mock interview: design Parkmobile.  Assume all the towers etc already exist.

I lost the scratchpad I used for that interview.  What follows is the best recollection I have of how it went.

Starting with **identifying functionalities**.  Parkmobile lets people park their cars in designated zones for specific periods of time under a cap. We presume that there is also an API for law enforcement to get data on whether a given car is supposed to be in the current zone at the current time.

Then I defined **entities and relationship**.  I made a simplifying assumption that a given user only has one car registered on parkmobile (which in hazy memory might even be true).

<div class="mermaid">
classDiagram
    class User {
        int id
        String first_name
        String last_name
        String license_plate
    }
	class Zone{
        int id
        geojson Perimeter
	}
	class Reservation{
		user User
		zone Zone
        datetime startTime
        int duration_seconds
	}
  User <|-- Reservation
  Zone <|-- Reservation

</div>
<script async src="https://unpkg.com/mermaid@8.7.0/dist/mermaid.min.js"></script>
<style>
g.classGroup rect {
  fill: none !important;
  stroke: black !important;
}
g.classGroup text {
  fill: black !important;
}
g.classGroup line {
  stroke: none !important;
}
</style>

Under prodding, I **identified workflows**:

1. Admins create a bunch of zones
1. User creates an account
1. User creates a reservation
1. Law enforcement passing a car checks the plate to see if it is current.

This tells us an **access pattern**: we will need to be able to get things by plate, which will be important later.

Then I defined **api access**.  I focused heavily on CRUD operations.

```
create_user(...)
get_user(...)
update_user(...)

create_zone(...)
get_zone(...)
update_zone(...)

create_reservation(...)
get_reservation(...)
update_reservation(...)

check_plate(...)
```

Handwaving a bit, I enumerated the attributes of those API calls and called them all <= 500B.  We now start with **sizing and usage assumptions**.

Most of the time, people are not using Parkmobile.  It's mostly useful in cities, and many people in cities are not using it as part of daily life; they drive from home to work and back.

Given that, I think ~1% of the population of the US is using Parkmobile on any given day, usually once or twice.

Making a **market sizing assumption** that we are building for the US and Europe, and that the EU is about as big as the US by population, that means about *7 million* new reservations per day, distributed throughout the day because of time zones.

Next: **data-sizing napkinmath.**  At ~86K seconds in a day, that is about 81 writes per second.  Handwaving a write as 500B, that means about 45KB of data written per second.  7 million writes/day at 500B/write is 3.5GB of newly-written data per day, or 1.2 TB/year, or 6TB in a 5 year auditing period.  I observed an at-least 5:1 read:write ratio because users and police would check a reservation multiple times even when it is only written once.  This fits in one very large RDS box, but in the actual session I used different assumptions and got a larger number, skewing toward a NoSQL implementation.

I deliberated at length about the appropriate storage of the data.  I broke it up into a storage block (primary and replica) per entity, with the understanding that app servers might make multiple requests to stitch the data together.

<div class="mermaid">
graph TD
	UP[User Primary] --> UB[User Backup]
	RP[Reservation Primary] --> RB[Reservation Backup]
	ZP[Zone Primary] --> ZB[Zone Backup]
</div>

I observed that consistent hashing based on the plate of the user could be used to scale the storage blocks as needed and were unlikely to have hotspots.

I added a stateless app tier in front of the storage tier to stitch data together in response to API requests and respond to things like rate limiting.  I added a load balancer in front of the app tier.

<div class="mermaid">
flowchart TD
    LB[Load Balancer]
	subgraph app
    AT1[App Tier Box 1]
	AT2[App Tier Box 2]
    end
    subgraph storage
	UP[User Primary] --> UB[User Backup]
	RP[Reservation Primary] --> RB[Reservation Backup]
	ZP[Zone Primary] --> ZB[Zone Backup]
    end
    app --> storage
    LB --> app
</div>

Under prompting, I added a cache in front of the LB to reduce redundant access.

<div class="mermaid">
flowchart TD
    MC[Cache: Plate => Reservation]
    LB[Load Balancer]
	subgraph app
    AT1[App Tier Box 1]
	AT2[App Tier Box 2]
    end
    subgraph storage
	UP[User Primary] --> UB[User Backup]
	RP[Reservation Primary] --> RB[Reservation Backup]
	ZP[Zone Primary] --> ZB[Zone Backup]
    end
    app --> storage
    MC --> LB
    LB --> app
</div>

And the app tier would need to reach out and invalidate the cache for a plate on any write of a reservation with that plate.  Problem observed by the interviewer: sure we could use least-recently-used, least-frequently-used, time-based, or FIFO eviction strategies.  But is this cache even going to get full?  If 80% of use comes from 20% of users, they will just keep updating the cache with their recent reservation, and it might not even be that large.

The mock interview ended.  I don't think, compared to the broad candidate pool, I did a good job.  But that is why this is the first calibration practice.

#### Things to improve on

 * Familiarity with heuristic metrics.  How many connections should I presume possible for a reasonable box?
 * Intuition for scale.  I should have been able to pick out faster that a standard Django and Postgres implementation would probably work here.
 * My thinking was muddy about reservation storage and access.  I implicitly was building around only storing one reservation at a time for a user.

#### Questions I have

In theory, there is a three-dimensional parameter space of number of connections, duration of connection, and result size of query that has a surface of maximum-stable-performance for a given RDS box.  I don't know what that frontier is shaped like.  How many connections, how much throughput, running for how long before the machine is exhausted?  What is this like for a few representative box types?  I don't know, and want to find out.
