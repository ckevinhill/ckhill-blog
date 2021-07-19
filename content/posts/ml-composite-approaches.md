---
title: "Composite Approaches in Machine Learning deployment"
date: 2021-07-14T17:02:02+08:00
draft: false
tags: ["datascience", "devops"]
---

# TL;DR

Today's IT-driven competitive advantage lies in the INTEGRATION of data and processes that have historically been silo'ed within the enterprise.  Machine Learning benefits from this integration (data) and from a composite architecture (integration into digitized business processes) but needs to adopt MLOps practices and a _composite_ deployment mindset to meet the expectations of a modernized, service-oriented IT architecture.

# Hyper-connected IT Architecture

IT practices have advanced to the point where digitalization of business processes within specific silos is a default expectation and no longer considered sufficient to deliver competitive advantage.  Therefore more ephasis should be applied to the integration of processes across silo's (marketing+sales, supply chain+marketing, consumer research+product, etc.) in order to extract competitive advantage.  To support these more "hyper-connected business operations", IT architectures have begun to require a _composite_ mentality where all solutions are developed by stitching together re-usable components (microservices, apis, etc.):

![hyper-connectivity](/images/hyper-connected-architecture.png)

The adoption of composite architectures has impact on IT product development decisions including the obvious increased adoption of cloud-native deployment capabilities and micro-service (and other SOA) design usage - but also increased valuation for custom-built/in-housed applications or service-enabled 3rd party applications that can be integrated into existing platforms.

# Benefits to Machine Learning

Integration efforts across enterprise domains typically yield increased HARMONIZED data for use in Machine Learning models which benefit Data Scientists both in improvement in existing model performance but also in expansion of potential addressable use-cases.

Additionally an IT organization pivot to composite architectures creates opportunity for Data Scientists to directly integrate algorithms into business operation platforms.  Compared to historical (BI-oriented) approaches where propertiary, in-house algorithms may produce a recommendation to a human operator via a dashboard vs. a composite approach where algorithm outputs can be directly fed into operation systems (either with human supervision or applied directly to automating a task) - the latter provides significant efficiency and compliance opportunity.

# Adopting ML Composite Strategies

To participate fully in the hyper-connected IT products, Machine Learning products need to adopt simliar world-views and be designed for cloud-based, api-accessibilty from Day 1.  There are (at least) 3 different composite models that can be explored for ML Products including:

## Library-based deployment

Models are deployed via pre-created training and inference functionality that can be installed into an application.  This provides significant flexibility to using application but likely works best for smaller data-sets that would not require specific backing infrastructure.  

__Example:__ provide an installable library that will pre-generate next best product recommendation for consumers based on a standard user, product purchase history.

## API-based deployment

Model inference and training endpoints are deployed as part of a cloud-based Pipeline (e.g. Azure Workspaces or GCP Kubeflow).  This provides the most typical reapplication of the microservice design concept but does require moving application data into the ML component space which may have security or cost implications.

__Example:__ trigger training pipeline via posting of new sales data to specified blob storage, call inference API endpoint to get future sales prediction for product using most recently trained model.

## Platform-based deployment

Model is deployed as part of an end-to-end use-case where using application (or organization) provides configuration and platform implementation is responsible for collection of data and possibly execution of operation.

__Example:__ inventory consumption prediction algorithm is provided with configuration (max bid, buffer size, etc.) as well as security permissions (tokens, etc.) to access current inventory warehouse.  Algorithm predicts requirements and submit orders to down-stream systems and provides summary of operations to planning system.

# Increased requirements for MLOps

Additionally strong ML-Ops practices need to built into ML products that will likely be deployed broadly with multiple potential use-case applications - efforts against documentations, Feature and concept drift monitoring and model cataloging become increasing valuable in a hyper-connected environment but are often deprioritized by Data Science organizations.
