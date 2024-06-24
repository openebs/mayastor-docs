# Prerequisites

{% hint style="danger" %}
This website/page will be End-of-life (EOL) after 31 August 2024. We recommend you to visit [OpenEBS Documentation](https://openebs.io/docs/user-guides/replicated-storage-user-guide/replicated-pv-mayastor/rs-installation) for the latest Mayastor documentation (v2.6 and above).
 
Mayastor is now also referred to as OpenEBS Replicated PV Mayastor.
{% endhint %}

## **General**

All worker nodes must satisfy the following requirements:

* **x86-64** CPU cores with SSE4.2 instruction support
* Linux kernel **5.13 or higher** with the following modules loaded:
  * nvme-tcp
  * ext4 and optionally xfs

## **Mayastor pods (Data Plane/Engine)**
Each worker node which will host an instance of a Mayastor pod must have the following resources _free and available_ for _exclusive_ use by that pod:

* Two CPU cores
* 4GiB RAM
* **HugePage support**
  * A minimum of **2GiB of** **2MiB-sized** pages

{% hint style="info" %}
Instances of the Mayastor pod _must_ be run in privileged mode
{% endhint %}


## **Worker Node Count**

As of version 1.0 the minimum supported worker node count is three nodes.

Note also, that when using the synchronous replication feature \(N+1 mirroring\), the number of worker nodes to which Mayastor pods are deployed should be no less than the desired replication factor.  E.g. four-way mirroring of a volume would require Mayastor pods to be deployed to a minimum of four worker nodes within the cluster.

## **Transport Protocols**

As of version 1.0, Mayastor supports the export and mounting of volumes over NVMe-oF TCP _only_. Worker nodes on which an application pod using Mayastor volume(s) may be scheduled must have the requisite NVMe initiator support installed and configured.



