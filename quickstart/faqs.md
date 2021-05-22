# OpenEBS Mayastor FAQs

## How does it help to keep my data safe?
Mayastor's approach is to as simply as possible create copies of data so that when a node is rescheduled, or lost, that otherwise would cause loss of the data on that node, instead the data is available elsewhere and is made available to the workload.  

Mayastor's storage engine, or "nexus", provides synchronous mirroring functionality to enhance the durability of data at rest within whatever physical persistence layer is in use.  When volumes are provisioned which are configured for replication \(a user can control the count of active replicas which should be maintained, on a per StorageClass basis\), write I/O operations issued by an application to that volume are 'amplified' by the nexus and sent to all active replicas.  Only if every replica completes the write successfully on its own underlying block device will the I/O completion be acknowledged.  Otherwise, the I/O is failed and must be retried by the caller.  If a replica is determined to have faulted \(I/O cannot be serviced within the configured timeout period, or not without error\), the control plane will automatically take corrective action and remove it from the volume.  If spare capacity is available within another Mayastor pool, a new replica will be created as a replacement and automatically brought into syncrhonisation with the existing replicas.  In short, Mayastor adds availability and consistency for data through ensuring that a minimum set of replicas ara available and by recovering from outages to retain that minimum number of replicas.  

The data path for a replicated volume is described in more detail [here](https://github.com/openebs/mayastor-docs/blob/master/reference/i-o-path-description.md#replicated-volume-io-path)

## How is it configured?
This OpenEBS Mayastor documentation contains sections which are focused on initial, 'quickstart' deployment scenarios, including the correct configuration of underlying hardware and software, and of Mayastor features such as "Storage Nodes" \(MSNs\) and "Disk Pools" \(MSPs\).  Information describing tuning for the optimisation of performance is also provided.

* [Quickstart Guide](https://mayastor.gitbook.io/introduction/quickstart/configure-mayastor)
* [Performance Tips](https://mayastor.gitbook.io/introduction/quickstart/performance-tips)

MayaData has authored resources which can automatate both these types of configuration tasks.  These include the open source ["automation playground" project](https://github.com/mayadata-io/deployment-automation-playground/tree/main/demo-playground), which uses Terraform and Ansible, is pluggable and extensible, and is able to deploy both FIO based benchmarking and other workloads for initial "first sighting" deployments.  Contributions, extensions and suggestions are welcome.

## What is the basis for its performance claims?  Has it been benchmarked?
Mayastor has been built to deliver as close as possible to local disk performance with a specific design goal of less than 10% overhead versus the performance capabilities of underlying devices including always improving high performance SSDs; while this may sound like common sense, <10% overhead is much lower overhead than traditional storage systems and storage software which often impart 40-50% or greater overhead. For this reason, the Mayastor I/O path is predicated on NVMe, a transport which is both highly CPU efficient and which demonstrates highly linear resource scaling. The data path runs entirely within user space, also contributing efficiency gains as syscalls are avoided, and is both interrupt and lock free; in other words, Mayastor like many high performance networking systems bypasses the kernel in order to deliver higher performance.

MayaData has performed its own benchmarking tests in collaboration with Intel, using latest generation Intel P5800X Optane devices "The World's Fastest Data Centre SSD".  In those tests it was determined that, on average, across a range of read/write ratios and both with and without synchronous mirroring enabled, the overhead imposed by the Mayastor I/O path was well under 10% \(in fact, much closer to 6%\).  

Other benchmarks are available and a goal of the Automation Playground mentioned above is to enable users to easily benchmark Mayastor on their own systems.  More information here:  ["automation playground" project](https://github.com/mayadata-io/deployment-automation-playground/tree/main/demo-playground)

Further information regarding the testing performed may be found [here](https://mayadata.io/assets/pdf/product/intel-and-mayadata-benchmarking-of-openEBS-mayastor.pdf)

##  What is the on-disk format used by Disk Pools?  Is it also open source?
Mayastor makes use of parts of the open source [Storage Performance Development Kit (SPDK)](https://spdk.io/) project, contributed by Intel.  Currently, and for the anticipated General Availability \(GA\) release, Mayastor's Storage Pools use the SPDK's Blobstore structure as their on-disk persistence layer.  Blobstore structures and layout are [documented](https://github.com/spdk/spdk/blob/master/doc/blob.md).

## Can the size / capacity of a Disk Pool be changed?
For the planned GA release, the size of a Disk Pool is fixed at the time of creation and is immutable.  A single Disk Pool may have only one block device as a member.  These constraints may be removed in later versions. Your feedback is welcome.  

## How can I ensure that replicas aren't scheduled onto the same node?  How about onto nodes in the same rack / availability zone?
The replica placement logic of Mayastor's control plane doesn't permit replicas of the same volume to be placed onto the same node, even if it were to be within different Disk Pools.  For example, if a volume with replication factor 3 is to be provisioned, then there must be three healthy Disk Pools available, each with sufficient free capacity and each located on its own Mayastor node.

We are investigating additional ways to leverage topology information including via open source operators now under development by MayaData for future releases.  

## How can I see the node on which the active nexus for a particular volume resides?
The `MayastorVolume` Kubernetes custom resource representing the state and configuration of the volume includes this information within its `Status.Nexus.Node` field.  

## Is there a way to automatically rebalance data across available nodes?  Can data be manually re-distributed?
We are currently working on operators and examining extensions to Mayastor itself to assist in the management of many Mayastor nodes including capacity management across and amongst these nodes.  Please contribute your suggestions and requirements.  

## Can mayastor do async replication to a node in the same cluster?  How about a different cluster?
Mayastor does not peform asynchronous replication at this time. You can use Velero and other solutions for per volume back up. We are also considering the use of snapshots for other use cases. Please contribute your suggestions and requirements.  

## Does Mayastor support RAID?
Currently, Mayastor's Disk Pools do not implement RAID, erasure coding or striping \(in the GA release they will support only the membership of a single disk device\).  However, Mayastor volumes can be provisioned with a replication factor of greater than 1, which will result in synchronously mirrored copies of their data being stored in multiple Disk Pools across multiple Storage Nodes.  If the block device on which a Disk Pool is created is actually a logical unit backed by its own RAID implementation \(e.g. a Fibre Channel attached LUN from an external SAN\) it can still be used within a Mayastor Disk Pool whilst providing protection against physical disk device failures.

## Which environments is Mayastor tested in?
As of March of 2021, Mayastor's maintainers perform end-to-end testing on Mayastor running on Kubernetes version 1.19.  Worker nodes run Ubuntu 20.04.2 LTS, using the docker runtime 20.10.5. Additional environments will be added as Mayastor approaches GA.

## Does Mayastor perform compression and/or deduplication?
Neither compression or deduplication are in scope for Mayastor 1.0.  

## Does Mayastor support snapshots?  Clones?
These features will not be available in the GA release. As mentioned above, we are considering snapshots for a future release.  There appears to be interest in the use of Mayastor with a single replica - i.e. where there is only one copy of the data much like LocalPV - and this data is then also secured somewhat via a snapshot - in particular for resilient workloads such as Cassandra where there may be several copies of the data already made.  As always, please get in touch and share your feedback and suggestions.   

## Which CPU architectures are supported?  What are the minimum hardware requirements?
Mayastor nightly builds and releases are compiled and tested on x86.  Some effort has been made to allow compilation on ARM platforms but this is currently considered experimental and is currently not subject to integration or end-to-end testing by Mayastor's maintainers. There are no plans to support ARM for the GA or 1.0 release. We are aware of efforts to run Mayastor on ARM on Smart NICs for example and conversations about these and other efforts can be found on a Discord server focused on Rust based contributions to Mayastor and related topics here. (https://discord.gg/ctRDfP5P)  

Minimum hardware requirements are discussed in the [quickstart section](https://mayastor.gitbook.io/introduction/quickstart/prerequisites) of this documentation.

Unfortunately Mayastor does not run on Raspbery Pi as the version of the SPDK used by Mayastor requires ARMv8 Crypto extensions which are not currently available for Pi.

## Does Mayastor suffer from TCP congestion when using NVME-TCP?
Mayastor, as any other solution leveraging TCP for network transport, may suffer from network congestion as TCP will try to slow down transfer speeds. It is important to keep an eye on networking and fine-tune the TCP/IP stack as appropriate. This tuning can include \(but is  not limited to\) send and receive buffers, MSS, congestion control algorithms \(e.g. you may try DCTCP\) etc. 

## How can I help?
OpenEBS Mayastor is a part of the OpenEBS community.  As such, we follow the OpenEBS code of conduct which in turn relies on the CNCF code of conduct (https://github.com/cncf/foundation/blob/master/code-of-conduct.md). We welcome all contributions and thank you for your interest.  These contributions can take the form of suggestions, benchmarking results, improvements to this document, improvements to related tooling such as the Automation Playground, improvements to our documentation or just telling your colleagues and collaborators about the promise of Mayastor. Please also consider joining the community on the #OpenEBS channnel on the Kubernetes CNCF Slack. Many Mayastor contributors and community members are also available on the Discord server mentioned above: (https://discord.gg/ctRDfP5P) 
