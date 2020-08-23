---
title: "Basic Webserver deployment on Kubernetes"
date: 2020-08-21T09:09:28+08:00
tags: ["devops", "tutorial"]
---

In a [previous post](/posts/kubernetes-setup-with-kind/) we setup a basic Kubernetes cluster on our local development environment using *Kubernetes in Docker*, a.k.a KinD (which is a cute but extremely google-unfriendly name).

Now we will do the next simplest possible thing to make sure we understand the basics of how to use our Kubernetes cluster - deploy a [nginx webserver](https://www.nginx.com/).

### Create a persistent storage layer

First, we are going to create a persistent storage volume within Kubernetes.  This will allow us to attach storage to our containers that persists beyond the life of the container itself.  This can be really useful to centralize logging or to create a set of files that you want to possibly re-use in different container configurations.  For instance in this case we will store the html files that will be stored by our webserver on a persistent location.  This way we can start, kill, and redeploy our webserver without worrying about losing any content.  
> Another common approach to make your content or code avaiable within a container would be to store our html content in git and have our containers automatically download from git.  This is particularly useful if using a [GitOps](https://www.weave.works/technologies/gitops/) approach.

We will use the local filesystem of our Node (which is hosted on Docker) as our persistent volume:  
`docker exec -it kind-control-plane /bin/bash`  
`mkdir /mnt/log-data`  
`chmod a+rwx /mnt/log-data`  
`echo "Hello world" >> /mnt/log-data/index.html`

Create the Persistent Volume mapping in Kubernetes:  
`kubectl apply -f https://raw.githubusercontent.com/ckevinhill/airflow-kubernetes-example/master/create-persistent-volume.yaml`

> This is a very simple YAML file that just creates a persistent volume named *logs-pv* mapped to a local hostpath of */mnt/log-data/*.  Follow the link to the raw content in gitHub to review further.

### Create a namespace

To help make it easy to manage our webserver deployment create a new namespace via:  
`kubectl create namespace webserver`

### Create a persistent volume claim

To actually use persistent storage Kubernetes requires to you create a *claim* to that storage.  The cluster will automatically connect your claim to available storage (or create available storage if configured to dynamically create new persistent volumes).  
> As one would expect, claim storage request need to be less or equal to availble persistent volume for the claim match to be made.

Below YAML creates a claim named *webserver-html-pvc* in the *webserver* namespace:  
`kubectl apply -f https://raw.githubusercontent.com/ckevinhill/airflow-kubernetes-example/master/webserver/create-webserver-persistent-volume-claim.yaml -n webserver`

You can now see that the webserver-html-pvc has been bound to the persistent volume:  
![console-output](/images/webserver-example-kubectl-get-pv.png)

### Deploying webserver container

Now we can deploy our container containing the nginx image with /usr/share/nginx/html volumne mount path attached to the webserver-html-pvc claim:  
`kubectl apply -f https://raw.githubusercontent.com/ckevinhill/airflow-kubernetes-example/master/webserver/deploy-nginx-webserver-container.yaml -n webserver`

### Verification

`kubectl get pods -n webserver` should not show nginx-webserver 1/1 instance running.  

If we create a terminal into the container via `kubectl exec -it nginx-webserver -n webserver -- /bin/bash` we can now see the the *index.html* file we created on our host node in the */usr/share/nginx/html* folder: 
![console-output](/images/webserver-example-index-html.png)

Since this is the folder that nginx uses to server web content from we can also view the webpage by setting up a port-forward to the container with `kubectl port-forward nginx-webserver -n webserver 80:80` and then visiting the page at *http://localhost*.
![webpage-hello-world](/images/webserver-example-helloworld.png)

### Clean up

If you are done with your example webserver you can delete it by completley erasing the webserver namespace via `kubectl delete namespace webserver`.  

After this delete you will see that your namespace (`kubectl get namespaces`), pods (`kubectl get pods -n webserver`) and claim (`kubectl get pvc -n webserver`) are all gone.  **However** the persistent volume *logs_pv* (`kubectl get pv`) still show a claim by *webserver/webserver-html-pvc* which means it can not be used by other claims.  To make this volume available for future use I will delete it via `kubectl delete pv logs-pv` and then re-attach using the same command above.