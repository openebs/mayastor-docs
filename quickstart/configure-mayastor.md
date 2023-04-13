# Configure Mayastor

## Create DiskPool\(s\)


### What is a DiskPool?

When a node allocates storage capacity for a replica of a persistent volume (PV) it does so from a DiskPool. Each node may create and manage zero, one, or more such pools. The ownership of a pool by a node is exclusive. A pool can manage only one block device, which constitutes the entire data persistence layer for that pool and thus defines its maximum storage capacity.

{% hint style="info" %}
This version of Mayastor only supports thick provisioning.
{% endhint %}

A pool is defined declaratively, through the creation of a corresponding `DiskPool` custom resource on the cluster. The DiskPool must be created in the same namespace where Mayastor has been deployed. User configurable parameters of this resource type include a unique name for the pool, the node name on which it will be hosted and a reference to a disk device which is accessible from that node. The pool definition requires the reference to its member block device to adhere to a discrete range of schemas, each associated with a specific access mechanism/transport/ or device type.

{% hint style="info" %}
Mayastor versions before 2.0.1 had an upper limit on the number of retry attempts in the case of failure in `create events` in the DSP operator. With this release, the upper limit has been removed, which ensures that the DiskPool operator indefinitely reconciles with the CR.
{% endhint %}

#### Permissible Schemes for `spec.disks` under DiskPool CR

{% hint style="info" %}
It is highly recommended to specify the disk using a unique device link that remains unaltered across node reboots. One such device link is its `UUID`.
To get the UUID of a disk, execute:
`sudo blkid`

Usage of the device name (for example, /dev/sdx) is not advised, as it may change if the node reboots, which might cause data corruption.
{% endhint %}

| Type | Format | Example |
| :--- | :--- | :--- |
| Disk(non PCI) with disk-by-guid reference <i><b>(Best Practice)</b></i> | Device File | aio:////dev/disk/by-uuid/<uuid> OR uring:////dev/disk/by-uuid/<uuid> |
| Asynchronous Disk\(AIO\) | Device File | /dev/sdx |
| Asynchronous Disk I/O \(AIO\) | Device File | aio:///dev/sdx |
| io\_uring | Device File | uring:///dev/sdx |


Once a node has created a pool it is assumed that it henceforth has exclusive use of the associated block device; it should not be partitioned, formatted, or shared with another application or process. Any pre-existing data on the device will be destroyed.

{% hint style="warning" %}
A RAM drive isn't suitable for use in production as it uses volatile memory for backing the data. The memory for this disk emulation is allocated from the hugepages pool. Make sure to allocate sufficient additional hugepages resource on any storage nodes which will provide this type of storage.
{% endhint %}

### Configure Pools for Use with this Quickstart

To get started, it is necessary to create and host at least one pool on one of the nodes in the cluster. The number of pools available limits the extent to which the synchronous N-way mirroring (replication) of PVs can be configured; the number of pools configured should be equal to or greater than the desired maximum replication factor of the PVs to be created. Also, while placing data replicas ensure that appropriate redundancy is provided. Mayastor's control plane will avoid placing more than one replica of a volume on the same node. For example, the minimum viable configuration for a Mayastor deployment which is intended to implement 3-way mirrored PVs must have three nodes, each having one DiskPool, with each of those pools having one unique block device allocated to it.

Using one or more the following examples as templates, create the required type and number of pools.

{% tabs %}
{% tab title="Example DiskPool definition" %}
```text
cat <<EOF | kubectl create -f -
apiVersion: "openebs.io/v1alpha1"
kind: DiskPool
metadata:
  name: pool-on-node-1
  namespace: mayastor
spec:
  node: workernode-1-hostname
  disks: ["/dev/disk/by-uuid/<uuid>"]
EOF
```
{% endtab %}

{% tab title="YAML" %}
```text
apiVersion: "openebs.io/v1alpha1"
kind: DiskPool
metadata:
  name: INSERT_POOL_NAME_HERE
  namespace: mayastor
spec:
  node: INSERT_WORKERNODE_HOSTNAME_HERE
  disks: ["INSERT_DEVICE_URI_HERE"]
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
When using the examples given as guides to creating your own pools, remember to replace the values for the fields "metadata.name", "spec.node" and "spec.disks" as appropriate to your cluster's intended configuration. Note that whilst the "disks" parameter accepts an array of values, the current version of Mayastor supports only one disk device per pool.
{% endhint %}

### Verify Pool Creation and Status

The status of DiskPools may be determined by reference to their cluster CRs. Available, healthy pools should report their State as `online`. Verify that the expected number of pools have been created and that they are online.

{% tabs %}
{% tab title="Command" %}
```text
kubectl get dsp -n mayastor
```
{% endtab %}

{% tab title="Example Output" %}
```text
NAME             NODE          STATE    POOL_STATUS   CAPACITY      USED   AVAILABLE
pool-on-node-1   node-1-14944  Online   Online        10724835328   0      10724835328
pool-on-node-2   node-2-14944  Online   Online        10724835328   0      10724835328
pool-on-node-3   node-3-14944  Online   Online        10724835328   0      10724835328
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}

Mayastor-2.0.1 adds two new fields to the DiskPool operator YAML:
1. **status.cr_state**: The `cr_state`, which can either be _creating, created or terminating_, will be used by the operator to reconcile with the CR. 
The `cr_state` is set to `Terminating` when a CR delete event is received.
2. **status.pool_status**: The `pool_status` represents the status of the respective control plane pool resource. 
{% endhint %}

Pool configuration and state information can also be obtained by using the [Mayastor kubectl plugin](https://mayastor.gitbook.io/introduction/reference/kubectl-plugin)


## Create Mayastor StorageClass\(s\)

Mayastor dynamically provisions PersistentVolumes \(PVs\) based on StorageClass definitions created by the user. Parameters of the definition are used to set the characteristics and behaviour of its associated PVs. For a detailed description of these parameters see [storage class parameter description](https://mayastor.gitbook.io/introduction/reference/storage-class-parameters). Most importantly StorageClass definition is used to control the level of data protection afforded to it \(that is, the number of synchronous data replicas which are maintained, for purposes of redundancy\). It is possible to create any number of StorageClass definitions, spanning all permitted parameter permutations.

We illustrate this quickstart guide with two examples of possible use cases; one which offers no data redundancy \(i.e. a single data replica\), and another having three data replicas. 
{% hint style="info" %}
Both the example YAMLs given below have [thin provisioning](https://mayastor.gitbook.io/introduction/quickstart/configure-mayastor/storage-class-parameters#thin) enabled. You can modify these as required to match your own desired test cases, within the limitations of the cluster under test.
{% endhint %}

{% tabs %}
{% tab title="Command \(1 replica example\)" %}
```text
cat <<EOF | kubectl create -f -
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: mayastor-1
parameters:
  ioTimeout: "30"
  protocol: nvmf
  repl: "1"
  thin: true
provisioner: io.openebs.csi-mayastor
EOF
```
{% endtab %}

{% tab title="Command \(3 replicas example\)" %}
```text
cat <<EOF | kubectl create -f -
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: mayastor-3
parameters:
  ioTimeout: "30"
  protocol: nvmf
  repl: "3"
  thin: true
provisioner: io.openebs.csi-mayastor
EOF
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
The default installation of Mayastor includes the creation of a StorageClass with the name `mayastor-single-replica`. The replication factor is set to 1. Users may either use this StorageClass or create their own.
{% endhint %}
