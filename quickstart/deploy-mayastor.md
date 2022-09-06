# Deploy Mayastor

## Overview

{% hint style="note" %}
Before deploying and using Mayastor please consult the [Known Issues](https://mayastor.gitbook.io/introduction/quickstart/known-issues) section of this guide.
{% endhint %}

In this Quickstart guide we demonstrate deploying Mayastor by using the Kubernetes manifest files provided within [`mayastor-extensions`](https://github.com/openebs/mayastor-extensions) repository. The repository is configured for the GitFlow release model, wherein the master branch contains official releases. By extension, the head of the master branch represents the latest official release.  Previous releases are identifiable by their annotated git tags.

The steps and commands which follow are intended only for use with, and tested against, the latest release. Earlier releases or development versions may require a modified or different installation process.


## Installation through helm

1. Clone the `mayastor-extensions` repository.

{% tab %}

```text

git clone https://github.com/openebs/mayastor-extensions.git

cd mayastor-extensions/chart

```

{% endtab %}

2. Next, create a new namespace. This namespace will have all the Mayastor components deployed in it. In the sample command given below, the namespace `mayastor` is created.


{% tab %}
```text

kubectl create ns mayastor

```
{% endtab %}


3. Download the dependent helm charts prior to the installation of Mayastor. These charts include:

 * <b>etcd:</b> this is used to maintain Mayastor topology.

 * <b>Jaeger operator(optional):</b> this is used for debugging any issues within the system.

 * <b>Loki stack:</b> this is used for centralized logging.

{% tabs %}
{% tab title="Command" %}
```text
helm dependency update
```
{% endtab %}

{% tab title="Sample Output" %}
```text
Getting updates for unmanaged Helm repositories...
...Successfully got an update from the "https://jaegertracing.github.io/helm-charts" chart repository
...Successfully got an update from the "https://grafana.github.io/helm-charts" chart repository
...Successfully got an update from the "https://raw.githubusercontent.com/bitnami/charts/eb5f9a9513d987b519f0ecd732e7031241c50328/bitnami" chart repository
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "kubera" chart repository
...Successfully got an update from the "prometheus-community" chart repository
Update Complete. ⎈Happy Helming!⎈
Saving 3 charts
Downloading etcd from repo https://raw.githubusercontent.com/bitnami/charts/eb5f9a9513d987b519f0ecd732e7031241c50328/bitnami
Downloading jaeger-operator from repo https://jaegertracing.github.io/helm-charts
Downloading loki-stack from repo https://grafana.github.io/helm-charts
Deleting outdated charts
```
{% endtab %}
{% endtabs %}

Once all the dependent helm charts have been downloaded successfully, install Mayastor. In the sample command given below, the release name is set as mayastor and namespace as mayastor.

{% tabs %}
{% tab title="Command" %}
```text
helm install mayastor . -n mayastor
```
{% endtab %}

{% tab title="Sample Output" %}
```text
NAME: mayastor
LAST DEPLOYED: Mon Sep  5 13:15:50 2022
NAMESPACE: mayastor
STATUS: deployed
REVISION: 1
NOTES:
OpenEBS Mayastor has been installed. Check its status by running:
$ kubectl get pods -n mayastor

For more information or to view the documentation, visit our website at https://openebs.io.
```
{% endtab %}
{% endtabs %}

To verify the installation, execute:

{% tabs %}
{% tab title="Command" %}
```text
kubectl get pods -n mayastor
```
{% endtab %}

{% tab title="Sample Output" %}
```text
NAME                                          READY   STATUS    RESTARTS   AGE
mayastor-agent-core-6d9d498df-vlttj           1/1     Running   0          2m41s
mayastor-api-rest-5c79485686-p5q88            1/1     Running   0          2m41s
mayastor-csi-controller-5dd99f559f-6cpnb      3/3     Running   0          2m41s
mayastor-csi-node-l2v7n                       2/2     Running   0          2m41s
mayastor-csi-node-wlkzc                       2/2     Running   0          2m41s
mayastor-csi-node-wsrxm                       2/2     Running   0          2m41s
mayastor-etcd-0                               1/1     Running   0          2m41s
mayastor-etcd-1                               1/1     Running   0          2m41s
mayastor-etcd-2                               1/1     Running   0          2m41s
mayastor-io-engine-4ns8d                      2/2     Running   0          2m41s
mayastor-io-engine-jmwzs                      2/2     Running   0          2m41s
mayastor-io-engine-vn9cf                      2/2     Running   0          2m41s
mayastor-loki-0                               1/1     Running   0          2m41s
mayastor-operator-diskpool-764fcf5d46-fw7jl   1/1     Running   0          2m41s
mayastor-promtail-c6t5w                       1/1     Running   0          2m41s
mayastor-promtail-mwtwb                       1/1     Running   0          2m41s
mayastor-promtail-xz8rv                       1/1     Running   0          2m41s

```
{% endtab %}
{% endtabs %}
