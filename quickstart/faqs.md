# FAQs

{% hint style="danger" %}
This website/page will be End-of-life (EOL) after 31 August 2024. We recommend you to visit [OpenEBS Documentation](https://openebs.io/docs/user-guides/replicated-storage-user-guide/replicated-pv-mayastor/rs-installation) for the latest Mayastor documentation (v2.6 and above).
 
Mayastor is now also referred to as OpenEBS Replicated PV Mayastor.
{% endhint %}

## How does it help to keep my data safe?

Mayastor's storage engine supports synchronous mirroring to enhance the durability of data at rest within whatever physical persistence layer is in use. When volumes are provisioned which are configured for replication \(a user can control the count of active replicas which should be maintained, on a per StorageClass basis\), write I/O operations issued by an application to that volume are amplified by its controller ("nexus") and dispatched to all its active replicas. Only if every replica completes the write successfully on its own underlying block device will the I/O completion be acknowledged to the controller. Otherwise, the I/O is failed and the caller must make its own decision as to whether it should be retried. If a replica is determined to have faulted \(I/O cannot be serviced within the configured timeout period, or not without error\), the control plane will automatically take corrective action and remove it from the volume. If spare capacity is available within a Mayastor pool, a new replica will be created as a replacement and automatically brought into synchronisation with the existing replicas. The data path for a replicated volume is described in more detail [here](https://github.com/openebs/mayastor-docs/blob/master/reference/i-o-path-description.md#replicated-volume-io-path)

## How is it configured?

This Mayastor documentation contains sections which are focused on initial, 'quickstart' deployment scenarios, including the correct configuration of underlying hardware and software, and of Mayastor features such as "Storage Nodes" \(MSNs\) and "Disk Pools" \(MSPs\). Information describing tuning for the optimisation of performance is also provided.

* [Quickstart Guide](https://mayastor.gitbook.io/introduction/quickstart/configure-mayastor)
* [Performance Tips](https://mayastor.gitbook.io/introduction/quickstart/performance-tips)


## What is the basis for its performance claims?  Has it been benchmarked?

Mayastor has been built to leverage the performance potential of contemporary, high-end, solid state storage devices as a foremost design consideration. For this reason, the I/O path is predicated on NVMe, a transport which is both highly CPU efficient and which demonstrates highly linear resource scaling. The data path runs entirely within user space, also contributing efficiency gains as syscalls are avoided, and is both interrupt and lock free.

MayaData has performed its own benchmarking tests in collaboration with Intel, using latest generation Intel P5800X Optane devices "The World's Fastest Data Centre SSD". In those tests it was determined that, on average, across a range of read/write ratios and both with and without synchronous mirroring enabled, the overhead imposed by the Mayastor I/O path was well under 10% \(in fact, much closer to 6%\).

Further information regarding the testing performed may be found [here](https://openebs.io/blog/mayastor-nvme-of-tcp-performance)

## What is the on-disk format used by Disk Pools?  Is it also open source?

Mayastor makes use of parts of the open source [Storage Performance Development Kit \(SPDK\)](https://spdk.io/) project, contributed by Intel.  Mayastor's Storage Pools use the SPDK's Blobstore structure as their on-disk persistence layer. Blobstore structures and layout are [documented](https://github.com/spdk/spdk/blob/master/doc/blob.md).

Since the replicas \(data copies\) of Mayastor volumes are held entirely within Blobstores, it is not possible to directly access the data held on pool's block devices from outside of Mayastor. Equally, Mayastor cannot directly 'import' and use existing volumes which aren't of Mayastor origin. The project's maintainers are considering alternative options for the persistence layer which may support such data migration goals.

## Can the size / capacity of a Disk Pool be changed?

The size of a Mayastor Pool is fixed at the time of creation and is immutable. A single pool may have only one block device as a member. These constraints may be removed in later versions.

## How can I ensure that replicas aren't scheduled onto the same node?  How about onto nodes in the same rack / availability zone?

The replica placement logic of Mayastor's control plane doesn't permit replicas of the same volume to be placed onto the same node, even if it were to be within different Disk Pools. For example, if a volume with replication factor 3 is to be provisioned, then there must be three healthy Disk Pools available, each with sufficient free capacity and each located on its own Mayastor node. Further enhancements to topology awareness are under consideration by the maintainers.

## How can I see the node on which the active nexus for a particular volume resides?

The Mayastor kubectl plugin is used to obtain this information.

## Is there a way to automatically rebalance data across available nodes?  Can data be manually re-distributed?

No.  This may be a feature of future releases.

## Can mayastor do async replication to a node in the same cluster?  How about a different cluster?

Mayastor does not peform asynchronous replication.

## Does Mayastor support RAID?

Mayastor pools do not implement any form of RAID, erasure coding or striping. If higher levels of data redundancy are required, Mayastor volumes can be provisioned with a replication factor of greater than 1, which will result in synchronously mirrored copies of their data being stored in multiple Disk Pools across multiple Storage Nodes. If the block device on which a Disk Pool is created is actually a logical unit backed by its own RAID implementation \(e.g. a Fibre Channel attached LUN from an external SAN\) it can still be used within a Mayastor Disk Pool whilst providing protection against physical disk device failures.


## Does Mayastor perform compression and/or deduplication?

No.

## Does Mayastor support snapshots?  Clones?

No but these may be features of future releases.

## Which CPU architectures are supported?  What are the minimum hardware requirements?

Mayastor nightly builds and releases are compiled and tested on x86-64, under Ubuntu 20.04 LTS with a 5.13 kernel. Some effort has been made to allow compilation on ARM platforms but this is currently considered experimental and is not subject to integration or end-to-end testing by Mayastor's maintainers.

Minimum hardware requirements are discussed in the [quickstart section](https://mayastor.gitbook.io/introduction/quickstart/prerequisites) of this documentation.

Mayastor does not run on Raspbery Pi as the version of the SPDK used by Mayastor requires ARMv8 Crypto extensions which are not currently available for Pi.

## Does Mayastor suffer from TCP congestion when using NVME-TCP?

Mayastor, as any other solution leveraging TCP for network transport, may suffer from network congestion as TCP will try to slow down transfer speeds. It is important to keep an eye on networking and fine-tune TCP/IP stack as appropriate. This tuning can include \(but is not limited to\) send and receive buffers, MSS, congestion control algorithms \(e.g. you may try DCTCP\) etc.

## Why do Mayastor pods show high levels of CPU utilization when there is little or no I/O being processed?

Mayastor has been designed so as to be able to leverage the peformance capabilities of contemporary high-end solid-state storage devices.  A significant aspect of this is the selection of a polling based I/O service queue, rather than an interrupt driven one.  This minimises the latency introduced into the data path but at the cost of additional CPU utilisation by the "reactor" - the poller operating at the heart of the Mayastor pod.  When Mayastor pods have been deployed to a cluster, it is expected that these daemonset instances will make full utilization of their CPU allocation, even when there is no I/O load on the cluster.  This is simply the poller continuing to operate at full speed, waiting for I/O.  For the same reason, it is recommended that when configuring the CPU resource limits for the Mayastor daemonset, only full, not fractional, CPU limits are set; fractional allocations will also incur additional latency, resulting in a reduction in overall performance potential. The extent to which this performance degradation is noticeable in practice will depend on the performance of the underlying storage in use, as well as whatvever other bottlenecks/constraints may be present in the system as cofigured.


## Does the supportability tool expose sensitive data?

The supportability tool generates support bundles, which are used for debugging purposes. These bundles are created in response to the user's invocation of the tool and can be transmitted only by the user. To view the list of collected information, visit the [supportability section](https://mayastor.gitbook.io/introduction/reference/supportability#does-the-supportability-tool-expose-sensistive-data).


## What happens when a PV with reclaim policy set to `retain` is deleted?

In Kubernetes, when a PVC is created with the reclaim policy set to 'Retain', the PV bound to this PVC is not deleted even if the PVC is deleted. One can manually delete the PV by issuing the command "kubectl delete pv ", however the underlying storage resources could be left behind as the CSI volume provisioner (external provisioner) is not aware of this. To resolve this issue of dangling storage objects, Mayastor has introduced a PV garbage collector. This PV garbage collector is deployed as a part of the Mayastor CSI controller-plugin.

  ### How does the PV garbage collector work?

   The PV garbage collector deploys a watcher component, which subscribes to the Kubernetes Persistent Volume deletion events. When a PV is deleted, an event is generated by the Kubernetes API server and is received by this component. Upon a successful validation of this event, the garbage collector deletes the corresponding Mayastor volume resources.


## How to disable cow for btrfs filesystem?

To disbale cow for `btrfs` filesystem, use `nodatacow` as a mountOption in the storage class which would be used to provision the volume.