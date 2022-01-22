---
layout: post
title:  "Comparing Microservices and Monoliths"
permalink: "microservices-vs-monoliths"
date:   2022-01-17 20:01:06 -0400
---


Advantages of monoliths:
- run, test, trace, debug on a single computer
- centralized consistent state (reporting is easy)
- easiest to implement
- fast go-to-market
- allows for discovery of domain (doesn't require domain knowledge ahead of time)
- one repo, one pipeline, easy ops

Disadvantages of monoliths:
- changes require complete redeployments, so coordinating multiple deployments from multiple teams in a single day can be difficult
- hard to keep modular structure, results in high coupling = ball of mud
- scaling is uniform, can result in inefficient use of infrastructure
- limited to the language of the stack
- slow startup/build times

Advantages of microservices:

- independent deployment (multiple teams deploying their changes to a monolith per day is difficult, microservices allow you to deploy independently)
- independent scaling (if a service requires a lot of resources, it can scale without hindering the rest of the application or requiring inefficient use of hardware)
- firm module boundary through network calls enforces modularity which allows for better maintainability
- choice of languages/databases (best tool for the situation, hiring)
- organizational structure can follow microservices structure with clear responsibility over services 
- resilience & lifecycle (one service going down does not cause the entire application to go down)

Disadvantages of microservices:
- data consistency (data must be consistent across the whole system, no longer using a single ACID database and must go through service to ensure eventual consistency)
- distributed tracing & monitoring can make the application harder to debug than debugging within a monolithic application
- data warehousing (to provide reporting internally, you must pull data from many different databases into a data warehouse for use in PowerBI, graphs, etc.)
- complex ops (ops is more involved as there are more services to keep alive, deploy, more pipelines, more tools to learn)
- performance can be affected if microservices are overly chatty (network calls are slower than function invocations. If your microservices are overly chatty, your microservices may be too coupled and the boundaries of the microservices in question should be reviewed)
- testability is more complex since it is more difficult to set up the entire application on a single computer and requires following across multiple services, which results in a reliance on logs
- devops culture is often needed
