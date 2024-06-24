# Configure Mayastor

{% hint style="danger" %}
This website/page will be End-of-life (EOL) after 31 August 2024. We recommend you to visit [OpenEBS Documentation](https://openebs.io/docs/user-guides/replicated-storage-user-guide/replicated-pv-mayastor/rs-installation) for the latest Mayastor documentation (v2.6 and above).
 
Mayastor is now also referred to as OpenEBS Replicated PV Mayastor.
{% endhint %}

## Create Mayastor Pool\(s\)

### What is a Mayastor Pool?

When a Mayastor node allocates storage capacity for a replica of a Persistent Volume it does so from a Mayastor Pool. Each Mayastor node may create and manage zero, one, or more such pools. The ownership of a pool by a node is exclusive. A pool can manage only one block device, which constitutes the entire data persistence layer for that pool and thus defines its maximum storage capacity.

{% hint style="info" %}
Note that in the current version of Mayastor pools support only thick provisioning.
{% endhint %}

A pool is defined declaratively, through the creation of a corresponding `MayastorPool` (MSP) custom resource on the cluster. The MSP must be created in the same namespace as Mayastor has been deployed.  User configurable parameters of this resource type include a unique name for the pool, the node name on which it will be hosted and a reference to a disk device which is accessible from that node. The pool definition requires the reference to its member block device to adhere to a discrete range of schemas, each associated with a specific access mechanism/transport/ or device type.

#### Permissible Schemes for the MSP field `disks`

| Type | Format | Example |
| :--- | :--- | :--- |
| Attached Disk Device \(AIO\) | Device File | /dev/sdx |
| Async. Disk I/O \(AIO\) | Device File | aio:///dev/sdx |
| io\_uring | Device File | uring:///dev/sdx |
| RAM drive | Custom | malloc:///malloc0?size\_mb=1024 |

Once a Mayastor node has created a pool it is assumed that it henceforth has exclusive use of the associated block device; it should not be partitioned, formatted, or shared with another application or process. Any pre-existing data on the device will be destroyed.

{% hint style="warning" %}
A RAM drive isn't suitable for use in production as it uses volatile memory for backing the data. The memory for this disk emulation is allocated from the hugepages pool. Make sure to allocate sufficient additional hugepages resource on any storage nodes which will provide this type of storage.
{% endhint %}

### Configure Pools for Use with this Quickstart

To continue with this quick start exercise, a minimum of one pool is necessary, created and hosted on one of the Mayastor nodes in the cluster. However, the number of pools available limits the extent to which the synchronous n-way mirroring feature \("replication"\) of Persistent Volumes can be configured for testing and evaluation; the number of pools configured should be no fewer than the desired maximum replication factor of the PVs to be created. Also, while placing data replicas ensure that appropriate redundancy is provided. Mayastor's control plane will avoid locating more than one replica of a volume on the same Mayastor node. Therefore, for example, the minimum viable configuration for a Mayastor deployment which is intended to test 3-way mirrored PVs must have three Mayastor Nodes, each having one Mayastor Pool, with each of those pools having one unique block device allocated to it.

Using one or more the following examples as templates, create the required type and number of pools.

{% tabs %}
{% tab title="Example MSP definition" %}
```text
cat <<EOF | kubectl create -f -
apiVersion: "openebs.io/v1alpha1"
kind: MayastorPool
metadata:
  name: pool-on-node-1
  namespace: mayastor
spec:
  node: workernode-1-hostname
  disks: ["/dev/sdx"]
EOF
```
{% endtab %}

{% tab title="YAML" %}
```text
apiVersion: "openebs.io/v1alpha1"
kind: MayastorPool
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

The status of Mayastor Pools may be determined by reference to their cluster CRs. Available, healthy pools should report their State as `online`. Verify that the expected number of pools have been created and that they are online.

{% tabs %}
{% tab title="Command" %}
```text
kubectl -n mayastor get msp
```
{% endtab %}

{% tab title="Example Output" %}
```text
NAME             NODE                       STATE    AGE
pool-on-node-0   aks-agentpool-12194210-0   online   127m
pool-on-node-1   aks-agentpool-12194210-1   online   27s
pool-on-node-2   aks-agentpool-12194210-2   online   4s
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Pool configuration and state information can also be obtained by using the [Mayastor kubectl plugin](https://mayastor.gitbook.io/introduction/reference/kubectl-plugin)
{% endhint %}

## Create Mayastor StorageClass\(s\)

Mayastor dynamically provisions Persistent Volumes \(PV\) based on StorageClass definitions created by the user. Parameters of the definition are used to set the characteristics and behaviour of its associated PVs. For a detailed description of these parameters see [storage class parameter description](https://mayastor.gitbook.io/introduction/reference/storage-class-parameters). Most importantly StorageClass definition is used to control the level of data protection afforded to it \(that is, the number of synchronous data replicas which are maintained, for purposes of redundancy\). It is possible to create any number of StorageClass definitions, spanning all permitted parameter permutations.

We illustrate this quickstart guide with two examples of possible use cases; one which offers no data redundancy \(i.e. a single data replica\), and another having three data replicas. You can modify these as required to match your own desired test cases, within the limitations of the cluster under test.

{% tabs %}
{% tab title="Command \(1 replica example\)" %}
```text
cat <<EOF | kubectl create -f -
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: mayastor-1
parameters:
  repl: '1'
  protocol: 'nvmf'
  ioTimeout: '60'
  local: 'true'
provisioner: io.openebs.csi-mayastor
volumeBindingMode: WaitForFirstConsumer
EOF
```
{% endtab %}

{% tab title="Command \(3 replicas example\)" %}
```text
cat <<EOF | kubectl create -f -
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: mayastor-3
parameters:
  repl: '3'
  protocol: 'nvmf'
  ioTimeout: '60'
  local: 'true'
provisioner: io.openebs.csi-mayastor
volumeBindingMode: WaitForFirstConsumer
EOF
```
{% endtab %}
{% endtabs %}

**Action: Create the StorageClass\(es\) appropriate to your intended testing scenario\(s\).**

