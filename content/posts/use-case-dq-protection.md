---
title: "Data Quality Should be Tied to Use-Case"
date: 2021-09-19T08:44:55+08:00
tags: ["devops", "other"]
---

### A Tale of Two Data Quality Checks

This was a [post](https://www.linkedin.com/posts/eczachly_dataengineering-data-activity-6845474719169343488-lowf) on LinkedIn discussing the challenges for Data Engineers in deciding what level of Data Quality checks should be implemented:

![post](/images/lin_post_de.jpg)

One of the things that struck me was the last line:

> That’s why unit testing and validating things like nullability/uniqueness of columns are much more important than anomaly detection in data pipelines.

To me this speaks to a dichotomy between 2 types of thinking:

* Collecting data for the sake of collecting data
* Collecting data for the sake of a use-case

When considering a use-case like an algorithmic prediction, systemic anamoly detection may be _more_ important than a few null values.  Extrapolated more generically I would say that there should always be some sort of Data Quality check that is define _with respect to a use-case_.

### Divide and Conquer

But how would this be executed?  It is not feasible or fair to have Data Engineers attempt to know the details of all use-cases - particularly if the Data Engineer is maintaining a corporate data asset.  

Therefore it should be considered a part of every Product/Application owner to provide Data Quality checks that are specific to their use-case (statistical or otherwise).  

Data Engineering can focus on structural checks as a base-line, but ultimately the Product/Application Owner should be accountable for incremental Data Quality checks that go beyond these basic measures.  Product/Data Science teams should staff and plan with this [responsibility](/posts/ds-feature-quality/) in mind and Data Engineering organizations should drive transparency in scope and details of what is included in "base" Data Quality assessment.

### Implementation

Where should these checks be implemented?  There may be a desire to try and push Data Quality checks defined by a specific Product Owner into the Data Engineering organization for ownership as there is always a desire to scale.  However this sacrifices agility for the Product Owner and moves accountability out of the Product Team into a central function which may lead to prioritization or communication challenges.  Additionally checks for one use-case may not be valid for other use-cases leading to potential issues.

In order to maintain ownership and agility use-case Data Quality checks should be implemented and maintained within the Product scope (i.e. within the ML Pipeline or Application API, etc.) with only base/foundational measures that are truly universal being scaled within centralized Data Engineering ETL pipelines.
