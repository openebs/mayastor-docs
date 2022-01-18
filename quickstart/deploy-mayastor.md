# Deploy Mayastor

## Overview

In this Quickstart guide we demonstrate deploying Mayastor by using the Kubernetes manifest files provided under multiple repositories(for [Control Plane](https://github.com/openebs/mayastor-control-plane), [Data Plane](https://github.com/openebs/mayastor) and [API](https://github.com/openebs/mayastor-api) components) under the OpenEBS project. The repository is configured for the GitFlow release pattern, wherein the master branch contains official releases. By extension, the head of the master branch represents the latest official release.

The steps and commands which follow are intended only for use with, and tested against, the latest release. Earlier releases or development versions may require a modified or different installation process.

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
kubectl apply -f https://raw.githubusercontent.com/openebs/mayastor-control-plane/master/deploy/operator-rbac.yaml
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
kubectl apply -f https://raw.githubusercontent.com/openebs/mayastor-control-plane/master/deploy/mayastorpoolcrd.yaml
```
{% endtab %}
{% endtabs %}

## Deploy Mayastor Dependencies

### NATS

Mayastor uses [NATS](https://nats.io/), an Open Source messaging system, as an event bus for some aspects of control plane operations.

{% tabs %}
{% tab title="Command \(GitHub Latest\)" %}
```text
kubectl apply -f https://raw.githubusercontent.com/openebs/mayastor/master/deploy/nats-deployment.yaml
```
{% endtab %}
{% endtabs %}

Verify that the deployment of the NATS application to the cluster was successful. Within the mayastor namespace ensure that there are 3 replicas with the name "nats-x", and with a reported status of Running.

{% tabs %}
{% tab title="Command" %}
```text
kubectl -n mayastor get pods --selector=app=nats
```
{% endtab %}

{% tab title="Example Output" %}
```text
NAME          READY   STATUS    RESTARTS   AGE
nats-0        3/3     Running   0          61s
```
{% endtab %}
{% endtabs %}

### etcd

Mayastor uses [etcd](https://etcd.io/), a distributed, reliable key-value store, to persist runtime configuration.  The steps described below deploy a limited etcd cluster instance which can be used in conjunction with Mayastor for the purposes of this quickstart guide's testing scenarios.  

{% hint style="warning" %}
The etcd cluster deployed here is intended for demonstration purposes only. For production use, the user is responsible for the deployment of a suitable etcd instance which satisfies their use case and follows the recommended best practices for etcd.
{% endhint %}

{% tabs %}
{% tab title="Command \(GitHub Latest\)" %}
```text
kubectl apply -f https://raw.githubusercontent.com/openebs/Mayastor/master/deploy/etcd/statefulset.yaml 
kubectl apply -f https://raw.githubusercontent.com/openebs/Mayastor/master/deploy/etcd/svc.yaml
kubectl apply -f https://raw.githubusercontent.com/openebs/Mayastor/master/deploy/etcd/svc-headless.yaml
```
{% endtab %}
{% endtabs %}

Verify that the deployment of etcd to the cluster was successful. Within the mayastor namespace there should be 3 replicas with the name "mayastor-etcd-", and with a reported status of "Running".

{% tabs %}
{% tab title="Command" %}
```text
kubectl -n mayastor get pods --selector=app.kubernetes.io/name=etcd
```
{% endtab %}

{% tab title="Example Output" %}
```text
NAME              READY   STATUS    RESTARTS   AGE
mayastor-etcd-0   3/3     Running   0          4m32s
```
{% endtab %}
{% endtabs %}

## Deploy Mayastor Components

### CSI Node Plugin

{% tabs %}
{% tab title="Command \(GitHub Latest\)" %}
```text
kubectl apply -f https://raw.githubusercontent.com/openebs/Mayastor/master/deploy/csi-daemonset.yaml
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

### Control Plane

### Core Agents

{% tabs %}
{% tab title="Command \(GitHub Latest\)" %}
```text
kubectl apply -f https://raw.githubusercontent.com/openebs/mayastor-control-plane/master/deploy/core-agents-deployment.yaml
```
{% endtab %}
{% endtabs %}

Verify that the Core agent pod is running.

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


### CSI Controller


{% tabs %}
{% tab title="Command \(GitHub Latest\)" %}
```text
kubectl apply -f https://raw.githubusercontent.com/openebs/mayastor-control-plane/master/deploy/csi-deployment.yaml
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

### MSP Operators
 
{% tabs %}
{% tab title="Command \(GitHub Latest\)" %}
```text
kubectl apply -f https://raw.githubusercontent.com/openebs/mayastor-control-plane/master/deploy/msp-deployment.yaml
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

### REST

{% tabs %}
{% tab title="Command \(GitHub Latest\)" %}
```text
kubectl apply -f https://raw.githubusercontent.com/openebs/mayastor-control-plane/master/deploy/rest-deployment.yaml
kubectl apply -f https://raw.githubusercontent.com/openbes/mayastor-control-plane/master/deploy/rest-service.yaml
```
{% endtab %}

{% tab title="Example Output" %}
```text
deployment.apps/rest created
service/rest created
```
{% endtab %}
{% endtabs %}

### Data Plane

{% tabs %}
{% tab title="Command \(GitHub Latest\)" %}
```text
kubectl apply -f https://raw.githubusercontent.com/openebs/mayastor/master/deploy/mayastor-daemonset.yaml
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

For each resulting Mayastor pod instance, a Mayastor Node \(MSN\) custom resource definition should be created. List these definitions and verify that the count meets the expected number and that all nodes are reporting their State as `online`

{% tabs %}
{% tab title="Command" %}
```text
kubectl mayastor get nodes -n mayastor 
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

