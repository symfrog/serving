# Resources

This document provides a high-level description of the resources deployed to a Kubernetes cluster in order to run Elafros. The exact list of resources is going to change frequently during the current phase of active development. In order to keep this document from becoming out-of-date frequently it doesn't describe the exact individual resources but instead the higher level objects which they form.

## Dependencies

Elafros depends on two other projects in order to function: [Istio][istio] and the [Build CRD][build-crd]. Istio is responsible for setting up the network routing both inside the cluster and ingress into the cluster. The Build CRD provides a custom resource for Kubernetes which provides an extensible primitive for creating container images from various sources, for example a Git repository.

You can find out more about both from their respective websites.

[istio]: https://istio.io/
[build-crd]: https://github.com/google/build-crd

## Components

There are two primary components to the Elafros system. The first is a controller which is responsible for updating the state of the cluster based on user input. The second is the webhook component which handles validation of the objects and actions performed.

The controller processes a series of state changes in order to move the system from its current, actual state to the state desired by the user.

All of the Elafros components are deployed into the `ela-system` namespace. You can see the various objects in this namespace by running `kubectl -n ela-system get all` ([minus some admin-level resources like service accounts](https://github.com/kubernetes/kubectl/issues/151)). To see only objects of a specific type, for example to see the webhook and controller deployments inside Elafros, you can run `kubectl -n ela-system get deployments`. The Elafros controller creates other child objects inside Kubernetes. You can specify the namespace for your child objects by specifying your desired namespace when deploying your Elafros configuration.

## Precise Object Listing

You can get a single file which contains a list of the Kubernetes resources that are installed with Elafros by using the following Bazel commands:

```
# All resources including dependencies
bazel run //:everything > everything.yaml

# Just resources from Istio
bazel run //:istio > istio.yaml

# Just resources from Build CRD
bazel run @buildcrd//:everything > buildcrd.yaml

# Just resources from Elafros itself
bazel run //:elafros > elafros.yaml
```

## Viewing resources after deploying Elafros

### Custom Resource Definitions

To view all of the custom resource definitions created, run `kubectl get customresourcedefinitions`. These resources are named according to their group, i.e. custom resources required by Elafros end with `elafros.dev`.

### Deployments

View the Elafros specific deployments by running `kubectl -n ela-system get deployments`. These deployments will ensure that the correct number of pods are running for that specific deployment.

For example, given:

```
$ kubectl -n ela-system get deployments
NAME             DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
ela-controller   1         1         1            1           6m
ela-webhook      1         1         1            1           6m
```

Based on the desired state shown above, we expect there to be a single pod running for each of the deployments shown above. We can verify this by running and seeing similar output as shown below:

```
$ kubectl -n ela-system get pods
NAME                              READY     STATUS    RESTARTS   AGE
ela-controller-5bfb798f96-2zjnf   1/1       Running   0          9m
ela-webhook-64c459569b-v5npx      1/1       Running   0          8m
```

Similarly, you can run the same commands in the build-crd (`build-system`) and istio (`istio-system`) namespaces to view the running deployments. To view all namespaces, run `kubectl get namespaces`.

### Service Accounts and RBAC policies

To view the service accounts configured for Elafros, run `kubectl -n ela-system get serviceaccounts`.

To view all cluster role bindings, run `kubectl get clusterrolebindings`. Unfortunately there is currently no mechanism to fetch the cluster role bindings that are tied to a service account.