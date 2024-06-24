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
