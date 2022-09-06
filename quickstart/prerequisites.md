# Prerequisites

## **General**

All worker nodes must satisfy the following requirements:

* **x86-64** CPU cores with SSE4.2 instruction support:
  * First generation Intel Core i5 and Core i7 (Nehalem microarchitecture) or newer
  * AMD Bulldozer processor and newer

* Linux kernel **5.4 or higher** with the following modules loaded:
  * nvme-tcp
  * ext4 and optionally xfs

* [Helm](https://helm.sh/docs/intro/install/) version must be v3.7 or later.

* Supported Kubernetes versions: 
  - [v1.23.1](https://v1-23.docs.kubernetes.io/)
  - [v1.22.3](https://v1-22.docs.kubernetes.io/)
  - [v1.21.8](https://v1-21.docs.kubernetes.io/)


## Networking requirements

* An Ethernet connection with a minimum speed of 10 Gbps between the nodes must be available at all times.

* Ensure that the following ports are **not** in use on the node:
  - **10124**: Mayastor gRPC server will use this port.
  - **8420 / 4421**: NVMf targets will use these ports.

* The firewall settings should not restrict connection to the node.


## Recommended resource requirements

The resource requirements for deployment of Mayastor are as follows:

{% tabs %}

{% tab title="io-engine DaemonSet" %}

```text

resources:
  limits:
    cpu: "2"
    memory: "1Gi"
    hugepages-2Mi: "2Gi"
  requests:
    cpu: "2"
    memory: "1Gi"
    hugepages-2Mi: "2Gi"
```

{% endtab %}

{% endtabs %}


{% tabs %}

{% tab title="csi-node DaemonSet" %}

```text

resources:
  limits:
    cpu: "100m"
    memory: "50Mi"
  requests:
    cpu: "100m"
    memory: "50Mi"

```

{% endtab %}

{% endtabs %}


{% tabs %}

{% tab title="csi-controller Deployment" %}

```text

resources:
  limits:
    cpu: "32m"
    memory: "128Mi"
  requests:
    cpu: "16m"
    memory: "64Mi"

```

{% endtab %}

{% endtabs %}


{% tabs %}

{% tab title="api-rest Deployment" %}

```text

resources:
  limits:
    cpu: "100m"
    memory: "64Mi"
  requests:
    cpu: "50m"
    memory: "32Mi"

```

{% endtab %}

{% endtabs %}


{% tabs %}

{% tab title="agent-core" %}

```text

resources:
  limits:
    cpu: "1000m"
    memory: "32Mi"
  requests:
    cpu: "500m"
    memory: "16Mi"

```

{% endtab %}

{% endtabs %}


{% tabs %}

{% tab title="operator-diskpool" %}

```text

resources:
  limits:
    cpu: "100m"
    memory: "32Mi"
  requests:
    cpu: "50m"
    memory: "16Mi"

```

{% endtab %}

{% endtabs %}


## DiskPool requirements

* Disks must be unpartitioned, unformatted, and used exclusively by the DiskPool.
* The minimum capacity of the disks should be 10 GB.

## RBAC permission requirements

* Kubernetes core v1 API-group resources: Pod, Event, Node, Namespace, ServiceAccount, PersistentVolume, PersistentVolumeClaim, ConfigMap, Secret, Service, Endpoint, Event.

* Kubernetes batch API-group resources: CronJob, Job

* Kubernetes apps API-group resources: Deployment, ReplicaSet, StatefulSet, DaemonSet

* Kubernetes `storage.k8s.io` API-group resources: StorageClass, VolumeSnapshot, VolumeSnapshotContent, VolumeAttachment, CSI-Node

* Kubernetes `apiextensions.k8s.io` API-group resources: CustomResourceDefinition

* Mayastor Custom Resources that is `openebs.io` API-group resources: DiskPool

* Custom Resources from Helm chart dependencies of Jaeger that is helpful for debugging:

   - ConsoleLink Resource from `console.openshift.io` API group

   - ElasticSearch Resource from `logging.openshift.io` API group

   - Kafka and KafkaUsers from `kafka.strimzi.io` API group

   - ServiceMonitor from `monitoring.coreos.com` API group

   - Ingress from `networking.k8s.io` API group and from extensions API group

   - Route from `route.openshift.io` API group

   - All resources from `jaegertracing.io` API group


## Minimum worker node count

    The minimum supported worker node count is three nodes. When using the synchronous replication feature (N+1 mirroring), the number of worker nodes to which Mayastor is deployed should be no less than the desired replication factor.


## Transport protocols
    
    Mayastor supports the export and mounting of volumes over NVMe-oF TCP only. Worker node(s) on which a volume may be scheduled (to be mounted) must have the requisite initiator support installed and configured.
    In order to reliably mount application volumes over NVMe-oF TCP, a worker node's kernel version must be 5.4 or later and the nvme-tcp kernel module must be loaded.



