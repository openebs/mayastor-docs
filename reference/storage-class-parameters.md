# Storage class parameters

Storage class resource in Kubernetes is used to supply parameters to volumes when they are created. It is a convenient way of grouping volumes with common characteristics. All parameters take a string value. Brief explanation of each supported Mayastor parameter follows.

## "fsType"

File system that will be used when mounting the volume. The default file system when not specified is 'ext4'. We recommend to use 'xfs' that is considered to be more advanced and performant. Though make sure that XFS is installed on all nodes in the cluster before using it.

## "ioTimeout"

Expressed in seconds and it sets the `io_timeout` parameter in the linux block device driver for Mayastor volume. It also sets `ctrl-loss-tmo` timeout in linux NVMe driver that is used for detecting inaccessible target (nexus in our case). In either case, if IO cannot be served by Mayastor volume, because it is stuck or because the nvmf target is not there, it will take roughly this time to the initiator to return an error to data consumer, which can be either filesystem or an application if using raw block volume. The usual reaction of a filesystem to such error is to switch the filesystem to read-only state and require manual intervention from user - remounting. Be careful when setting this to a low value because any node reboot that does not fit into this time interval may require manual intervention afterwards or can result in application failure - depending on how the error is handled by the application.
 
{% hint style="info" %}
The setting is supported only when using "nvmf" protocol.
{% endhint %}

## "local"

It is a boolean-type flag with the default value of false. All following values are interpreted as "true" and anything else as "false": 'y', 'Y', 'yes', 'Yes', 'YES', 'true', 'True', 'TRUE', 'on', 'On', 'ON'. The flag controls scheduling of nexus and replicas to storage nodes in the cluster. Mayastor always tries to schedule nexus and one of the replicas on the same node as the application that is using the volume for two reasons:

1. minimal number of network hops in the IO path and
2. simplified failure recovery if the application and nexus run on the same node.

The only difference depending on the flag's value is what happens when it isn't possible to schedule the application and nexus on the same node. If the value of "local" is true, then the application will become unscheduleable and will not start. If "local" is false, then the application will run regardless but on a different node than the nexus.

{% hint style="info" %}
Note that in order to enable the locality feature, the `volumeBindingMode` in storage class must be set to `WaitForFirstConsumer`.
{% endhint %}

Is there any disadvantage of setting "local" to true? Well, there are at least two reasons why someone could want to use the default value of false:

1. Nexus can be created only on a storage node. That implies that all stateful applications must run on the storage nodes as well, which is not ideal in case when there are only a few storage nodes in the cluster that are overloaded.
2. The pod with the application will be limited to run only on a subset of storage nodes for its whole lifetime. The locality is enforced by means of "node affinity" on PVs which is immutable for the whole lifetime of the volume. Consider an example of a volume with three replicas. The PV will have node affinity set to three storage nodes that were initially chosen for hosting the replicas. The application using the volume could never run on different nodes then.

In spite of the severe implications mention above we *still recommend* to set "local" to true because otherwise your data might become inaccessible to the application if the node with the nexus goes down.

## "protocol"

Supported values are "nvmf" and "iscsi". It is the protocol that is used for mounting the volume (target) on the application node. Not to be confused with the protocol that is used between nexus and replicas, that is always "nvmf". By "nvmf" here, we mean NVMe over TCP protocol - the next generation protocol that is supposed to replace iSCSI. We definitely recommend to use "nvmf". "iscsi" is not full-featured and provided only for users running older kernels that do not support NVMe over TCP yet but still would like to give Mayastor a try.

## "repl"

The string value should be a number and the number should be greater than zero. Mayastor control plane will try to keep always this many copies of the data if possible. If set to one then the volume does not tolerate any node failure. If set to two, then it tolerates one node failure. If set to three, then two node failures, etc.
