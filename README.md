# Overview

The leading open source automation server, Jenkins provides hundreds of plugins to support building, deploying and automating any project.

Please, visit [Jenkins website](https://jenkins.io/) to know more about it.

## About Google Click to Deploy K8s Solutions

Popular open source software stacks on Kubernetes packaged by Google and made available in Google Cloud Marketplace.

# Installation

## Quick install with Google Cloud Marketplace

Install this Jenkins app to a Google Kubernetes Engine cluster using Google Cloud Marketplace. Follow the
[on-screen instructions](https://console.cloud.google.com/launcher/details/google/jenkins2).

## Command line instructions

### Prerequisites

#### Set up command-line tools

You'll need the following tools in your environment:
- [gcloud](https://cloud.google.com/sdk/gcloud/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

If you don't have gcloud command authenticated (or installed), please, follow this procedure to configure it for the first time.
- [Download and authenticate gcloud](https://cloud.google.com/sdk/#Quick_Start)

#### Configuration referenced in this readme

```shell
### a bit of configuration
### feel free to put it in a file for future use
export CLUSTER=a-cluster
export ZONE=us-west1-a
export namespace=jenkins
```

#### Create a Google Kubernetes Engine cluster

This step is optional if you have a cluster running and don't need a new one.

```shell
gcloud container clusters create "$CLUSTER" --zone "$ZONE"
```

#### Configure kubectl to use specific cluster

```shell
gcloud container clusters get-credentials "$CLUSTER" --zone "$ZONE"
```

#### Install Jenkins application

```shell
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /tmp/jenkins.key -out /tmp/jenkins.crt \
  -subj "/CN=jenkins/O=jenkins"

kubectl create namespace $namespace
kubectl create secret generic tls --namespace $namespace \
  --from-file=/tmp/jenkins.crt --from-file=/tmp/jenkins.key
kubectl create clusterrolebinding cluster-admin-binding \
  --clusterrole=cluster-admin --user=$(gcloud config get-value account)

kubectl apply -f ./jenkins-deployment.yaml
```

#### Take a five minutes break...

or you can check installation status if you like (it's boring, believe me)

```shell
kubectl -n $namespace describe ingress
```

Check a line looking like

```
ingress.kubernetes.io/backends:      {"k8s-be-30242--088f66391640a749":"Unknown"}
```

When "Unknown" changes to "HEALTHY" your installation is ready to use.

#### Login into your brand new Jenkins instance

Get the Jenkins HTTP/HTTPS address

```shell
echo https://$(kubectl -n$namespace get ingress \
  -ojsonpath="{.items[0].status.loadBalancer.ingress[0].ip}")/
### it's just a funcy way to do this
kubectl -n $namespace get ingress
### and copy ip address into browser
```

For HTTPS you have to accept a certificate (we created a temporary one). Now you probably need a password

```shell
kubectl -n$namespace exec $(kubectl -n$namespace get pod -oname|sed s.pod/..) \
  cat /var/jenkins_home/secrets/initialAdminPassword
```

#### Follow on screen instructions

- install plugins
- create first admin user
- set jenkins URL (default is ok and you can change it later)
- start using your fresh Jenkins installation

# Scaling

This installation is single master. If you need more power, just configure additional jenkins workers (slaves).

# Backup and Restore

Copy content of jenkins persistent volume or install Jenkins backup plugin, configure it to use .tar.gz format and create backup from Jenkins UI.

Copy backup into jenkins persistent volume or use Jenkins backup plugin to restore configuration.

# Update and Upgrade

Just kill your Jenkins pod and let Kubernetes install new version (please, consider creating backup before).

```shell
### did I mention backup?
kubectl -n$namespace delete $(kubectl -n$namespace get pod -oname)
```

# Deletion

Warning! Nothing will left, persistent volume will be deleted as well and there is no "are you sure?" question. Have you thought about backup?

```shell
kubectl delete namespace $namespace
```

# Logging and Monitoring

This Jenkins installation logs to [Stackdriver](https://cloud.google.com/monitoring/)

