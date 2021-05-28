# Deploy Mayastor

## Overview

In this Quickstart guide we demonstrate deploying Mayastor by using the Kubernetes manifest files provided within the `deploy`folder of the [Mayastor project's GitHub repository](https://github.com/openebs/Mayastor).  The repository is configured for the GitFlow release pattern, wherein the master branch contains official releases.  By extension, the head of the master branch represents the latest official release.

The steps and commands which follow are intended only for use with, and tested against, the latest release.  Earlier releases or development versions may require a modified or different installation process.

## Create Mayastor Application Resources

### Namespace

{% tabs %}
{% tab title="Command \(GitHub Latest\)" %}
```
kubectl create namespace mayastor
```
{% endtab %}
{% endtabs %}

### RBAC Resources

{% tabs %}
{% tab title="Command \(GitHub Latest\)" %}
```
kubectl create -f https://raw.githubusercontent.com/openebs/Mayastor/master/deploy/moac-rbac.yaml
```
{% endtab %}

{% tab title="Expected Output" %}
```
serviceaccount/moac created
clusterrole.rbac.authorization.k8s.io/moac created
clusterrolebinding.rbac.authorization.k8s.io/moac created
```
{% endtab %}
{% endtabs %}

### Custom Resource Definitions

{% tabs %}
{% tab title="Command \(GitHub Latest\)" %}
```
kubectl apply -f https://raw.githubusercontent.com/openebs/Mayastor/master/csi/moac/crds/mayastorpool.yaml
```
{% endtab %}
{% endtabs %}

## Deploy Mayastor Dependencies

### NATS

Mayastor uses [NATS](https://nats.io/), an Open Source messaging system, as an event bus for some aspects of control plane operations, such as registering Mayastor nodes with MOAC \(Mayastor's primary control plane component\).

{% tabs %}
{% tab title="Command \(GitHub Latest\)" %}
```
kubectl apply -f https://raw.githubusercontent.com/openebs/Mayastor/master/deploy/nats-deployment.yaml
```
{% endtab %}
{% endtabs %}

Verify that the deployment of the NATS application to the cluster was successful. Within the mayastor namespace there should be a single pod having a name starting with "nats-", and with a reported status of Running.

{% tabs %}
{% tab title="Command" %}
```text
kubectl -n mayastor get pods --selector=app=nats
```
{% endtab %}

{% tab title="Example Output" %}
```
NAME                   READY   STATUS    RESTARTS   AGE
nats-b4cbb6c96-nbp75   1/1     Running   0          28s
```
{% endtab %}
{% endtabs %}

### etcd 

**Note: Execute this step ONLY if deploying a version of Mayastor after v0.8.0**
If you have used the definition files from the links given in the previous sections, this step should be omitted.

Mayastor uses [etcd](https://etcd.io/), a distributed, reliable key-value store, to persist runtime configuration.

The steps described below provide an example of how etcd cluster can be deployed for use by Mayastor. 

**&#9888; Warning: This etcd cluster should not be used in production. The user is requested to deploy their own etcd cluster for production use.** 

{% tabs %}
{% tab title="Command \(GitHub Latest\)" %}
```
kubectl apply -f https://raw.githubusercontent.com/openebs/Mayastor/develop/deploy/etcd/secrets.yaml 
kubectl apply -f https://raw.githubusercontent.com/openebs/Mayastor/develop/deploy/etcd/statefulset.yaml 
kubectl apply -f https://raw.githubusercontent.com/openebs/Mayastor/develop/deploy/etcd/svc.yaml
kubectl apply -f https://raw.githubusercontent.com/openebs/Mayastor/develop/deploy/etcd/svc-headless.yaml
```
{% endtab %}
{% endtabs %}

Verify that the deployment of etcd to the cluster was successful. Within the mayastor namespace there should be 3 pods having a name starting with "mayastor-etcd-", and with a reported status of "Running".

{% tabs %}
{% tab title="Command" %}
```text
kubectl -n mayastor get pods --selector=app.kubernetes.io/name=etcd
```
{% endtab %}

{% tab title="Example Output" %}
```
NAME             READY  STATUS   RESTARTS  AGE
mayastor-etcd-0  1/1    Running  0         24m
mayastor-etcd-1  1/1    Running  0         24m
mayastor-etcd-2  1/1    Running  0         24m
```
{% endtab %}
{% endtabs %}

## Deploy Mayastor Components

### CSI Node Plugin

{% tabs %}
{% tab title="Command \(GitHub Latest\)" %}
```
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
```
NAME           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR              AGE
mayastor-csi   3         3         3       3            3           kubernetes.io/arch=amd64   26m
```
{% endtab %}
{% endtabs %}

### Control Plane

{% tabs %}
{% tab title="Command \(GitHub Latest\)" %}
```
kubectl apply -f https://raw.githubusercontent.com/openebs/Mayastor/master/deploy/moac-deployment.yaml
```
{% endtab %}
{% endtabs %}

Verify that the MOAC control plane pod is running.

{% tabs %}
{% tab title="Command" %}
```text
kubectl get pods -n mayastor --selector=app=moac
```
{% endtab %}

{% tab title="Example Output" %}
```
NAME                    READY   STATUS    RESTARTS   AGE
moac-7d487fd5b5-9hj62   3/3     Running   0          8m4s
```
{% endtab %}
{% endtabs %}

### Data Plane 

{% tabs %}
{% tab title="Command \(GitHub Latest\)" %}
```
kubectl apply -f https://raw.githubusercontent.com/openebs/Mayastor/master/deploy/mayastor-daemonset.yaml
```
{% endtab %}
{% endtabs %}

Verify that the Mayastor DaemonSet has been correctly deployed.  The reported Desired, Ready and Available instance counts should be equal, and should match the count of worker nodes which carry the label `openebs.io/engine=mayastor` \(as performed earlier in the "[Preparing the Cluster](preparing-the-cluster.md#label-the-storage-nodes)" stage\).

{% tabs %}
{% tab title="Command" %}
```text
kubectl -n mayastor get daemonset mayastor
```
{% endtab %}

{% tab title="Example Output" %}
```
NAME       DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR                                         AGE
mayastor   3         3         3       3            3           kubernetes.io/arch=amd64,openebs.io/engine=mayastor   108s
```
{% endtab %}
{% endtabs %}

For each resulting Mayastor pod instance, a Mayastor Node \(MSN\) custom resource definition should be created.  List these definitions and verify that the count meets the expected number and that all nodes are reporting their State as `online`

{% tabs %}
{% tab title="Command" %}
```text
kubectl -n mayastor get msn
```
{% endtab %}

{% tab title="Example Output" %}
```
NAME                       STATE    AGE
aks-agentpool-12194210-0   online   8m18s
aks-agentpool-12194210-1   online   8m19s
aks-agentpool-12194210-2   online   8m15s
```
{% endtab %}
{% endtabs %}

