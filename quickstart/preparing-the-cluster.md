# Preparing the Cluster

{% hint style="danger" %}
This website/page will be End-of-life (EOL) after 31 August 2024. We recommend you to visit [OpenEBS Documentation](https://openebs.io/docs/user-guides/replicated-storage-user-guide/replicated-pv-mayastor/rs-installation) for the latest Mayastor documentation (v2.6 and above).
 
Mayastor is now also referred to as OpenEBS Replicated PV Mayastor.
{% endhint %}

## Configure Mayastor Nodes

In the context of the Mayastor application, a "Mayastor Node" \(MSN\) is a Kubernetes worker node on which is scheduled an instance of a Mayastor data plane pod and which is thus capable of hosting storage "pools" and exporting Persistent Volumes \(PV\) to application pods within the cluster.  A MSN makes use of block storage device\(s\) attached to it to contribute storage capacity to its pool\(s\), which supply backing storage for the Persistent Volumes dynamically-provisioned on the cluster by Mayastor.

{% hint style="info" %}
In this version of Mayastor, a worker node _MUST_ be configured as a MSN in order for it to be able to mount PVs provisioned by the Mayastor control plane.  That is to say, application pods using Mayastor volumes can only be successfully scheduled on MSNs.  It is not necessary for a MSN to have any Pools configured on it, if it is required only to mount volumes for applications rather than host data replicas for them.

This restriction will be removed in version 2.0
{% endhint %}

New MSN nodes can be provisioned within the cluster at any time after the initial deployment, as aggregate demands for capacity, performance and availability levels increase.

### Verify / Enable Huge Page Support

_2MiB-sized_  Huge Pages must be supported and enabled on a MSN.  A minimum number of 1024 such pages \(i.e. 2GiB total\) must be available _exclusively_ to the Mayastor pod on each node, which should be verified thus:

```text
grep HugePages /proc/meminfo

AnonHugePages:         0 kB
ShmemHugePages:        0 kB
HugePages_Total:    1024
HugePages_Free:      671
HugePages_Rsvd:        0
HugePages_Surp:        0

```

If fewer than 1024 pages are available then the page count should be reconfigured on the worker node as required, accounting for any other workloads which may be scheduled on the same node and which also require them.  For example:

```text
echo 1024 | sudo tee /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages
```

This change should also be made persistent across reboots by adding the required value to the file`/etc/sysctl.conf` like so:

```text
echo vm.nr_hugepages = 1024 | sudo tee -a /etc/sysctl.conf
```

{% hint style="warning" %}
If you modify the Huge Page configuration of a node, you _MUST_ either restart kubelet or reboot the node.  Mayastor will not deploy correctly if the available Huge Page count as reported by the node's kubelet instance does not satisfy the minimum requirements.
{% endhint %}

### Label Mayastor Node Candidates

All worker nodes which will have Mayastor pods running on them must be labelled with the OpenEBS engine type "mayastor".  This label will be used as a node selector by the Mayastor Daemonset, which is deployed as a part of the Mayastor data plane components installation. To add this label to a node, execute:

```text
kubectl label node <node_name> openebs.io/engine=mayastor
```



