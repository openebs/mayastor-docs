# Prerequisites

## **General**

All worker nodes must satisfy the following requirements:

* **x86-64** CPU cores with SSE4.2 instruction support:
  * Intel Nehalem processor \(march=nehalem\) and newer
  * AMD Bulldozer processor and newer

* Linux kernel **5.4 or higher** with the following modules loaded:
  * nvme-tcp
  * ext4 and optionally xfs

## **Mayastor pods (Data Plane/Engine)**
Each worker node which will host an instance of a Mayastor pod must have the following resources _free and available_ for use by Mayastor:

* Two CPU cores
* 4GiB RAM
* **HugePage support**
  * A minimum of **2GiB of** **2MiB-sized** pages

{% hint style="info" %}
Instances of the Mayastor pod _must_ be run in privileged mode
{% endhint %}


## **Worker Node Count**

As of version 1.0 the minimum supported worker node count is three nodes.

Note also, that when using the synchronous replication feature \(N+1 mirroring\), the number of worker nodes to which Mayastor is deployed should be no less than the desired replication factor.  E.g. four-way mirroring of a volume would require Mayastor pods to be deployed to a minimum of four worker nodes within the cluster.

## **Transport Protocols**

As of version 1.0, Mayastor supports the export and mounting of volumes over NVMe-oF TCP only. Worker node\(s\) on which a volume may be scheduled to be mounted must have the requisite initiator support installed and configured.

In order to reliably mount application volumes over NVMe-oF TCP, a worker node's kernel version must be 5.3 or later and the `nvme-tcp`  kernel module must be loaded.



