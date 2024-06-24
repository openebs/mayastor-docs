# Storage class parameters

{% hint style="danger" %}
This website/page will be End-of-life (EOL) after 31 August 2024. We recommend you to visit [OpenEBS Documentation](https://openebs.io/docs/user-guides/replicated-storage-user-guide/replicated-pv-mayastor/rs-installation) for the latest Mayastor documentation (v2.6 and above).
 
Mayastor is now also referred to as OpenEBS Replicated PV Mayastor.
{% endhint %}

Storage class resource in Kubernetes is used to supply parameters to volumes when they are created. It is a convenient way of grouping volumes with common characteristics. All parameters take a string value. Brief explanation of each supported Mayastor parameter follows.


{% hint style="info" %}

The storage class parameter `local` has been deprecated and is a breaking change in Mayastor version 2.0. Ensure that this parameter is not used.
 
{% endhint %}


## "fsType"

File system that will be used when mounting the volume. The default file system when not specified is 'ext4'. We recommend to use 'xfs' that is considered to be more advanced and performant. Though make sure that XFS is installed on all nodes in the cluster before using it.

## "ioTimeout"

Expressed in seconds and it sets the `io_timeout` parameter in the linux block device driver for Mayastor volume. It also sets `ctrl-loss-tmo` timeout in linux NVMe driver that is used for detecting inaccessible target (nexus in our case). In either case, if IO cannot be served by Mayastor volume, because it is stuck or because the nvmf target is not there, it will take roughly this time to the initiator to return an error to data consumer, which can be either filesystem or an application if using raw block volume. The usual reaction of a filesystem to such error is to switch the filesystem to read-only state and require manual intervention from user - remounting. Be careful when setting this to a low value because any node reboot that does not fit into this time interval may require manual intervention afterwards or can result in application failure - depending on how the error is handled by the application.
 
{% hint style="info" %}
The setting is supported only when using "nvmf" protocol.
{% endhint %}

## "protocol"

The parameter 'protocol' takes the value `nvmf`(NVMe over TCP protocol). It is used to mount the volume (target) on the application node.

## "repl"

The string value should be a number and the number should be greater than zero. Mayastor control plane will try to keep always this many copies of the data if possible. If set to one then the volume does not tolerate any node failure. If set to two, then it tolerates one node failure. If set to three, then two node failures, etc.

## "thin"

The volumes can either be `thick` or `thin` provisioned. Adding the `thin` parameter to the StorageClass YAML allows the volume to be thinly provisioned. To do so, add `thin: true` under the `parameters` spec, in the StorageClass YAML. [Sample YAML](https://mayastor.gitbook.io/introduction/quickstart/configure-mayastor#create-mayastor-storageclass-s)
When the volumes are thinly provisioned, the user needs to monitor the pools, and if these pools start to run out of space, then either new pools must be added or volumes deleted to prevent thinly provisioned volumes from getting degraded or faulted. This is because when a pool with more than one replica runs out of space, Mayastor moves the largest out-of-space replica to another pool and then executes a rebuild. It then checks if all the replicas have sufficient space; if not, it moves the next largest replica to another pool, and this process continues till all the replicas have sufficient space.

{% hint style="info" %}
The capacity usage on a pool can be monitored using [exporter metrics](https://mayastor.gitbook.io/introduction/advanced-operations/monitoring#pool-metrics-exporter).
{% endhint %}

The `agents.core.capacity.thin` spec present in the Mayastor helm chart consists of the following configurable parameters that can be used to control the scheduling of thinly provisioned replicas:

1. **poolCommitment** parameter specifies the maximum allowed pool commitment limit (in percent).
2. **volumeCommitment** parameter specifies the minimum amount of free space that must be present in each replica pool in order to create new replicas for an existing volume. This value is specified as a percentage of the volume size.
3. **volumeCommitmentInitial** minimum amount of free space that must be present in each replica pool in order to create new replicas for a new volume. This value is specified as a percentage of the volume size.


{% hint style="info" %}
Note:
1. By default, the volumes are provisioned as `thick`. 
2. For a pool of a particular size, say 10 Gigabytes, a volume > 10 Gigabytes cannot be created, as Mayastor 2.3.0 does not support pool expansion.
3. The replicas for a given volume can be either all thick or all thin. Same volume cannot have a combination of thick and thin replicas
{% endhint %}


## "stsAffinityGroup" 

 `stsAffinityGroup` represents a collection of volumes that belong to instances of Kubernetes StatefulSet. When a StatefulSet is deployed, each instance within the StatefulSet creates its own individual volume, which collectively forms the `stsAffinityGroup`. Each volume within the `stsAffinityGroup` corresponds to a pod of the StatefulSet.

This feature enforces the following rules to ensure the proper placement and distribution of replicas and targets so that there isn't any single point of failure affecting multiple instances of StatefulSet.

1. Anti-Affinity among single-replica volumes :
 This rule ensures that replicas of different volumes are distributed in such a way that there is no single point of failure. By avoiding the colocation of replicas from different volumes on the same node.

2.  Anti-Affinity among multi-replica volumes : 

If the affinity group volumes have multiple replicas, they already have some level of redundancy. This feature ensures that in such cases, the replicas are distributed optimally for the stsAffinityGroup volumes.


3. Anti-affinity among targets :

The [High Availability](https://mayastor.gitbook.io/introduction/advanced-operations/ha) feature ensures that there is no single point of failure for the targets.
The `stsAffinityGroup` ensures that in such cases, the targets are distributed optimally for the stsAffinityGroup volumes.

By default, the `stsAffinityGroup` feature is disabled. To enable it, modify the storage class YAML by setting the `parameters.stsAffinityGroup` parameter to true.

