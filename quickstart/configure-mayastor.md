# Configure Mayastor

## Create Mayastor Pool\(s\)

### What is a Mayastor Pool \(MSP\)?

When a Mayastor Node \(MSN\) allocates storage capacity for a Persistent Volume \(PV\) it does so from a construct named a Mayastor Pool \(MSP\).  Each MSN may create and manage zero, one, or more such pools.  The ownership of a pool by a MSN is exclusive.  In the current version of Mayastor, a pool may have only a single block device member, which constitutes the entire data persistence layer for that pool and thus determines its maximum capacity.

A pool is defined declaratively, through the creation of a corresponding `MayastorPool`custom resource \(CR\) on the cluster.  User configurable parameters of this CR type include a unique name for the pool, the host name of the MSN on which it is hosted and a reference to a disk device which is accessible from that node \(for inclusion within the pool\).  The pool definition allows the reference to its member block device to adhere to one of a number of possible schemes, each associated with a specific access mechanism/transport/device type and differentiated by corresponding performance and/or attachment locality.

#### Permissible Schemes for the MSP CRD field `disks`

| Type | Format | Example |
| :--- | :--- | :--- |
| Attached Disk Device | Device File | /dev/sdx |
| NVMe-oF | NQN | nvmf://nqn.2014-08.com.vendor:nvme:nvm-subsystem-sn-d78432 |
| iSCSI | IQN | iscsi://iqn.2000-08.com.datacore.com:cloudvm41-2 |
| Async. Disk I/O \(AIO\) | Device File  | aio:///dev/sdx |
| io\_uring | Device File  | io\_uring:///dev/sdx |
| RAM drive | Custom | malloc:///malloc0?size\_mb=1024 |

Once a Mayastor node has created a pool it is assumed that it henceforth has exclusive use of the associated block device; it should not be partitioned, formatted, or shared with another application or process.  Any existing data on the device will be destroyed.

### Configure Pool\(s\) for Use with this Quickstart

To continue with this quick start exercise, a minimum of one pool is necessary, created and hosted by one of the MSNs in the cluster.  However the number of pools available limits the extent to which the synchronous n-way mirroring feature \("replication"\) of Persistent Volumes can configured for testing and evaluation; the number of pools configured should be no lower than the desired maximum replication factor of the PVs to be created.  Note also that when placing data replicas, to provide appropriate redundancy, Mayastor's control plane will avoid locating more than one replica of a PV on the same MSN.  Therefore, for example, the minimum viable configuration for a Mayastor deployment which is intended  to test 3-way mirrored PVs must have three Mayastor Nodes, each having one Mayastor Pool,  with each of those pools having one unique block device allocated to it. 

Using one or more the following examples as templates, create the required type and number of pools.

{% tabs %}
{% tab title="Example MSP CRD \(device file\)" %}
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

{% tab title="Example \(NVMe-oF Fabric device\)" %}
```
cat <<EOF | kubectl create -f -
apiVersion: "openebs.io/v1alpha1"
kind: MayastorPool
metadata:
  name: nvme-pool-on-node-2
  namespace: mayastor
spec:
  node: workernode-2-hostname
  disks: ["nvme://nqn.2014-08.com.vendor:nvme:nvm-subsystem-sn-d78432"]
EOF
```
{% endtab %}

{% tab title="Example \(iSCSI device\)" %}
```
cat <<EOF | kubectl create -f -
apiVersion: "openebs.io/v1alpha1"
kind: MayastorPool
metadata:
  name: iscsi-pool-on-node-3
  namespace: mayastor
spec:
  node: workernode-3-hostname
  disks: ["iscsi://iqn.yyyy-mm.naming-authority:unique_name"]
EOF
```
{% endtab %}

{% tab title="YAML" %}
```
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
When following the examples in order to create your own Mayastor Pool\(s\), remember to replace the values for the fields "name", "node" and "disks" as appropriate to your cluster's intended configuration.  Note that whilst the "disks" parameter accepts an array of scheme values, the current version of Mayastor supports only one disk device per pool.
{% endhint %}

### Verify Pool Creation and Status

The status of Mayastor Pools may be determined by reference to their cluster CRs.  Available, healthy pools should report their State as `online`.  Verify that the expected number of pools have been created and that they are online.

{% tabs %}
{% tab title="Command" %}
```text
kubectl -n mayastor get msp
```
{% endtab %}

{% tab title="Example Output" %}
```
NAME             NODE                       STATE    AGE
pool-on-node-0   aks-agentpool-12194210-0   online   127m
pool-on-node-1   aks-agentpool-12194210-1   online   27s
pool-on-node-2   aks-agentpool-12194210-2   online   4s
```
{% endtab %}
{% endtabs %}

## Create Mayastor StorageClass\(s\)

Mayastor dynamically provisions Persistent Volumes \(PV\) based on custom StorageClass definitions defined by the user.  Parameters of the StorageClass resource definition are used to set the characteristics and behaviour of its associated PVs.  In the current version of Mayastor, StorageClass definitions are used to control both which transport protocol is used to mount the PV to the worker node hosting the consuming application pod \(iSCSI, or NVMe-oF TCP\) and the level of data protection afforded to it \(that is, the number of synchronous data replicas which are maintained, for purposes of redundancy\).  It is possible to create any number of custom StorageClass definitions to span this range of permutations.

We illustrate this quickstart guide with two examples of possible use cases; one which uses iSCSI and offers no data protection \(i.e. a single data replica\), and another using NVMe-oF TCP transport and having three data replicas.  You may modify these as required to match your own desired test cases, within the limitations of the cluster under test.

{% tabs %}
{% tab title="Command \(iSCSI Example\)" %}
```text
cat <<EOF | kubectl create -f -
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: mayastor-iscsi
parameters:
  # Set the number of data replicas ("replication factor")
  repl: '1'
  # Set the export transport protocol
  protocol: 'iscsi'
provisioner: io.openebs.csi-mayastor
EOF
```
{% endtab %}

{% tab title="Command \(NVME-oF Example\)" %}
```
cat <<EOF | kubectl create -f -
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: mayastor-nvmf
parameters:
  # Set the number of data replicas ("replication factor")
  repl: '3'
  # Set the export transport protocol
  protocol: 'nvmf'
provisioner: io.openebs.csi-mayastor
EOF
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Note:  Permissible values for the field "protocol" are either "iscsi", or "nvmf"
{% endhint %}

**Action: Create the StorageClass\(es\) appropriate to your intended testing scenario\(s\).**

