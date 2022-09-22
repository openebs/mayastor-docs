# Deploy Mayastor

## Overview

{% hint style="note" %}
Before deploying and using Mayastor please consult the [Known Issues](https://mayastor.gitbook.io/introduction/quickstart/known-issues) section of this guide.
{% endhint %}

The steps and commands which follow are intended only for use with, and have only been tested against, the _latest release for version 2.x_. Earlier releases or development versions may require a modified or different installation process.

## Installation via helm

1. Add the `mayastor-extensions` helm repository.
{% tabs %}
{% tab title="Command" %}
```text
helm repo add mayastor https://openebs.github.io/mayastor-extensions/ 
```
{% endtab %}
{% tab title="Output" %}
```text
"mayastor" has been added to your repositories
```
{% endtab %}
{% endtabs %}


Run the following command to discover all the _stable versions_ of the added chart repository:

{% tabs %}
{% tab title="Command" %}
```text
helm search repo mayastor --versions
```
{% endtab %}
{% tab title="Sample Output" %}
```text
 NAME             	CHART VERSION	APP VERSION  	DESCRIPTION                       
mayastor/mayastor	v2.0.0        	v2.0.0       	Mayastor Helm chart for Kubernetes
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
To discover all the versions (including unstable versions), execute:
`helm search repo mayastor --devel --versions`
{% endhint %}


3. Run the following command to install Mayastor _version 2.0_.
{% tabs %}
{% tab title="Command" %}
```text
helm install mayastor mayastor/mayastor -n mayastor --create-namespace
```
{% endtab %}
{% tab title="Sample Output" %}
```text
NAME: mayastor
LAST DEPLOYED: Thu Sep 22 18:59:56 2022
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

Verify the status of the pods by running the command:

{% tabs %}
{% tab title="Command" %}
```text
kubectl get pods -n mayastor
```
{% endtab %}
{% tab title="Sample Output" %}
```text
NAME                                         READY   STATUS             RESTARTS   AGE
mayastor-agent-core-6d9d498df-kz6t4          1/1     Running            0          25m
mayastor-agent-ha-node-cx8vn                 1/1     Running            0          25m
mayastor-agent-ha-node-fc98l                 1/1     Running            0          25m
mayastor-agent-ha-node-tdcfx                 1/1     Running            0          25m
mayastor-api-rest-5c79485686-sw2ll           1/1     Running            0          25m
mayastor-csi-controller-65d6bc946-qvl2h      3/3     Running            0          25m
mayastor-csi-node-g8ngz                      2/2     Running            0          25m
mayastor-csi-node-w86kw                      2/2     Running            0          25m
mayastor-csi-node-z6lc9                      2/2     Running            0          25m
mayastor-etcd-0                              1/1     Running            0          25m
mayastor-etcd-1                              1/1     Running            0          25m
mayastor-etcd-2                              1/1     Running            0          25m
mayastor-io-engine-72zp6                     2/2     Running            0          25m
mayastor-io-engine-gk7z2                     2/2     Running            0          25m
mayastor-io-engine-wfld7                     2/2     Running            0          25m
mayastor-loki-0                              1/1     Running            0          25m
mayastor-obs-callhome-5f47c6d78b-h7x98       1/1     Running            0          25m
mayastor-operator-diskpool-b64b9b7bb-m7l6c   1/1     Running            0          25m
mayastor-promtail-cw8lc                      1/1     Running            0          25m
mayastor-promtail-kbp4t                      1/1     Running            0          25m
mayastor-promtail-xnhcm                      1/1     Running            0          25m
```
{% endtab %}
{% endtabs %}