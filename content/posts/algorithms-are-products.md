---
title: "Algorithms are Products, not Projects"
date: 2021-10-16T08:44:55+08:00
tags: ["datascience", "mlops"]
---

## TL;DR

In order to maximize value for business, algorithms should be thought of as Products instead of Projects.  This should include initial focusing on a algorithmic success goal that is equivalent (not better) than human performance and an architectural structure that expects continual improvement via algorithm versions over time.

## The Value of Algorithmic Operating Models

The power of business scalability with near-zero incremental marginal cost is hard to over-state.  There is a distinctively different valuation of businesses that scale well (e.g. near zero marginal cost) vs. businesses that have a fixed cost model.

> The market cap per employee of a highly scalable business model like Facebook is nearly 110X that of a human-intensive/agency (and thus fixed cost) business model like WPP.

Algorithms are one vector that can be applied to an existing business model to reduce marginal expansion cost via the automation of decision-making.  This alone would be a worth-while pursuit, but algorithms can also typically operate at a lower-level granularity that enables increased operational efficiency which can translate into in-market competitive advantage.

## Experimental Nature of Algorithm Development

While I believe that the current state of Data Science has *significantly* lowered the barrier to entry via [maturation of ML frameworks](https://scikit-learn.org/stable/) and [emergence of transference learning](https://www.tensorflow.org/hub/) there is still an experimental component to algorithm development.  In particular, most algorithmic approaches have several hyper-parameters that can be specified to determine "how" the model is learned.  This may involve setting particular parameters for:

* How quickly the algorithm learns (and thus possibly over-shoots)?
* How long the algorithm should train for (and thus possibly over-fits)?
* What optimization approach will be used to reduce error?
* What architecture will be used (e.g. neural networks)?
* What metrics will be optimized against?

The number of hyper-parameters and potential values per each parameter may define a huge exploration space for a Data Scientist.  Given that each hyper-parameter may have significant impact on the performance of the resulting model it is possible that in high-value scenarios a Data Scientist could search for an optimal approach for an infinite period of time.

## Initial Goal: Human Comparable Performance

Given the fact that 1) business ecomomics can be radically transformed by automation as a reduction force on marginal expansion cost, and that 2) the "search" for best possible hyper-parameters may grow exponentially for very high-performing algorithms, the initial objective of algorithms should always be establishing automated decision-making that is comparable to existing human decision-making.  At this point value can be unlocked via operating model transformation but also via establishing further downstream automation that was constrained by existing, manual decision-making processes.

> Temptation to vastly exceed current performance benchmarks in decision-automation as time-lines may expand exponential due the exploratory nature of model hyper-parameter discovery.

## Establish a Product Mentality, Backed by Rapid Versioning/Deployment

Instead of a "one-time" project approach where decision-making goes from a human-based approach to an automated approach that significantly outperforms human base-lines, we should establish an initial (v0) model that matches human performance but then can be rapidly followed by a (v1), (v2), (v3) algorithm that continues to build improvement over base-line (v0) approach.

A Product-oriented approach has several key impacts that need to be understood at the beginning of the model lifecycle:

1. Data Science staffing for model developement is likely long-term driven by ongoing innovation (i.e. model versioning) needs as well as model maintenance
2. Model deployment processes and infrastructure needs to be compatible with quick-release requirements (e.g. CI/CD driven, cloud-based, etc.)
3. Model architectures need to be highly compartmentalized so that changes can be made across the algorithm ecosystem easily without fear of broad, negative impacts
4. Clear performance criteria alignment is required to provide "Go/No Go" approval for new model release candidates

The above are consistent with trends in generalized software development and the establishment of micro-service/DevOp best practices and so are not necessarily unique to algorithm development but are often overlooked in the algorithm space which is often treated more inline with historical "BI" projects that are one-off and have less deployment concerns.
