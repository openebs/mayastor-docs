# Deploy Mayastor

{% hint style="danger" %}
This website/page will be End-of-life (EOL) after 31 August 2024. We recommend you to visit [OpenEBS Documentation](https://openebs.io/docs/user-guides/replicated-storage-user-guide/replicated-pv-mayastor/rs-installation) for the latest Mayastor documentation (v2.6 and above).
 
Mayastor is now also referred to as OpenEBS Replicated PV Mayastor.
{% endhint %}

## Overview

{% hint style="note" %}
Before deploying and using Mayastor please consult the [Known Issues](https://mayastor.gitbook.io/introduction/quickstart/known-issues) section of this guide.
{% endhint %}

In this Quickstart guide we demonstrate deploying Mayastor by using the Kubernetes manifest files provided within the project's repositories ([control plane components](https://github.com/openebs/mayastor-control-plane), [data plane components](https://github.com/openebs/mayastor)). These repositories are configured for the GitFlow release model, wherein the master branch contains official releases. By extension, the head of the master branch represents the latest official release.  Previous releases are identifiable by their annotated git tags.

The steps and commands which follow are intended only for use with, and have only be tested against, the _latest release for version 1.x_. Earlier releases or development versions may require a modified or different installation process and _will_ require that the manifest files appropriate to that specific release are used.  These are available in the repositories named above - use the files from the tagged commit appropriate to the release you wish to deploy.

## Create Mayastor Application Resources

### Namespace

{% tabs %}
{% tab title="Command \(GitHub Latest\)" %}
```text
kubectl create namespace mayastor
```
{% endtab %}
{% endtabs %}

### RBAC Resources

{% tabs %}
{% tab title="Command \(GitHub Latest\)" %}
```text
kubectl apply -f https://raw.githubusercontent.com/openebs/mayastor-control-plane/v1.0.5/deploy/operator-rbac.yaml
```
{% endtab %}

{% tab title="Expected Output" %}
```text
serviceaccount/mayastor-service-account created
clusterrole.rbac.authorization.k8s.io/mayastor-cluster-role created
clusterrolebinding.rbac.authorization.k8s.io/mayastor-cluster-role-binding created
```
{% endtab %}
{% endtabs %}

### Custom Resource Definitions

{% tabs %}
{% tab title="Command \(GitHub Latest\)" %}
```text
kubectl apply -f https://raw.githubusercontent.com/openebs/mayastor-control-plane/v1.0.5/deploy/mayastorpoolcrd.yaml
```
{% endtab %}
{% endtabs %}

## Deploy Mayastor Dependencies

### NATS

Mayastor uses [NATS](https://nats.io/), an Open Source messaging system, as an event bus for some aspects of control plane operations.

{% tabs %}
{% tab title="Command \(GitHub Latest\)" %}
```text
kubectl apply -f https://raw.githubusercontent.com/openebs/mayastor/v1.0.5/deploy/nats-deployment.yaml
```
{% endtab %}
{% endtabs %}

Verify that the deployment of the NATS application to the cluster was successful. Within the mayastor namespace ensure that there are 3 replicas with the name "nats-x", and with a reported status of "Running".

{% tabs %}
{% tab title="Command" %}
```text
kubectl -n mayastor get pods --selector=app=nats
```
{% endtab %}

{% tab title="Example Output" %}
```text
NAME     READY   STATUS    RESTARTS   AGE
nats-0   2/2     Running   0          50s
nats-1   2/2     Running   0          30s
nats-2   2/2     Running   0          10s
```
{% endtab %}
{% endtabs %}

### etcd

Mayastor uses [etcd](https://etcd.io/), a distributed, reliable key-value store, to persist configuration.  The steps described below deploy a dedicated, clustered etcd  instance for Mayastor's own use. This is the only configuration supported by this release.  

To deploy the PersistentVolumes that will be used by the etcd in the next step, execute:
{% tabs %}
{% tab title="Command \(GitHub Latest\)" %}
```text
kubectl apply -f https://raw.githubusercontent.com/openebs/mayastor/v1.0.5/deploy/etcd/storage/localpv.yaml
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Command \(GitHub Latest\)" %}
```text
kubectl apply -f https://raw.githubusercontent.com/openebs/mayastor/v1.0.5/deploy/etcd/statefulset.yaml 
kubectl apply -f https://raw.githubusercontent.com/openebs/mayastor/v1.0.5/deploy/etcd/svc.yaml
kubectl apply -f https://raw.githubusercontent.com/openebs/mayastor/v1.0.5/deploy/etcd/svc-headless.yaml
```
{% endtab %}
{% endtabs %}

Verify that the deployment of etcd to the cluster was successful. Within the mayastor namespace there should be 3 replicas with a name of the form "mayastor-etcd-" and with a reported status of "Running".

{% tabs %}
{% tab title="Command" %}
```text
kubectl -n mayastor get pods --selector=app.kubernetes.io/name=etcd
```
{% endtab %}

{% tab title="Example Output" %}
```text
NAME              READY   STATUS    RESTARTS   AGE
mayastor-etcd-0   1/1     Running   0          70m
mayastor-etcd-1   1/1     Running   0          70m
mayastor-etcd-2   1/1     Running   0          70m
```
{% endtab %}
{% endtabs %}

## Deploy Mayastor Components

### CSI Node Plugin

{% tabs %}
{% tab title="Command \(GitHub Latest\)" %}
```text
kubectl apply -f https://raw.githubusercontent.com/openebs/mayastor/v1.0.5/deploy/csi-daemonset.yaml
```
{% endtab %}
{% endtabs %}

Verify that the CSI Node Plugin DaemonSet has been correctly deployed to all worker nodes in the cluster.

{% tabs %}
{% tab title="Command" %}
```text
kubectl -n mayastor get daemonset mayastor-csi
```
{% endtab %}

{% tab title="Example Output" %}
```text
NAME           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR              AGE
mayastor-csi   3         3         3       3            3           kubernetes.io/arch=amd64   26m
```
{% endtab %}
{% endtabs %}

## Control Plane

### Core Agents

{% tabs %}
{% tab title="Command \(GitHub Latest\)" %}
```text
kubectl apply -f https://raw.githubusercontent.com/openebs/mayastor-control-plane/v1.0.5/deploy/core-agents-deployment.yaml
```
{% endtab %}
{% endtabs %}

Verify that the core agent pod is running.

{% tabs %}
{% tab title="Command" %}
```text
kubectl get pods -n mayastor --selector=app=core-agents
```
{% endtab %}

{% tab title="Example Output" %}
```text
NAME                           READY   STATUS    RESTARTS   AGE
core-agents-5f4d9f786b-6vvxc   1/1     Running   0          117s
```
{% endtab %}
{% endtabs %}

### REST

{% tabs %}
{% tab title="Command \(GitHub Latest\)" %}
```text
kubectl apply -f https://raw.githubusercontent.com/openebs/mayastor-control-plane/v1.0.5/deploy/rest-deployment.yaml
kubectl apply -f https://raw.githubusercontent.com/openebs/mayastor-control-plane/v1.0.5/deploy/rest-service.yaml
```
{% endtab %}

{% tab title="Example Output" %}
```text
deployment.apps/rest created
service/rest created
```
{% endtab %}
{% endtabs %}

Verify that the REST pod is running.

{% tabs %}
{% tab title="Command" %}
```text
kubectl get pods -n mayastor --selector=app=rest
```
{% endtab %}

{% tab title="Example Output" %}
```text
NAME                    READY   STATUS    RESTARTS   AGE
rest-5cd9665499-cdgmm   1/1     Running   0          2m35s
```
{% endtab %}
{% endtabs %}

### CSI Controller


{% tabs %}
{% tab title="Command \(GitHub Latest\)" %}
```text
kubectl apply -f https://raw.githubusercontent.com/openebs/mayastor-control-plane/v1.0.5/deploy/csi-deployment.yaml
```
{% endtab %}
{% endtabs %}

Verify that the CSI-controller pod is running.

{% tabs %}
{% tab title="Command" %}
```text
kubectl get pods -n mayastor --selector=app=csi-controller
```
{% endtab %}

{% tab title="Example Output" %}
```text
NAME                             READY   STATUS    RESTARTS   AGE
csi-controller-579f77f64-7dq7g   3/3     Running   0          39m
```
{% endtab %}
{% endtabs %}

### MSP Operator
 
{% tabs %}
{% tab title="Command \(GitHub Latest\)" %}
```text
kubectl apply -f https://raw.githubusercontent.com/openebs/mayastor-control-plane/v1.0.5/deploy/msp-deployment.yaml
```
{% endtab %}
{% endtabs %}

Verify that the pool operator pod is running.

{% tabs %}
{% tab title="Command" %}
```text
kubectl get pods -n mayastor --selector=app=msp-operator
```
{% endtab %}

{% tab title="Example Output" %}
```text
NAME                            READY   STATUS    RESTARTS   AGE
msp-operator-7849d59fcd-mcw5b   1/1     Running   0          21s
```
{% endtab %}
{% endtabs %}


## Data Plane

{% tabs %}
{% tab title="Command \(GitHub Latest\)" %}
```text
kubectl apply -f https://raw.githubusercontent.com/openebs/mayastor/v1.0.5/deploy/mayastor-daemonset.yaml
```
{% endtab %}
{% endtabs %}

Verify that the Mayastor DaemonSet has been correctly deployed. The reported Desired, Ready and Available instance counts should be equal, and should match the count of worker nodes which carry the label `openebs.io/engine=mayastor` \(as performed earlier in the "[Preparing the Cluster](preparing-the-cluster.md#label-the-storage-nodes)" stage\).

{% tabs %}
{% tab title="Command" %}
```text
kubectl -n mayastor get daemonset mayastor
```
{% endtab %}

{% tab title="Example Output" %}
```text
NAME       DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR                                         AGE
mayastor   3         3         3       3            3           kubernetes.io/arch=amd64,openebs.io/engine=mayastor   108s
```
{% endtab %}
{% endtabs %}

The number and status of mayastor pods can be observed by using the [Mayastor kubectl plugin](https://mayastor.gitbook.io/introduction/reference/kubectl-plugin). Check that the expected number of nodes are reporting their State as `online`

{% tabs %}
{% tab title="Command" %}
```text
kubectl mayastor get nodes
```
{% endtab %}

{% tab title="Example Output" %}
```text
NAME                       STATE    AGE
aks-agentpool-12194210-0   online   8m18s
aks-agentpool-12194210-1   online   8m19s
aks-agentpool-12194210-2   online   8m15s
```
{% endtab %}
{% endtabs %}

