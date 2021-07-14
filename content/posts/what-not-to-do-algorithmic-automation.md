---
title: "5 Mistakes to Avoid in Algorithmic Automation"
date: 2021-07-13T17:02:02+08:00
draft: false
tags: ["datascience"]
---

As part of a talk given to [The American Chamber of Commerce in Singapore](https://www.linkedin.com/company/amchamsingapore/) on Digital Transformation I outlined 5 common mistakes that I have seen made as part of integrating automated decision-making (algorithms) into digital transformation efforts.

## Aiming at the Wrong Target

Most digital transformation projects should expect there to be 3 distinct program phases:

* Establishing the digital platform - this includes understanding, digitalizing and beginning operations of a digital work process.
* Establishing KPI visibility - providing dashboards/BI capability for analysts to generate insights out of data captured in platform.
* Establishing automated decisioning - integrating algorithms to automate process decision-making.

This may seem intuitive to many, but I still seem projects stumble in 2 main ways:

First, business leaders may be overly optimistic on algorithmic automation
and try to immediately push the project to deliver this outcome.  Unfortunately this rarely works due to flawed/weak foundations (data, process
integration, etc.) and typically necessitates the project "rebooting" to focus on these fundamentals.

Second, short-term business leaders claim success with the emergence of dashboards (which can create transformation value) and never proceed to
the development of automated decisioning reducing the total value of the effort.  Typically this is driven by a lack of cultural evolution within the executive suite within a company as leaders have not yet adopted an "algorithm-first mentality".

*Take-away:* Projects should set algorithmic automation as the success criteria from Day 1, but should expect to progress through foundational platform and BI phases before tackling algorithm implementation.

## Relying on Algorithmic Complexity

The vast majority of improvement in your algorithms performance will come through data improvement and feature creation, yet may projects at setup expecting increasing algorithm complexity or tuning to drive progress to goals.  Typically this is seen through staffing imbalance where Data Science resources are tasked
with algorithm performance metrics with little/no support for Data Management activities.

If you are not talking about data quality as part of your algorithmic automation you are significantlly decreasing your chances of success.  If the resources that you are working with are talking to you about increasingly complex algorithms and not talking about data quality, then they are probably more concerned with selling you something than helping you...

*Take-away:* Data management (cleaning, structuring & documentation) is hard work but is the most effective way to improve algorithm performance.

*Take-away:* Data management is hard and messy, if you aren't feeling pain someone is hiding something from you.

## Forgetting Operations

Historically Data Science has been most often associated with experimentation (crazy scientists in a lab), however if we are moving to a world where an organization has thousands of algorithms deployed to make business decisions daily we need to increase the importance of MLOps within the Data Science culture and skill-set (assembly line with quality assurance).

[Expectations for Data Scientists need to be raised to enable increased speed to deployment](https://thuijskens.github.io/2018/11/13/useful-code-is-production-code/):

>Since data science by design is meant to affect business processes, most data scientists are in fact writing code that can be considered production. Data scientists should therefore always strive to write good quality code, regardless of the type of output they create. Whatever type of data scientist you are, the code you write is only useful if it is production code.

This should be done in a way that does not slow down experimentation which Data Scientists will still need to do on data and algorithm feasibilty, but which accelerates scale and quality assurance efforts.

Data Science organizations should establish:

* Consistent development methodologies to reduce redundant development and enable seamless hand-off to AI Engineering organizations.
* Automated or platform approaches for model risk managemend and governance.
* Skill-sets that incorporate DevOp best practices and cloud-native technologies.

*Take-away:* Expect Data Scientists to produce outputs that can be productionized.

## Set and Forget

Unlike traditional IT projects that can be deployed and operated with little change, model quality will vary with externalities that are not controllable.  Consumer behaviors, external contexts (e.g. pandemics), and business model changes will fluctuate meaning that models performance may degrade.

2 most common issues that need to be addressed include:

* Input/Output Feedback loops - where outputs from your prediction may impact systems that are also inputs into prediction creating unexpected signals.  Example: Google Flu Trend predictor where increased news/searches for information about the PREDICTOR made the predictor think that prevalence of flu was increasing).
* Historical trend disruption - where discontinuous historical trends invalidate previous data.  Example: Pandemic events like Covid that change purchasing and travel behaviors making previous patterns invalid.

*Take-away:* Safe-guards need to be establishing to catch model degradation over time and expectation for continual updates/corrections should be established at project beginning.

## Ignoring the Role of Humans

Decision-automation is a significant cultural change that requires top-down and bottom-up support.  Critical to define what roles existing resources should fill and provide required up-skilling and incentives.

Today, many knowledge workers spend significant type in executional tasks that generate little additional innovation value (e.g. copying campaign settings from last month into this month).  Algorithms can effectively take-over these tasks and operate them at a more granular level than humans (e.g. moving from regional to neighborhood level of planning) but it raise the question on what should the humans do then?

Deliberate efforts should be incorporated into project plans to elevate the value of humans redirecting freed time such that knowledge workers go from owning execute to owning strategy, go from focusing on tactics to focusing on success criteria definition and measurement plans, and go from having limited time for experimentaiton to innovation/experimentation being a required role expectation.

For P&G, tomorrow's Brand Builders and Sales teams probably look alot more like today's Analysts in terms of skills and data fluency.  Having them spend disproportionately more time on quantitative success definitions (which algorithmis will optimize against) and experiments that are counter to ML recommendation (which will generate more data for learning algorithms) will create net impact larger than either humans or machines operating alone.

*Take-away:* Organizations need to define the role that current knowledge workers will play in the age of algorithms and begin building required skillsets and talent.
