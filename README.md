# A Simple City App (Flask, Elasticsearch, Docker, Kubernetes)

A simple container application for CRUDing a city and its population.

## Getting Started

This example has been tested in a fully-isolated linux environment using [Linux KVM](https://www.linux-kvm.org/page/Downloads) and [Kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/). It appears that it could also be applied in other settings.

This solution uses [Python Flask](https://flask.palletsprojects.com/en/2.3.x/installation/) as the main app, and [Elasticsearch](https://www.elastic.co/) as a database.

Run in this directory to build an image:

```shell
cd myapp
docker build -t cityapp:v1.0.0 .
```

It will trigger the build by looking into the Dockerfile. The image will be saved on your local computer / laptop.

Next, we want to save this image to a container registry so that we can use it later. 
I use [Docker Hub](https://hub.docker.com) as a container registry. 

Before I can push the image to the docker hub, I need to login first.

```shell
docker login --username <account_name> --password <account_password>
```

After a successful login, I need to tag the image. The tag has an advantage to mark your image so docker knows where the image will be:

```
 docker tag cityapp:v1.0.0 rchronic/cityapp:v1.0.0
```

This tag has some rules to follow. The first `cityapp:v1.0.0` comes from `docker build` that we run at the first time. The `rchronic` is my docker project on `docker hub`. The last `cityapp` is my repository on `docker hub`. And the last ‘v1.0.0’ is the image version.

Push the tagged image:

```
docker push rchronic/cityapp:v1.0.0
```

This `rchronic/cityapp:v1.0.0` comes from the tag. You need to wait until it finishes its job.


#### . . . Build and Push app done!!!




## Run the app in Kubernetes

It’s time to push our app to the server. Here, I will be using a [Helm Chart](https://helm.sh).

Before we proceed to the `helm chart`, you must have a running kubernetes. Setting the kubectl to appropriate kubernetes API makes the `helm chart` to work.

Run the following command to create the deployments and services:

```
helm install cityapp . -n city-app --create-namespace
```

That’s it!!! Now our app is on kubernetes

[OPTIONAL] To remove the app from the server:

```
helm -n city-app uninstall cityapp && kubectl delete ns city-app
```





## Install the Database Using Elasticsearch

Installing the elasticsearch needs an additional repo from its [Official Elasticsearch Helm Chart](https://artifacthub.io/packages/helm/elastic/elasticsearch):

```
helm repo add elastic https://helm.elastic.co
helm repo update
helm repo list
```

Check the latest version of a specific package that you want to use:

```
helm search repo elasticsearch
```

Run the helm chart like this:

```
helm install es elastic/elasticsearch -f ./values.yaml -n es --create-namespace
```

It will get a package from `helm.elastic.co`, then the values.yaml will override some of the content. Also this will create a new `Kubernetes Mamespace` called `es` if it doesn’t exist.

Because I run this app on top of Kubernetes Vanilla, I need other storage to save the data. So I take a `Persistent Volume` manifest yaml file, and then apply it:

```
kubectl apply -f pv-0.yaml 
```

Wait for a moment until the storage is bind to the app. You can run some various command to check that:

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

If those all run normally, you can try to access some endpoints:

| URL                                           | METHOD      | BODY                                       |
|-----------------------------------------------|-------------|--------------------------------------------|
| `http://<<ip_service>>:5000/health`           | GET         |                                            |
| `http://<<ip_service>>:5000/city`             | POST        | { 'city': city, 'population': population } |
| `http://<<ip_service>>:5000/city/<city_name>` | PUT         | { 'population': population }               |
| `http://<<ip_service>>:5000/city/<city_name>` | DELETE      |                                            |
| `http://<<ip_service>>:5000/city/<city_name>` | GET         |                                            |

