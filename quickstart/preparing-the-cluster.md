# Preparing the Cluster

### Verify / Enable Huge Page Support

_2MiB-sized_  Huge Pages must be supported and enabled on the mayastor storage nodes.  A minimum number of 1024 such pages \(i.e. 2GiB total\) must be available _exclusively_ to the Mayastor pod on each node, which should be verified thus:

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

{% hint style="warning" %}
If you set `csi.node.topology.nodeSelector: true`, then you will need to label the worker nodes accordingly to `csi.node.topology.segments`. Both csi-node and agent-ha-node Daemonsets will include the topology segments into the node selector.
{% endhint %}

