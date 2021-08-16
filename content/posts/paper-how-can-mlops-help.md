---
title: "[Review] Who Needs MLOps: What Data Scientists Seek to Accomplish and How Can MLOps Help?"
date: 2021-08-15T08:44:55+08:00
tags: ["paper", "mlops"]
---

## Paper

[Who Needs MLOps: What Data Scientists Seek to Accomplish and How Can MLOps Help?](https://arxiv.org/pdf/2103.08942.pdf) provides survey response from 331 Data Scientists to determine current work focus and barriers.

## Data Points

* 40% respondents say that they work with both models and infrastructure.
* 37% indicate they will spend time in next 3 months on production related dev or deploy processes.
* The biggest issue called out continues to be data accessibilty and quality (~50% top 2 box on significant challenge).

There is some skew on concerns based on team size (e.g. smaller teams tend to be more concerned with data issues whereas as teams get larger there are more operational, deployment concerns).

## Interesting Conclusions

The paper highlights the CD4ML approach which has clear designs between:

* Data preparation - owned by Data Engineering
* Model development - owned by Data Science
* Model deploymnent - owned by Application Developers

However it notes that increasingly Data Scientists are becoming more invovled in Architecture and Deployment and the definition of "Full Stack" Data Scientist is not fully defined in industry.

Additionally the paper talks about a dichotomy in Data Science work process where early stages tend to be "waterfall" (data exploration) and later stages are "agile" (deployment and feature improvement).  It is a consideration that early phases of DS project work "are about understanding the data" which may be handled differently than typical agile/software project delivery.

There is a potential disconnect however in these process defintions and the objective of CD4ML which indicates:

```text
a cross-functional team produces machine
learning applications based on code, data, and models in
small and safe increments that can be reproduced and reliably
released at any time, in short adaptation cycles
```

Additionally there is a realization that ML components need to integrate and be deployed in the context of modern application development such that:

```text
often the model, which can be the core of the application, is
just a small part of the whole software system, so the interplay
between the model and the rest of the software and context is
essential
```

The paper indicates a maturity progress for Data Science organizations where:

* _Data-centric_ - focused on cleaning up the data
* _Model-centric_ - focused on learning how to bulid models
* _Pipeine-centric_ - focused on building factories to operationalize ML (at this point considered business critical)
