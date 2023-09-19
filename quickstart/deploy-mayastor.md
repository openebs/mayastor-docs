# Deploy Mayastor

## Overview

{% hint style="note" %}
Before deploying and using Mayastor please consult the [Known Issues](https://mayastor.gitbook.io/introduction/quickstart/known-issues) section of this guide.
{% endhint %}

The steps and commands which follow are intended only for use in conjunction with Mayastor version(s) 2.1.x and above.

## Installation via helm

{% hint style="info" %}
The Mayastor Helm chart now includes the Dynamic Local Persistent Volume (LocalPV) provisioner as the default option for provisioning storage to etcd and Loki. This simplifies storage setup by utilizing local volumes within your Kubernetes cluster.
For etcd, the chart uses the `mayastor-etcd-localpv` storage class, and for Loki, it utilizes the `mayastor-loki-localpv` storage class. These storage classes are bundled with the Mayastor chart, ensuring that your etcd and Loki instances are configured to use openEbs localPV volumes efficiently. 
`/var/local/{{ .Release.Name }}` paths should be persistent accross reboots.
{% endhint %}

1.  Add the OpenEBS Mayastor Helm repository.
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
mayastor/mayastor	2.4.0        	2.4.0       	Mayastor Helm chart for Kubernetes
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
To discover all the versions (including unstable versions), execute:
`helm search repo mayastor --devel --versions`
{% endhint %}


3. Run the following command to install Mayastor _version 2.4.
{% tabs %}
{% tab title="Command" %}
```text
helm install mayastor mayastor/mayastor -n mayastor --create-namespace --version 2.4.0
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
{% tab title="Sample Output for a three Mayastor node cluster" %}
```text
NAME                                         READY   STATUS    RESTARTS   AGE
mayastor-agent-core-6c485944f5-c65q6         2/2     Running   0          2m13s
mayastor-agent-ha-node-42tnm                 1/1     Running   0          2m14s
mayastor-agent-ha-node-45srp                 1/1     Running   0          2m14s
mayastor-agent-ha-node-tzz9x                 1/1     Running   0          2m14s
mayastor-api-rest-5c79485686-7qg5p           1/1     Running   0          2m13s
mayastor-csi-controller-65d6bc946-ldnfb      3/3     Running   0          2m13s
mayastor-csi-node-f4fgd                      2/2     Running   0          2m13s
mayastor-csi-node-ls9m4                      2/2     Running   0          2m13s
mayastor-csi-node-xtcfc                      2/2     Running   0          2m13s
mayastor-etcd-0                              1/1     Running   0          2m13s
mayastor-etcd-1                              1/1     Running   0          2m13s
mayastor-etcd-2                              1/1     Running   0          2m13s
mayastor-io-engine-f2wm6                     2/2     Running   0          2m13s
mayastor-io-engine-kqxs9                     2/2     Running   0          2m13s
mayastor-io-engine-m44ms                     2/2     Running   0          2m13s
mayastor-loki-0                              1/1     Running   0          2m13s
mayastor-obs-callhome-5f47c6d78b-fzzd7       1/1     Running   0          2m13s
mayastor-operator-diskpool-b64b9b7bb-vrjl6   1/1     Running   0          2m13s
mayastor-promtail-cglxr                      1/1     Running   0          2m14s
mayastor-promtail-jc2mz                      1/1     Running   0          2m14s
mayastor-promtail-mr8nf                      1/1     Running   0          2m14s
```
{% endtab %}
{% endtabs %} 
