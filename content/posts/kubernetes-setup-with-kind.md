---
title: "Local Kubernetes Setup with Kind"
date: 2020-08-13T19:54:38+08:00
draft: true
tags: ["devops"]
---

Within my P&G Data Science team I would like to:
1. Increase our ability to re-use algorithmic components globally
2. Speed up our deployment from development to production
3. Enable direct algorithmic service integration in cloud-based applications

As part of achieving these goals we are exploring more modernized deployment architectures - in particular Kubernetes.  One big question I had was how to setup a local Kubernetes environment so this post reviews the steps required to establish a local cluster with associated dashboard and metrics reporting on my Win10 developement machine.

### Kubernetes background

First - a litle background on key Kubernetes concepts and terms.  Kubernetes is comprised of:
* Master server - provides orchestration to coordinate the hosting of mulitple container-running nodes.
* Pods - the "atomic" unit within Kubernetes comprised of 1 or more containers that are tightly coupled.  Conceptually a pod can be thought of as an application where the containers have a shared life-cycle, volumes and IP space.
* Replicas - identical copies of a Pod that are managed by a replication controller and scaled up/down and self-healed as needed.
* Namespaces - organizational hierarchy for grouping Pods.  Common namespaces include default, kube-public and kube-system.  Kube-system is used for resources that are created by the Kubernetes system.

Most often users create "Deployments" via YAML files that define the replicas and services that need to be deployed across the cluster nodes.  For more detailed information you can review this [Introduction to Kubernetes](https://www.digitalocean.com/community/tutorials/an-introduction-to-kubernetes).

### Selecting Kubernetes host environment

There are several applications that can be used to host Kubernetes clusters including:
* [Minikube](https://kubernetes.io/docs/setup/learning-environment/minikube/)
* [Docker Desktop](https://collabnix.com/kubernetes-dashboard-on-docker-desktop-for-windows-2-0-0-3-in-2-minutes/)
* [Tilt](https://tilt.dev/)
* [Kind](https://kubernetes.io/docs/setup/learning-environment/kind/)

The internal P&G Data Engineering team has recommended Kind (which stands for Kubernetes in Docker) for our local development environment.  I have not explored the other applications but all seem more than sufficient to support my local development needs.

### Installations

Before we can launch our Kubernetes cluster we need to do multiple installations including:

| Command                        | Description                                                                        |
---------------------------------|------------------------------------------------------------------------------------|
choco install docker-desktop     | Docker provides the environment used by Kind to setup Kubernetes cluster and nodes |
choco install golang             | Kind is developed in Go and requires Go run-time                                   |
choco install kubernetes-cli     | The Kubernetes "kubectl" CLI that enables access/control of Kubernetes cluster     |
choco install kind               | The Kind application to enable Kubernetes deployment to Docker                     |

>In order to support the Kubernetes cluster and potential deployments it is recommended that you configure your Docker Desktop to have a minimum of 4 cores and 8 Gb memory via the *Settings / Resource* options.  You can use `docker stats` in Powershell to check current Docker resource consumption if concerned about resource limitations.

### Creating our first cluster

With Kind and other installations completed we can create our first Kubernetes cluster via `kind create cluster`.  This will create an initial cluster named *kind*.  If you want to create an additional cluster or override the default you can use the --name option.

You can use `kind get clusters` to confirm that your cluster has been created successfully and is running.  Additionally you can run `kubectl version` to confirm that your kubectl CLI is able to connect to your server.  You should be able to see both client and server versions.

As a side note, if you need to completely delete your Kubernetes cluster you can use `kind delete cluster`.  This has become hung at times for me, requiring a restart of Docker Desktop to complete deletion.

### Deploying Kubernetes dashboard

Having a dashboard for Kubernetes is helpful to review Pods and resource usage without having to rely solely on the CLI.  You can directly deploy the Kubernetes dashboard via: 

`kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml`.

This downloads the dashboard deployment YAML and executes deployment within the Kubernetes cluster.  You can now view your deployed Dashboard Pod via `kubectl get namespaces` and `kubectl get pods -n kubernetes-dashboard`.

>Almost all kubectl take the *-n* namespaace option.  If you get tired of adding -n to all commands you can set a default namespace with `kubectl config set-context --current --namespace=kube-system`.

To access our new dashbaord we need to launch a Proxy that allows access to the Kubernetes network.  This can be done via `kubectl proxy` which will enable you to access the dashboard via:

`http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/`

>When you first access your Dashboard you will be greated with a message requiring you to select a method of authentication.  You should see *Every Service Account has a Secret with valid Bearer Token that can be used to log in to Dashboard. To find out more about how to configure and use Bearer Tokens, please refer to the Authentication section* as one of the options.

In order to provide an authentication token we will:
1. Create a cluster service account named *dashboard-admin-sa* via `kubectl create serviceaccount dashboard-admin-sa`
2. Give the account admin-level permissions via `kubectl create clusterrolebinding dashboard-admin-sa --clusterrole=cluster-admin --serviceaccount=default:dashboard-admin-sa`

We can now get generated a list of accounts that have generated secrets using `kubectl get secrets -n default`.  From this list you can then export the service account token using `kubectl describe secret dashboard-admin-sa-token-tcxxl`.  The *dashboard-admin-sa-token-tcxxxl* will need to be replaced with value from your system.  You can also access token in one line with `kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | sls admin-user | ForEach-Object { $_ -Split '\s+' } | Select -First 1)` in Windows Powershell.

Pasting this token value into the Dashboard should now give you access to overview and reporting on your Kubernetes cluster:
![Dashboard image](https://miro.medium.com/max/700/1*Cewl4uR4rOeabli-zfN7NA.png)

### Adding resource metrics

The dashboard provides overview of Pods deployed and current Pod health status but does not provide information on CPU and memory usage.  To enable this metric tracking we need to also deploy the [Kubernetes Metric Server](https://github.com/kubernetes-sigs/metrics-server).

Unfortunately when using Docker Desktop there is an issue where [TLS certificates are not created](https://github.com/docker/for-mac/issues/2751#issuecomment-419676829) in a way [to enable the Metric Service to work](https://blog.codewithdan.com/enabling-metrics-server-for-kubernetes-on-docker-desktop/).

As a workaround we can download the metric-server YAML file from `https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.6/components.yaml` and modify it with the `kubelet-insecure-tls=true` argument which allows the container to ignore the TLS issues.

I have incorporated this change into YAML file at [https://github.com/ckevinhill/ckhill-blog/blob/master/static/files/metrics-server.yaml](https://github.com/ckevinhill/ckhill-blog/blob/master/static/files/metrics-server.yaml)

You can now deploy the metrics-service within the Kind cluster via `kubectl apply -f https://github.com/ckevinhill/ckhill-blog/blob/master/content/resources/file/metrics-server.yaml`


### Alternative approach for deployments

Instead of downloading and applying YAML files we could also use the Kubernetes "package manager" named [Helm](https://www.bmc.com/blogs/kubernetes-helm-charts).  *Note:  As of Helm 3.0 there is [no longer need to install the back-end Tiller service](https://helm.sh/docs/faq/) and Helm permissions are managed via build-in RBAC controls*.

You can install helm with `choco install kubernetes-helm`.  You can then add the official stable repository with `helm repo add stable https://kubernetes-charts.storage.googleapis.com/` and update helm with `helm repo update`.

You can search for the metrics server with `helm search repo metrics` and should see something simliar to:
![console-output](/images/ps-helm-repo-search.png)

Finally you can install the metrics-server with `helm install metrics-server stable/metrics-server --namespace kube-system`.
>Unfortunately I was unable to pass the *--kubelet-insecure-tls=true* argument via helm even though documentation seems to indicate that you can so I was unable to use this appraoch for metric server deployment.


### Accessing metrics

You can access the metrics directly within the Dashboard or you can query the metric-server API direclty with `kubectl get --raw "/apis/metrics.k8s.io/v1beta1/nodes"` or see node stats with `kubectl top nodes`.

![Dashboard with metrics](https://docs.aws.amazon.com/eks/latest/userguide/images/kubernetes-dashboard.png)


### Debugging

I may post more information on debugging Kubernetes nodes in the future but for now I'll just leave a few potential commands for exploration:
* ` kubectl -n [namespace] logs [pod name]`
* `kubectl exec --stdin --tty [pod name] -- /bin/bash`

Congrats - you should now have a Kubernetes cluster up and running!  We have also covered how to do deployments via pre-created YAML files and via the helm package manager.