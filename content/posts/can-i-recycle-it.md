---
title: "CIRI Application - Kubernetes (GKE) + TensorFlow"
date: 2020-12-12T06:42:55+08:00
tags: ["datascience", "gcp", "mlops"]
---

This project was produced as part of the final project for [Harvard University’s AC295 Fall 2021](https://harvard-iacs.github.io/2021-AC215/) course.

![waste-picture](https://ec.europa.eu/environment/sites/default/files/styles/oe_theme_medium_no_crop/public/2021-01/Homepage_cropped.jpg?itok=mFWMtHR_)

## Context

Waste management is one of the most challenging problems to solve in the 21st century.

> The average American produces ~1,700 pounds of garbage per year which is roughly three times the global average, according to a report by the research firm Verisk Maplecroft in 2019.

Poor waste management is linked to significant environmental risks, such as climate change and pollution will likely have significant long-term effects to our environment that will impact future generations.

The introduction of a "single-stream" approach to recycling where materials are not pre-sorted into common recycling classes like paper, aluminum, metal, and glass has greatly increased the rate of participation in recycling programs - but has led to [substantial incrases in contamination as well](http://mediaroom.wm.com/the-battle-against-recycling-contamination-is-everyones-battle/).

Recycling contamination occurs when non-recyclable materials are mixed with recyclable materials.  Current estaimtes are that [1 in 4 items inserted into a recycling bin are inappropriate for recycling](http://mediaroom.wm.com/the-battle-against-recycling-contamination-is-everyones-battle/).  [Contamination can inadvertently lead to](https://www.valleywasteservice.com/valley-waste-news/what-happens-if-you-put-non-recyclable-items-into-recycling-4034):

* Increased cost of recycling as more effort is required for waste sorting - which can lead to non-viable economic models for local recycling.
* Reduces over-all recycling possible via contamination of recyclable items to the point where they are no longer suitable for recycling.
* Potential damage to recyling equipment or danger to recycling plant employees.

In many cases consumers are unaware of the negative impacts of recycling contamination and are well intented by trying to add items to recycling bins.  Additionally the introduction of varied plastics and packaging has [led to increased difficulty in identifying recycable vs. non-recylcable items](https://www.nytimes.com/2021/09/08/climate/recycling-california.html).

__Our goal is to develop a prototype application that allows users to easily classify “recyclable” vs. “non-recyclable” materials via a Deep Learning model.__ The application will be comprised of a multi-tier architecture hosted on Kubernetes (GKE).  Application deployment will be via CI/CD integration into project GitHub repositories.

## Data

![recyclable-vs-non](/images/rec-vs-nonrec.png)

The dataset used for training included 2467 images from [Trashnet](https://github.com/garythung/trashnet/) challenge segmented into 6 human-annotated categories: cardboard (393), glass (491), metal (400), paper (584), plastic (472) and trash (127).  

Training dataset was expanded with additional curated images from the [Waste Classification v2 dataset](https://www.kaggle.com/techsash/waste-classification-data) which included images labeled as Recycleable, Non-Recyclable or Organic.  As the Waste Classification dataset was primarily gathered via web-crawling we curated a subset of the images and placed them within the (existing) trash category or in the (newly created) organic category.

Given the limited availability of annotated recyclables datasets, we will also look to provide ongoing enhancement of training data via incorporate a "user upload" capaabiltiy into application that allows end-users to directly annotate and submit images.

| Dataset | Labels | Quality |
| - | - | - |
| TrashNet | cardboard, glass, metal, paper, plastic, trash | Good |
| Waste Classification v2 | recyclable, non-recylcable, organic | Average/Poor|
| User Provided | cardboard, glass, metal, paper, plastic, trash, organic & other | Unknown |

## Model Selection

Several different transfer models were assessed for both size and performance.  Best performing models are listed below:

![experiment results](/images/ciri-exp-results.png)

Additionally several hyper-parameters were modified through-out experimentation including:

| Parameter | Optimized Value |
|-|-|
| Training/Validation Split | 80/20 |
| Decay Rate | 0.5 |
| Learning Rate | 0.01 |
| Max Epochs (w/ Early Stopping) | 15 |
| Kernel Weight | 0.0001 |
| Drop Out Weight | 0.2 |

Model build pipeline utilizes both Early Stopping as well as a LearningRateScheduler to achieve maximum accuracy without overfitting.

```python
optimizer = keras.optimizers.SGD(learning_rate=learning_rate)
loss = keras.losses.SparseCategoricalCrossentropy(from_logits=True)
es = EarlyStopping(monitor="val_accuracy", verbose=1, patience=3)
lr = keras.callbacks.LearningRateScheduler(
    lambda epoch: learning_rate / (1 + decay_rate * epoch)
)
```

Top layers added to downloaded transfer model were defined as:

![model layers](/images/ciri-model-layers.png)

1. Transfer Layer Base (non-trainable)
2. Dense Layer (124, relu)
3. Dropout Layer
4. Dense Layer (64, relu)
5. Dropout Layer
6. Dense Layer (# classes, none)

Current production model was selected to be `https://tfhub.dev/google/imagenet/efficientnet_v2_imagenet21k_ft1k_b1/classification/2` based on performance and size characteristics.  In particular the smaller size of the efficientnet models allow for a more cost effective architecture were model inference could be hosted on small resources due to lower memory requirements.  This ultimately saves platform cost but also improves inference speed for a better use-experience.

## Model Pipeline

An [end-to-end training pipeline](https://github.com/canirecycleit/model_training_pipeline) was implemented on the [Luigi](https://github.com/spotify/luigi) framework with the following DAG structure:

![pipeline](/images/pipeline.png)

A Tensorflow based pipeline is used to process the TFRecords, augment, normalize, build and fit model.  Model build results are stored in MLFlow via experimentation API and model registry API in order to allow a central application appraoch for model lifecycle management:

```python
mlflow.log_param("model_origin", "efficientnet_v2_imagenet21k_ft1k_b1")
mlflow.log_param("decay_rate", decay_rate)
mlflow.log_param("learning_rate", learning_rate)
mlflow.log_param("num_classes", num_classes)

history = training_results.history
mlflow.log_metric("accuracy", history["accuracy"][-1])
mlflow.log_metric("val_loss", history["val_loss"][-1])
mlflow.log_param("epochs", len(history["accuracy"]))

# Log label mapping for retrieval with model:
mlflow.log_artifact(label_mapping)

mlflow.keras.log_model(
    model, "model", registered_model_name="ciri_trashnet_model"
)
```

The Model Pipeline is deployed as a [Kubernetes CronJob](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/) scheduled to run weekly.  Models runs will be logged as subsequent model versions which can optionally be updated to "production" as desired by Application admins:

![cronjob](/images/ciri-model-2.png)

## Deployment Architecture

![ciri-architecture](/images/ciri-architecture.png)

The CIRI Application is composed of multiple Services deployed on Kubernetes.  By deploying on Kubernetes the CIRI application has significant resiliency and scalability "built-in" with an automated monitoring of Deployment health, and ability to auto-scale replicas of Deployments as needed.

| Services | Type | Description |
|-|-|-|
| [mlflow](k8s_deployment/kompose/mlflow-service.yaml) | LoadBalancer | Provides intra-cluster and external access to the [MLFlow](https://www.mlflow.org/) Experiment Tracking and Model Repository services. |
| [api](k8s_deployment/kompose/api-service.yaml) | NodePort | Provides backend APIs that enable IO operations on training data-set as well as execution of predictions.
| [ui](k8s_deployment/kompose/ui-service.yaml) | NodePort | Provides an HTML based front-end for user interaction with application.

| Deployments | Description |
|-|-|
| [mlflow](k8s_deployment/kompose/mlflow-deployment.yaml) | Deployment of [ciri_mlflow:latest](ghcr.io/canirecycleit/mlflow/ciri_mlflow:latest) Docker container. |
| [api](k8s_deployment/kompose/api-deployment.yaml) | Deployment of [ciri_apis:latest](ghcr.io/canirecycleit/backend_apis/ciri_apis:latest) Docker container.  |
| [ui](k8s_deployment/kompose/ui-deployment.yaml) | Deployment of [ciri_frontend:latest](ghcr.io/canirecycleit/frontend_ui/ciri_frontend:latest) Docker container.  |

Additionally a CronJob is deployed to Kubernetes to run the model-pipeline weekly.  Alternatively a model-pipeline "run once" Pod can be launched if an adhoc training run is desired:

| Pods | Description |
|-|-|
| model-pipeline | Deployment of model training pipeline via [ciri_model_pipeline:latest](ghcr.io/canirecycleit/model_training_pipeline/ciri_model_pipeline:latest) Docker container.  Pod runs one time and executes download of raw training images from Cloud Storage, processing/transformation of image files, training of model and registration of model in MLFlow model registry. |

### Kubernetes Infrastructure

Ingress to the Kubernetes-hosted application is provided via [Kubernetes ingress-nginx controller](https://kubernetes.github.io/ingress-nginx).  Controller mappings are provided via the following [definitions](k8s_deployment/kompose/ingress.yaml):

| Rule-Path | Service |
|-|-|
|/*| ui:8080|
|/api/*| api:8080|

Access to mlflow services is provided via external_ip:5000 as a hosted LoadBalancer component.

Application currently is designed for 2 node-pools to reflect a varied compute requirement for "always-on" application hosting vs. more intensive training:

| Node Pool | Description |
|-|-|
| default-pool | Always on pool that runs application services including mlflow, ui and api via default e2-medium instances.|
| training-pool | Auto-scaling pool of higher memory machines (e2-highmem-4) to execute training pipeline.|

The application also leverages two cloud storage resources:

| Cloud Storage | Name | Description |
|-|-|-|
| Image Store | canirecycleit-data | Provides storage of raw images used as input for training where "folders" reflect the classification. |
| Artifact Store | canirecycleit-artifactstore |Stores serialized metadata and model files from execution of model-pipeline. |

Initial Kubernetes (GKE) cluster provisioning can be executed via [Ansible](https://github.com/canirecycleit/ciri_app/blob/master/k8s_deployment/ansible/deploy-k8s-cluster.yml) or [shell scripts](https://github.com/canirecycleit/ciri_app/blob/master/k8s_deployment/create_cluster.sh) to enable an autoamted approach to infrastructure provisioning.

### Application Deployment & Updating

Individual deployment containers as well as deployment of Kubernetes application are automatically executed via [GitHub Actions](https://docs.github.com/en/actions).  K8s deployment Action can be found [here](.github/workflows/deployment.yml).

For individual Dockerized components (e.g. [ciri_apis:latest](ghcr.io/canirecycleit/backend_apis/ciri_apis:latest) the docker container is built on all merges of new code to master branch and made available via the GitHub Container Registry as a public image (no secrets or confidential information is stored within the Image).

For deployment of Kubernetes application (ciri_app) a request for deployment is generated with all changes merged to master branch.  

![deployment](/images/ciri-deploy-1.png)

Requests must be approved by an repository team member that is autorized for approval of the production environment.  When approved, the Deployment Action will either deploy all components or patch existing components depending on if components already exist or not within cluster.

![deployment](/images/ciri-deploy-2.png)

Deployments are set to pull new images so latest component Docker containers will be used on Deploy or Patch. Service Account (SA) secrets for deployment operations are stored within GitHub Environment Secrets for security purposes.

## Application Usage & Screenshots

### Application Home Page

A very simple MVP homepage that provides access to the core functionality to take a picture/upload an image for classification.  Additional navigation links are available to Upload a new image (particularly helpful if image has been miscategorized) and to learn more About the application - including links to the application GitHub repository.

![homepage](/images/ciri-ss-1.png)

Example of indicating that an Organic item is not recylcable:

![organic-no](/images/ciri-ss-2.png)

Example of indicating that Cardboard is recycable:

![cardboard-yes](/images/ciri-ss-3.png)

### Application Upload Page

Given that recyclables annotated image data is relatively limited and of varied quality we will determined that it was necessary to provide an ongoing way for users to enrich the annotated data-set so a basic upload form is provided to add new images to the CIRI Image data-store.  The form provides a drop-down to select from all defined classification categories as well as a catch-all "other" category.

![upload](/images/ciri-ss-4.png)

![success](/images/ciri-ss-5.png)

_Models are retrained weekly to take advantage of any newly uploaded images that have been added to the data-store._

### Model Management (MLFlow)

Weekly (or manually triggered) model builds are stored using [MLFlow](https://www.mlflow.org/) for both model build experiment tracking as well as model serialization as a model repository.  This allows continual training and model performance metric capture but an operationally controllable approach for "promoting" experimental models into Production.  CIRI administrators can log into the MLFlow interface, view models that are avaialble for use and then move that Model through a lifestage (staging, production, archival) as part of the application management process - all without having to redeploy any application code.

History of model run experiments:

![mlflow](/images/ciri-ss-6.png)

Current model registry versions in model lifecycle stages:

![model registry](/images/ciri-ss-7.png)
