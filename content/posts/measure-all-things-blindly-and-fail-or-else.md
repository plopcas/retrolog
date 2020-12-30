---
title: "Measure all things blindly and fail, or else?"
date: 2020-12-30T18:00:00Z
draft: false
tags: ["management"]
author: "Pedro Lopez"
---

![image](/images/measure-all-things-blindly-and-fail-or-else.jpg)


If you want to improve, you need to measure how you are doing. You need data. This is typically one of those things organisations get wrong from the get go, meaning:

- not enough data is being captured,
- the data available is useless,
- or worse, the data available is harmful.

<!--more-->

A perennial example is lines of code (LoC). Martin Fowler [wrote about this](https://martinfowler.com/bliki/CannotMeasureProductivity.html) back in 2003. He said, “false measures only make things worse”. Agreed.

I personally like what Pat Kua describes in [this post](https://martinfowler.com/articles/useOfMetrics.html) from 2013. This is what he proposes:

1. Explicitly link metrics to goals.
2. Favour tracking trends over absolute numbers.
3. Use short tracking periods.
4.  Change metrics when they stop driving change.

This is good because it reinforces the idea of paying attention to what you measure, being critical about how you interpret data, experimenting in short iterations and evolving.

Being more specific, when talking about project management, my preference is to classify metrics in three buckets: project, product and process (coincidentally 3P).

**Project metrics**

Used to understand whether the delivery of a project is at risk, and whether we are still aligned with our goals. E.g.:

- Weekly commitments
- Project milestones
- Key results ( if you use Objectives and Key Results - OKRs)

Can usually take one of “on track”, “at risk” or “achieved”. In the case of key results, they can be more specific.

**Product metrics**

Used to understand how the product evolves over time, and incredibly useful during experimentation. They are totally context dependent e.g. login success rate, bounce rate, etc. It depends on what matters to your product.

(Tip: talk to your product people to define and understand these as soon as possible, run!)

**Process metrics**

Used to understand how the team goes about work. E.g.:

- Cycle time: how long does it take from picking up a task to it being in production.
- Throughput / velocity: how many tasks / velocity points are done in a period of time.
- Work in progress: how many tasks are started but not completed.
- Work item age: how long do tasks remain in a particular stage.

Particularly with process, I believe it’s important to use metrics that complement each other by eliminating blind spots. For example, if we measure _work in progress_ in isolation and we expect this number to be low, we might be favouring bottlenecks, harming psychological safety and discouraging parallelisation. On the other hand, if we expect this number to be high, we might struggle to get things done, lose focus and pay a high cost for context switching.

However, when combined with other metrics we get closer to a 360° vision. _Work item age_ can help us identify blockers. _Velocity_ can help us plan for holidays or sickness and identify trends in stable teams (are we slowing down or speeding up and why). _Cycle time_ can help us find inconsistencies in our estimates.

**Other**

There are a lot of other metrics that can sometimes be useful too, for example when _refining_ a process. Sometimes they are simply interesting. If the cost of capturing them is low, I’d say go for it. But be careful with sharing those without context, as it’s very easy to mutate a well-intentioned metric into a harmful one. Some examples I’ve used in the past:

- Language / framework (useful in order to have an overview of skills required for example)
- Code category (back-end, front-end, mobile)
- Change type (new feature, bug fix, refactoring)
- Test coverage
- Pull request elapsed time
- Pull request throughput
- Pull request size
- Rollback rate
- Roll-forward rate (when an issue is fixed by deploying new code vs rolling back latest change)
- Number of production incidents by impact
- Cost per service (in the context of project management, to make sure we are within budget)

Only recently I learned about code risk metrics i.e. what areas of the code are more at risk if a core maintainer leaves the organisation ([CodeScene](https://codescene.io/)). Another twist to the more traditional static analysis tools, like SonarQube or Codacy.

Be wary of using metrics to compare different teams, though. For the most part, metrics should only be used to compare a team with itself, with a focus on continuous improvement. They are very much context dependent and its interpretation requires a critical but flexible eye (favouring tracking trends over absolute numbers ☝️).

Of course, there's a whole set of operational metrics to monitor our services, infrastructure and security. But that’s a different beast for a different time.

Thanks for reading!
