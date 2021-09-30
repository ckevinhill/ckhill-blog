---
title: "In-Housed Data Science"
date: 2021-09-28T08:44:55+08:00
tags: ["datascience"]
---

## Out-Sourcing

In the early days of Global Business Service (GBS) out-sourcing there was an adage that I really liked:

> Only out-source what you deeply understand

This speaks to the fact that if you don't understand a domain then it will be extremely difficult for you to provide governance or direction for external parties that are acting on your behalf.  One common example of this is when organizations that are not familiar with good DevOps practices out-source application development work to 3rd parties.  In these cases you almost always end up with project over-runs and eventually a poorly structured code-base that is difficult to maintain and operate.

It is occasionally argued that in deeply technical areas that are "not core" to a company there are agility and/or cost benefits in out-sourcing.  For large companies in particular [this may be questionable logic](http://danluu.com/in-house).

Even worse, in many cases the desire to out-source is driven by a desire to avoid the complexities of _understanding_ a difficult problem.  Complicated business processes, externalities, data and integrations take significant time to unravel - however for the out-sourcing manager it can feel like an easy win to delegate this complexity out of their responsibility, effectively setting up out-sourced providers as scape-goats in many cases.

## Benefit of In-House Data Science

Related to general challenges with out-sourcing, for Data Science projects specifically there can be significant challenges with an out-sourcing strategy:

* Data Science often needs to enable business process transformation which requires internal process and organization understanding
* Data Science feature generation is heavily tied to business domain understanding
* Data sets are often of poor quality or have outages that require organizational escalation
* Data access is often limited and may have long lead times associated with approval processes

In particular for innovation/exploratory projects that are _fully_ out-sourced I often see the following outcomes:

* Out-sourcing manager does not engage on complexity eventually leading out-source vendor to "give in" and produce "something" to meet contract requirements (e.g. a model built with invalid or limited data or invalid assumptions, etc.)
* Model outputs are not tied to actual business process and thus have limited impact (e.g. optimize against a wrong of incomplete metric)
* Deliverables that have limited scalability (e.g. a bunch of Jupyter notebooks built on manually modified XLS sheets)
* Little documentation on feature engineering experimentation history leading to limited instutional understanding

Net, due to the inherent transformation/innovation nature of many Data Science projects combined with need for significant domain knowledge and potential "painful" escalation and push-back on scope and requirements, out-sourced resources may be poorly setup for success.  This is most often true when out-sourced resources work directly with business partners that may have limited experience with algorithms or data automation.

## Where can Out-Sourcing be applied in Data Science organizations?

Given the limited availabiltiy of Data Science resources, the difficulty in acquiring and maintaining a Data Science organization and the accelerating need to deliver algorithmic, decision-automation having a flexible, out-sourced resource pool does have value but these resources should be used in specific ways that limit the inheret downsides outlined above.

> For all of these approaches it is critical to protect the capacity of in-house Data Science resources to oversee project execution.  Amplification of in-house resources with out-sourced resource still requires >=20% of in-house Data Science effort in almost all cases.

### Model Operations

For deployed models that require monitoring and maintenance over an extended period of time, out-sourced resources can be applied to free capacity of in-house resources.  Contracted resources should be provided with clear documentation of model parameters and expectations and should have clearly outlined scope for modifications they can make (e.g. items like hyper-parameter tuning over time to address concept drift or structured feature engineering experimentation).  In-house Data Science resources should be maintained as Product Owners for deployed solution and should be able to provide Level 3 support and approve any major changes to deployed model that may impact model risk or performance profiles.

### Model Reapplication

For models that have been developd for a particular context (e.g. for a particular country or customer), out-sourced resources can be applied to replicate model pipeline for other highly-similar implementations.  For example, this can include executing pre-defined data quality sufficiency checks for a new country, running new country data through pre-established modeling methodology and providing results of performance vs. new country base-line and deployed country models.  In-house Data Science should still have responsibility to confirm model robustness before new model variant deployment.

### Innovation Staff Augmentation

For innovation projects there can be value in enrolling out-sourced resources to help executing feature-generation, experiments and implementation under the guidance of an in-housed Data Scientist.  For example out-sourced resources can execute hyper-parameter searches or can run experiments to quantify the up-lift of a proposed new feature.  Having this additional through-put can enable innovation projects to explore additional avenues that may not be feasible if constrained to in-house resources.  In-house Data Science resources should still be heavily involved in project and should have the capacity to provide clear, detailed direction and review results.
