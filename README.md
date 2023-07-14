# A Simple City App (Flask, Elasticsearch, Docker, Kubernetes)

A simple container application for CRUDing a city and its population.

## Getting Started

This example has been tested in a fully-isolated linux environment using [Linux KVM](https://www.linux-kvm.org/page/Downloads) and [Kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/). It appears that it could also be applied in other settings.

This solution uses [Python Flask](https://flask.palletsprojects.com/en/2.3.x/installation/) as the main app, and [Elasticsearch](https://www.elastic.co/) as a database.

## Build and Push Image into Registry
Before we proceed to the build step. You need to login to [Docker Hub](https://hub.docker.com).

```shell
docker login --username <account_name> --password <account_password>
```

Run in this directory to build an image and push the image to the registry:

```shell
cd myapp
./buildpush.sh 1.4.0
```

It will trigger the docker build by looking into the Dockerfile. The image will be saved on your local machine. And then, the image will be tagged so that it can be pushed to the registry. The tag has an advantage to mark your image so docker knows where the image will be stored.

You need to wait until it finishes its job.

#### . . . Build and Push App Done!!!

## Run the app on Kubernetes

It’s time to deploy our app to the server. Here, I will be using a [Helm Chart](https://helm.sh).

Before we proceed to the `helm chart`, you must have a running kubernetes cluster. I configure my pc to have this setting:

| NODES                 | CPU      | MEMORY    |
|-----------------------|----------|-----------|
| `1 MASTER NODE`       | 4core    | 4096MB    |
| `1 WORKER NODE`       | 4core    | 4096MB    |


After that, you need to install a kubectl and then set its configuration to contain appropriate IP of the kubernetes API.

Go back to previous dir:

```
cd ..
```

Run the following command to create the deployments and services:

```
cd chart-myapp
helm install cityapp . -n city-app --create-namespace
```

That’s it!!! Now our app is on kubernetes

#### . . . Deploy App Done!!!

##

[OPTIONAL] To remove the app from the server:

```
helm -n city-app uninstall cityapp && kubectl delete ns city-app
```

## Using Elasticsearch as the database

Installing elasticsearch needs an additional repo from its Official [Elasticsearch Helm Chart](https://artifacthub.io/packages/helm/elastic/elasticsearch):

```
helm repo add elastic https://helm.elastic.co
helm repo update
helm repo list
```

Check the latest version of a specific package that you want to use:

```
helm search repo elasticsearch
```

Go back to previous dir:

```
cd ..
```

To install the elasticsearch, you can run the helm chart like this:

```
cd chart-es
helm install es elastic/elasticsearch -f ./values.yaml -n es --create-namespace
```

It pulls an "elasticsearch" chart from `https://helm.elastic.co`, then the values.yaml will override some of the config fields. Also this created a new `Kubernetes Namespace` called `es` if it doesn’t exist.

Because I run this app on top of Kubernetes Vanilla using Kubeadm, I need other storage to save the data. So I take a `Persistent Volume` manifest yaml file, and then apply it with this:

```
kubectl apply -f pv-0.yaml 
```

Wait for a moment until the storage is bound to the app. You can run some various command to check the status:

Check the status of persistent volume:

```
kubectl get pv
```

Check the status of persistent volume claim in namespace `es`:

```
kubectl -n es get pvc
```

Check the status of pod in namespace `es`:

```
kubectl -n es get pod
```

If those all run normally, you can try to access some endpoints. Here, I use `NodePort Service` so you can access the app by accessing `Node IP`:`Node Port`.

| URL                                | METHOD      | BODY                                       |
|------------------------------------|-------------|--------------------------------------------|
| `http://<<ip_node>>:30050/health`  | GET         |                                            |
| `http://<<ip_node>>:30050/city`    | POST        | { 'city': city, 'population': population } |
| `http://<<ip_node>>:30050/city`    | PUT         | { 'city': city, 'population': population } |
| `http://<<ip_node>>:30050/city`    | DELETE      | { 'city': city }                           |
| `http://<<ip_node>>:30050/city`    | GET         | { 'city': city }                           |

##

[OPTIONAL] You can destroy the database server by running this command:

```
helm -n es uninstall es && kubectl delete ns es && k delete -f pv-0.yaml
```
