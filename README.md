# MongooseIM in Kubernetes

This repository contains a [StatefulSet][sts] definition describing a simple yet
scalable and fault-tolerant MongooseIM cluster.

[sts]: https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/

MongooseIM pods use the already existent MongooseIM container available
from DockerHub: https://hub.docker.com/r/mongooseim/mongooseim/
or GitHub: https://github.com/esl/MongooseIM.


## Getting started - initialize gcloud and kubectl

The following steps apply directly to [Google Kubernetes Engine][gke] (aka GKE),
but most Kubernetes environments should behave in a similar fashion.
If you're completely new to GKE you might be interested
in [GKE Quickstart](https://cloud.google.com/kubernetes-engine/docs/quickstart).

[gke]: https://cloud.google.com/kubernetes-engine/

Below is a recap of steps needed to setup our local machine to control
a Kubernetes cluster hosted on Google Cloud:

```sh
brew install google-cloud-sdk
gcloud init
```

Now let's create a cluster in Google Cloud web dashboard:
GCP -> Kubernetes Engine -> Create cluster. Then:

```sh
CLUSTER_NAME=standard-cluster-1  ## our actual cluster name
gcloud container clusters get-credentials $CLUSTER_NAME
```

Our `kubectl` is now configured to operate on the configured cluster.
Let's check that:

```sh
# Short info on what cluster we're operating on
kubectl config current-context
# Full info
kubectl config view
```

We're all set if the current context is something like:

```sh
$ kubectl config current-context
gke_praxis-magnet-229515_europe-west4-b_standard-cluster-1
```

Let's get the ball rolling!


## Deploy MongooseIM

When working with a Kubernetes cluster it's convenient to see the results
of taken actions. In order to do that, let's start a terminal window and run:

```sh
watch kubectl get node,pod,pv,pvc,sts,svc
```

This is going to be our monitoring window. We'll input commands in another terminal.

If you're reading this in your browser, it's time to clone the repo:

```sh
git clone https://github.com/erszcz/mongooseim-kubernetes
cd mongooseim-kubernetes
```

First, we have to store the configuration for MongooseIM pods into Kubernetes' etcd:

```sh
kubectl apply -f mongoose-cm.yaml
```

The config map will not be visible in the monitoring window,
since we're not monitoring this kind of resource.

MongooseIM pods will automatically initiate communication between one
another and form a cluster, but they require a DNS service for that.
We'll use a [Kubernetes headless service](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services)
and [StatefulSet stable network IDs](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#stable-network-id)
to provide DNS for our cluster members:

```sh
kubectl apply -f mongoose-svc.yaml
```

Having enabled the service,
we should be able to see it appear in the monitoring window.

Let's deploy the cluster:

```sh
kubectl apply -f mongoose-sts.yaml
```

First, a `mongoose` StatefulSet should appear in the monitoring window,
followed by its pods.

When a pod is stuck in `ContainerCreating` state, the first step should be:

```
kubectl describe pods
```

One reason for that might be forgetting to store the `mongoose-cm.yml`
config map prior to starting the StatefulSet.


### Important!

If you're just trying things out
**remember to delete your cluster to avoid unnecessary costs**
after finishing the experiments.
