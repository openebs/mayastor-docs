# Preparing the Cluster

## Configure Mayastor Nodes \(MSNs\)

Within the context of the Mayastor application, a "Mayastor Node" is a Kubernetes worker node on which is scheduled an instance of a Mayastor data plane pod, so is thus capable of hosting a Storage Pool and exporting Persistent Volumes\(PV\).  A MSN makes use of block storage device\(s\) attached to it to contribute storage capacity to its pool\(s\), which supply backing storage for the Persistent Volumes provisioned on the parent cluster by Mayastor.

Kubernetes worker nodes are not required to be MSNs in order to be able to mount Mayastor-provisioned Persistent Volumes for the application pods scheduled on them.  New MSN nodes can be provisioned within the cluster at any time after the initial deployment, as aggregate demands for capacity, performance and availability levels increase.

### Verify / Enable Huge Page Support

_2MiB-sized_  Huge Pages must be supported and enabled on a MSN.  A minimum number of 512 such pages \(i.e. 1GiB total\) must be available on each node, which should be verified thus:

```text
grep HugePages /proc/meminfo

AnonHugePages:         0 kB
ShmemHugePages:        0 kB
HugePages_Total:    1024
HugePages_Free:      671
HugePages_Rsvd:        0
HugePages_Surp:        0

```

If fewer than 512 pages are available then the page count should be reconfigured as required, accounting for any other workloads which may be co-resident on the worker node and which also require them.  For example:

```text
echo 512 | sudo tee /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages
```

This change should also be made persistent across reboots by adding the required value to the file`/etc/sysctl.conf` like so:

```text
echo vm.nr_hugepages = 512 | sudo tee -a /etc/sysctl.conf
```

{% hint style="warning" %}
If you modify the Huge Page configuration of a node, you _must_ either restart kubelet or reboot the node.  Mayastor will not deploy correctly if the available Huge Page count as reported by the node's kubelet instance does not satisfy the minimum requirements.
{% endhint %}

### Label Mayastor Node Candidates

All worker nodes in the cluster which are intended to operate as MSNs should be labelled with the OpenEBS engine type "mayastor".  This label is used as a selector by the Mayastor Daemonset, which will be deployed during the next stage of the installation.  Here we demonstrate the correct labeling of a worker node named "node1":  

```text
kubectl label node node1 openebs.io/engine=mayastor
```



