---
title: "A Pragmatic Approach to System Design"
date: 2018-11-04T20:00:00+00:00
draft: false
tags: ["software", "design"]
author: "Pedro Lopez"
---

![image](/images/a-pragmatic-approach-to-system-design.jpg)

Designing a new system is not an easy task. In this post I will introduce a pragmatic approach to system design that will help you tackle any system in a repeatable and consistent way.

<!--more-->

This approach consists of four sections or steps:

- Requirements
- Capacity
- Application layers
- Optimisation
- Final considerations

#### 1. Requirements

First clarify all the requirements. Ask your stakeholders. They could be a project manager, a product owner, a business analyst or even an interviewer (if that is what you are doing).

Ask as many questions as you need to explore the details. Focus on two different groups of requirements.

1. Functional: they specify something that the system should do, e.g. let user post tweets, send emails or upload photos.
2. Non Functional: they describe how the system should behave, e.g. in terms of performance, scalability, capacity, availability, latency or security.

Then take a moment to consider your design goals. Now is the time to balance some of those requirements and make compromises where necessary, e.g. consistency vs availability.

#### 2. Capacity

How many users does your system have? How much data do you need? Now it's the time to clarify those, apply basic Maths and jot down the numbers. Remember that 1 character in UTF-8 can be between 1 and 4 bytes.

Number of users, requests per second, gigabytes of memory... those are things that you are typically looking for here.

#### 3. Application layers

There will be three layers to consider in any system.

1. Data layer: if you have a database, which type? SQL/NoSQL/Graph... Do you need tables? Design the schema now if you need one.
2. API/UI: what kind of interface will your system have? a web UI or a REST API are some of the options depending on what your system does. You might even have both. Design the API now if you need one, what endpoints, how would you name them and what data will they accept and return.
3. Business logic: this is the meat in the middle, the code that actually solves the business problems that your project sponsor has. At this point you should consider different algorithms and techniques. If you don't know where to start, try first a naive approach and optimise later.

At this point you should be able to draw a basic block diagram of your system. Don't worry too much about the details just yet, there is still room for optimisation. Use a whiteboard if you can, as you will have to move things around a few times most likely.

#### 4. Optimisation

There are at least four things to consider:

1. Redundancy/replication: if you need high availability, you will need some kind of redundancy in the form of multiple nodes. Also, you will need to decide how the data will be replicated between those nodes.
2. Sharding: similar concept but now for your database. Depending on what DB technology you used, sharding might come out of the box. But in any case is good to take a minute to consider the options.
3. Caching: most likely some of the data in your application can be cached to reduce latency and improve performance. Look at different CDN options and in-app caching. Adding a key-value in-memory data store like Redis is a great option to consider.
4. Loadbalancing: finally, some of your services, depending on your capacity analysis, will require some form of loadbalancing so that the load is distributed between the nodes. Round-robin DNS and virtual IP are the most common ones.

#### 5. Final considerations

If you followed these steps, you should now have a pretty comprehensive draft of your system. Spend a few minutes going through all the information that you collected and produced, making sure you covered all the requirements.

Think about whether you will need any special kind of testing. Unit, integration, functional, load and security are the most common ones, and most likely you will need all of those.

Consider aspects like infrastructure-as-code, containers, blue/green deployments and continuous integration. Your system should be compatible with these practices to be easily deployable and maintainable.

It's worth at this point considering also how many resources you would need to build the system, in terms of teams, time and budget.

Don't forget to include security in your design, do you need an API gateway? will your services be internal or exposed to the public? Are you using basic auth, an API key, JWOT tokens...

And remember to share your design with other people! It's very easy to miss something and there is no silver bullet when it comes to system design, so a second pair of eyes is always useful.
