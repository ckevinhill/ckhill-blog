---
title: "Automation is Governance"
date: 2021-07-25T08:44:55+08:00
tags: ["other"]
---

# TL;DR

Efforts to automate data pipelines and machine learning projects are not just about efficiency - but are also key drivers of increased governance and stewardship.

## A Losing Strategy

In majority of today's Data Science organizations, there is a lack of standardization with respect to deployment and development practices.  This leads to a "starting with a white sheet of paper" syndrome which complicates all downstream processes (data quality monitoring, deployment, model risk review, etc.).  Typically this is addressed via multiple-layers of human-driven governance.  Data Scientists then submit forms to get approval for cloud resources, sign-ons off data access, even minutia of security group creation.

Ultimately this results in a downward spiral:

* Data Scientists blame platform/governance for slow speed of deployment.
* Governance organizations feel combative with Data Science organization and (rightly) focus on pointing out compliance issues.
* Data Scientists try and find ways "around" governance processes as coping methods.
* Governance/Engineering organizations add additional layers to prevent work-arounds and close holes in process.
* Repeat ad-nauseum

This can completely bottleneck organization efforts to shift to data-driven/algorithm-driven business processes as well as starve your innovation pipeline.

## Root Cause

Ultimately, this is driven by a lack of standardizaion in Data Science projects.  When you have not standardized your project structure, supporting platform and deployment architectures you are completely dependant on a human-driven (form-based) approach for governance over-sight.  As indicated above, this will almost always lead to a non-ideal outcome.  The Data Science organization should take the lead to define these standards ("eat your own dog food") in conjunction with supporting AI Engineering/MLOps organizations.

## Initial Focus Areas

I don't think you can overstate the advantages of automation in terms of deliverying increasing efficiency AND stewardship for ML projects.  Areas that you should look to automate include:

* Project creation - automate the startup of projects based on standardized project structures/code-bases, library usage, and integration with common DevOps tools (linting, unit testing, etc.) and process tools (Jira, Confluence, etc.)
* Deployment architecture - provide "blueprints" for ML architectures that can be selected "off the shelf" for deployment.  Build network security and access requirements directly into these pre-approved architectures.
* Establishment of CI/CD pipelines - automate the creation of CI/CD pipelines for all launched projects so that ongoing deployment is standardized and "zero effort" for Data Scientists (this links your standard project to your standard deployment architcture)
* Model Catalog population - automate the collection of model meta-data into Model Catalog at creation time.
* Model performance monitoring - automatically integrate ongoing model telemetrics or provide access to standardized libraries to support model telemetric collection.

Most of the above _should_ be considered a requirement for any modern ML project but are often dropped due to capacity or knowledge constraints.  Automation (and standardization) can turn this from this from a low-compliance, high-effort activity into a delighter for Data Scientists, Engineering and Governance organizations.  This is likely not a one-off effort but wil require continual organization effort to continue to reduce manual effort over time.

## Trade-off

With standardization comes some limitations on what a Data Scientist can do.  In general expect that the "standard" solutions will support 80% of your efforts but there will be another 20% that don't fit into standard project structures or deployment architectcures.  For these projects you should fall back to default custom behaviors and work with AI Engineering organization to deploy a be-spoked solution.  

As more projects are supported via standardized approaches, AI/ML Engineering resources should be freed to support more of these high-value, bespoked use-cases vs. spending time fighting fires/keeping lights on for all projects in deployment.

## Potential Barriers

### Available Capacity

DS/AIE organization are so over-taxed in fighting fires for non-standard solutions that there is no capacity to build standardized approaches.  Leadership needs to either find incremental funding for external support or prioritize current workload.

### Experimentation vs. Scale

Data Science often prioritizes rapid experimentation and believes that standardization will limit ability to experiment.  Capability needs to bring clear automation benefits (faster projects) for both experimentation and production use-cases.  The false narrative that "doing things right" means "doing things slower" needs to be combatted and should be replace with the expectation that even experimentation should be "fit for production" from Day 1.

### Internal processes

It is un-realistic to believe that you will be able to eliminate all existing internal processes from project life-cycle.  Cross-functional/organizational efforts need to be created to eliminate manual processes that often show up at the seams of automation efforts.  Expectations should be set that customized development will be required to automate data/process hand-off across organizations (i.e. the answer is not to buy an external "black box" solution that does not integrate with internal processes.)
