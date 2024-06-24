# Deploy Mayastor

{% hint style="danger" %}
This website/page will be End-of-life (EOL) after 31 August 2024. We recommend you to visit [OpenEBS Documentation](https://openebs.io/docs/user-guides/replicated-storage-user-guide/replicated-pv-mayastor/rs-installation) for the latest Mayastor documentation (v2.6 and above).
 
Mayastor is now also referred to as OpenEBS Replicated PV Mayastor.
{% endhint %}

## Overview

{% hint style="note" %}
Before deploying and using Mayastor please consult the [Known Issues](https://mayastor.gitbook.io/introduction/quickstart/known-issues) section of this guide.
{% endhint %}

The steps and commands which follow are intended only for use in conjunction with Mayastor version(s) 2.2.x.

## Installation via helm

{% hint style="info" %}
Installing Mayastor via the Helm chart sets the StorageClass as 'default' for Loki and etcd StatefulSets. This will work in clusters which have a StorageClass named 'default'. If there is no 'default' StorageClass in the cluster, storage provisioning will fail for Loki and etcd. In such cases, one can change the StorageClass to 'manual' which will provision a PersistentVolume of type hostPath. To do this, make the following changes in the `values.yaml` file:
- loki-stack.persistence.storageClassName: manual
- etcd.persistence.storageClassName: manual
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
mayastor/mayastor	2.2.0        	2.2.0       	Mayastor Helm chart for Kubernetes
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
To discover all the versions (including unstable versions), execute:
`helm search repo mayastor --devel --versions`
{% endhint %}


3. Run the following command to install Mayastor _version 2.2.
{% tabs %}
{% tab title="Command" %}
```text
helm install mayastor mayastor/mayastor -n mayastor --create-namespace --version 2.2.0
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
