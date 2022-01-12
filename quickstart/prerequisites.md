# Prerequisites

### **General**

Each worker node that will host an instance of a Mayastor pod, must have the following resources _free and available_ for use by Mayastor:

* **Two  x86-64 CPU cores with SSE4.2 instruction support**:
  * Intel Nehalem processor \(march=nehalem\) and newer
  * AMD Bulldozer processor and newer
* **4GiB of RAM**
* HugePage support
  * A minimum of **2GiB of** **2MiB-sized** **huge pages**
* Linux kernel **5.4 or higher** with the following modules loaded:
  * nvme-tcp
  * ext4 and optionally xfs

{% hint style="info" %}
Instances of the Mayastor pod _must_ be run in privileged mode
{% endhint %}

### Node Count

As long as the resource prerequisites are met, Mayastor can deployed to a cluster with just a single worker node.  However note that in order to evaluate the synchronous replication feature \(N+1 mirroring\), the number of worker nodes to which Mayastor is deployed should be no less than the desired replication factor.  E.g. three-way mirroring of Persistent Volumes \(PV\) would require Mayastor to be deployed to a minimum of three worker nodes.

### Transport Protocols

Mayastor supports the export and mounting of a Persistent Volume over NVMe-oF TCP. Worker node\(s\) on which a PV is to be mounted must have the requisite initiator support installed and configured for the protocol in use.

#### NVMe-oF

In order to reliably mount application PVs  over NVMe-oF TCP, a worker node's kernel version must be 5.3 or later.  Verify that the `nvme-tcp`  kernel module is loaded and if necessary, load it.



