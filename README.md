# PCF vs Docker and Kubernetes the Structured vs Unstructured Platform Comparison

##Objective: 
This is a demo for Pivotal Field Engineers to show the value Pivotal Cloud Foundry offers to customers as an opinionated, structured platform designed for continuous delivery of apps compared to a Do-It-Yourself platform built by customers (or possibly other vendors) using Docker and Kubernetes as a platform to build and orchestrate containers. 

The following demo will walk you through a basic real-life scenarios such as deploying an application, accessing application logs, and discovery and connecting to an existing set of services, such as a messaging service (Redis) or a MySQL database. Additional scenarios are also provided for comparison on additional capabilities for more in-depth demonstrations.

##Summary:
The biggest technical value proposition that I have come to realize is Pivotal Cloud Foundry, though opinionated, is still very open platform with many forms of integration offered to enterprise customers as a turnkey solution. Whether it be giving customers the choice to run Pivotal Cloud Foundry on any cloud of their choice, such as AWS, VMware, vCloudAir, Openstack and Azure or providing essential features such as integration with existing enterprise users through an existing Active Directory Service, Pivotal Cloud Foundry provides an easy to use and operationalize platform to continuously deliver apps providing real business or mission benefits to end users that cannot be offered by DIY PaaS solutions such as Docker and Kubernetes. 

##Pre-Reqs: 
This demo assumes that you have a basic knowledge of Cloud Foundry and can deploy your application to Pivotal Web Services, this includes installing and using the CF CLI as well as GIT. It also assumes that you are familiar with the basics of using Docker & core concepts of Kubernetes such as pods, replication controllers, and services. Links are provided in Appendix for those who may wish to explore these technologies prior to running through the demo. 


##Deploying to Cloud Foundry

Clone the repository to your local machine
```bash
$ git clone https://github.com/cloudfoundry-samples/lattice-app
```
Navigate to root of the applications directory that was just cloned
Example
```bash
$ cd /lattice-app
```
Push to cloud foundry
```bash
$ cf push [APP-NAME]
```
> Substitute [APP-NAME] with a unique name you would like to give the application

View just the most recent logs for all instances of your application
```bash
$ cf logs --recent [APP-NAME]
```
View all new logs real time by starting tailing mode to print out logs for all application instances
```bash
$ cf logs [APP-NAME]
```

##Deploying App to Kubernetes 
Sign up for a  [Google Account](https://accounts.google.com/SignUp)
 
Enable [billing](https://console.developers.google.com/billing) in the Developers Console in order to use Google Compute Engine resources. 

Create a new [project](https://console.developers.google.com/project/_/kubernetes/list) and enable the Container Engine API

We will be using Google Cloud Shell, so downloading Google Cloud CLI (gcloud) and Kubernetes CLI (kubectl) to your local machine is not necessary.

Log-In to the [Developers Console](https://console.developers.google.com/)

Launch [Google Cloud Shell](https://cloud.google.com/cloud-shell/docs/) [Docs](https://cloud.google.com/cloud-shell/docs/quickstart)

![http://imgur.com/9GUK1av.png](http://imgur.com/9GUK1av.png)

#####Setup Default Zone, Project Configuration, and Launch Container Cluster

Set Project ID
```bash
$ gcloud config set project <ENTER_PROJECT_ID>
```
To find the value of Project ID, check [here](https://console.developers.google.com/project) and refer to the Project ID column. 

Set Compute Engine zone: 
```bash
$ gcloud config set compute/zone us-east1-d
```
Select an alternative [availability region & zone](https://cloud.google.com/compute/docs/zones#available) for Google Compute Engine.

Create a Container Engine cluster with 5 Virtual Machines running Open Source Kubernetes in Google Compute Engine. 
```bash
$ gcloud beta container clusters create my-kube-cluster --num-nodes=5
```

The command above will take a few minutes to run. See the newly created compute instances by visiting the Container Enginer Dashboard link in Google Developer Console. 

![http://imgur.com/PRb1McB.png](http://imgur.com/PRb1McB.png)

Set newly created container cluster as the default cluster
```bash
$ gcloud config set container/cluster my-kube-cluster
```

Fetch container cluster credentials for the Kubernetes (kubectl) command line tool:
```bash
$ gcloud container clusters get-credentials my-kube-cluster
```

Verify Configuration Settings for the Project
```bash
$ gcloud config list
```


#####Running the Application on Kubernetes

Pull App’s Docker Image from Docker Hub Public Registry. 
```bash
$ docker pull cloudfoundry/lattice-app
```
* Docker is already installed in Google Shell, so no manual installation of Docker client is required. 

Run App on Kubernetes
```bash
$ kubectl run demo-app --image=cloudfoundry/lattice-app
```
* Kubernetes should display a message confirming that it is running a single container inside a pod within the container cluster. 

![http://imgur.com/07CCWGk.png](http://imgur.com/07CCWGk.png)
Add an external-facing load balancer to the deployed application. 
```bash
$ kubectl expose rc demo-app --create-external-load-balancer=true --port=80 --target-port=8080
```
This creates a load-balancer as a  “service” inside Kubernetes, which serves on port 80 and connects to the container, where application is running on port 8080.

View the Application running inside Kubernetes
```bash
$ kubectl get services demo-app
```
* This provides an public IP address for the newly created load-balancer which directs external traffic to our application. Type the external IP to view the app in a browser. 
In this example, 104.196.29.247 is the external IP address provided to view the application. 

![http://imgur.com/1Kix2K6.png](http://imgur.com/1Kix2K6.png)

Print logs for a container in a pod.
```bash
$ kubectl get pods [ * Make note of the name of the pod running your application ]
$ kubctl logs <name-of-the-pod> -c demo-app 
```

Scale the App
```bash
$ kubectl scale rc demo-app --replicas=5
```
* Verify that the app scaled to desired number by get the new number of existing pods. 
```bash
$ kubectl get pods
```
Clean Up
```bash
$ kubectl delete services demo-app
$ kubectl delete rc demo-app
$ gcloud container clusters delete my-kube-cluster 
```
###Appendix:
#####Pivotal Cloud Foundry
*[Developer Guide](http://docs.pivotal.io/pivotalcf/devguide/index.html)  
*[Getting Started](http://docs.pivotal.io/pivotalcf/getstarted/pcf-docs.html)
#####Kubernetes Documentation
*[Kubernetes Home](http://kubernetes.io/)  
*[kubectl](http://kubernetes.io/v1.0/docs/user-guide/kubectl/kubectl.html)
#####Docker
*[Docker Home](https://www.docker.com/)  
*[User Guide](https://docs.docker.com/userguide/)
#####Google Cloud Platform
*[Before you begin](https://cloud.google.com/container-engine/docs/before-you-begin)  
*[Hello Node](https://cloud.google.com/container-engine/docs/tutorials/hello-node)








