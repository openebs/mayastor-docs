---
Title: OpenEBS Volume Snapshot
Sidebar: Volume snapshot
---


**Volume snapshots** are copies of persistent volumes at a specific point in time. They can be used to restore a volume to a previous state or create a new volume. Mayastor provides support for industry standard copy-on-write (COW) snapshots, which is a popular methodology for taking snapshots by keeping track of only those blocks that have changed.
Mayastor incremental snapshot capability enhances data migration and portability in Kubernetes clusters across different cloud providers or data centers. Using standard kubectl commands, you can seamlessly perform operations on snapshots and clones in a fully Kubernetes-native manner.

Use cases for volume snapshots include:

- Efficient replication for backups.
- Utilization of clones for troubleshooting.
- Development against a read-only copy of data.


Volume snapshots allow the creation of read-only incremental copies of volumes, enabling you to maintain a history of your data. These volume snapshots possess the following characteristics:
- **Consistency**: The data stored within a snapshot remains consistent across all replicas of the volume, whether local or remote.
- **Immutability**: Once a snapshot is successfully created, the data contained within it cannot be modified.
- **Thin Provisioning**: Thin provisioning will be enabled explicitly when creating the volume. It allows volumes to be allocated with minimal initial capacity and dynamically grow as data is written.


Currently, Mayastor supports the following operations related to volume snapshots:

1. Creating a snapshot for a PVC
2. Listing available snapshots for a PVC
3. Deleting a snapshot for a PVC


------------------


## Prerequisites


1. Deploy and configure Mayastor by following the steps given [here](https://mayastor.gitbook.io/introduction/quickstart/deploy-mayastor) and create disk pools. 
2. Create a Mayastor StorageClass with thin provisioning enabled. Add the `thin: true` parameter in the StorageClass YAML to enable thin provisioning.

:::note
Currently Mayastor only supports snapshots for volumes with a single replica. Snapshot support for volumes with more than one replica will be available in the future versions.
:::

{% tabs %}

{% tab title="Command (single replica)" %}

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
provisioner: io.openebs.csi-mayastor
EOF
```

{% endtab %}

{% tabs %}

{% tab title="YAML (single replica)" %}

```text
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: mayastor-1
parameters:
  ioTimeout: "30"
  protocol: nvmf
  repl: "1"
provisioner: io.openebs.csi-mayastor

```

{% endtab %}

{% endtabs %}

3. Create a PVC using [these](https://mayastor.gitbook.io/introduction/quickstart/deploy-a-test-application#define-the-pvc) steps and check if the status of the PVC is **Bound**.

{% tabs %}
{% tab title="Command" %}
```text
kubectl get pvc
```
{% endtab %}

{% tab title="Example Output" %}
```text
NAME                STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS     AGE
ms-volume-claim     Bound    pvc-fe1a5a16-ef70-4775-9eac-2f9c67b3cd5b   1Gi        RWO            mayastor-1       15s
```
{% endtab %}
{% endtabs %}

> Copy the PVC name, for example, `ms-volume-claim`.


4. (Optional) Create an application by following [these](https://mayastor.gitbook.io/introduction/quickstart/deploy-a-test-application#deploy-the-fio-test-pod) steps. 
 
 ----------
 
## Create a Snapshot

You can create a snapshot **with or without an application using PVCs**. Follow the steps below to create a volume snapshot:


### Step 1: Create a Kubernetes VolumeSnapshotClass object


{% tabs %}

{% tab title="Command" %}

```text=
cat <<EOF | kubectl create -f -
kind: VolumeSnapshotClass
apiVersion: snapshot.storage.k8s.io/v1
metadata:
  name: csi-mayastor-snapshotclass
  annotations:
    snapshot.storage.kubernetes.io/is-default-class: "true"
driver: io.openebs.csi-mayastor
deletionPolicy: Delete
EOF
```

{% endtab %}

{% tab title="YAML" %}

```text
kind: VolumeSnapshotClass
apiVersion: snapshot.storage.k8s.io/v1
metadata:
  name: csi-mayastor-snapshotclass
  annotations:
    snapshot.storage.kubernetes.io/is-default-class: "true"
driver: io.openebs.csi-mayastor
deletionPolicy: Delete

```

{% endtab %}

{% endtabs %}



| Parameters | Type | Description|
| -------- | ---- | -------- |
| **Name**| String | Custom name of the snapshot class|
| **Driver** | String | CSI provisioner of the storage provider being requested to create a snapshot (io.openebs.csi-mayastor)|

**Apply VolumeSnapshotClass details**


{% tabs %}
{% tab title="Command" %}
```text
kubectl apply -f class.yaml
```
{% endtab %}

{% tab title="Example Output" %}
```text
volumesnapshotclass.snapshot.storage.k8s.io/csi-mayastor-snapshotclass created
```
{% endtab %}
{% endtabs %}




### Step 2: Create the snapshot



{% tabs %}


{% tab title="Command" %}

```text=
cat <<EOF | kubectl create -f -
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: mayastor-pvc-snap-1
spec:
  volumeSnapshotClassName: csi-mayastor-snapshotclass
  source:
    persistentVolumeClaimName: ms-volume-claim   
EOF
```
{% endtab %}

{% tab title="YAML" %}

```text
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: mayastor-pvc-snap-1
spec:
  volumeSnapshotClassName: csi-mayastor-snapshotclass
  source:
    persistentVolumeClaimName: ms-volume-claim   

```

{% endtab %}

{% endtabs %}


| Parameters | Type | Description|
| -------- | ---- | -------- |
| **Name** | String | Name of the snapshot | 
| **VolumeSnapshotClassName** | String | Name of the created snapshot class | 
| **PersistentVolumeClaimName** | String | Name of the PVC. Example- `ms-volume-claim` | 


**Apply the snapshot**


{% tabs %}
{% tab title="Command" %}
```text
kubectl apply -f snapshot.yaml
```
{% endtab %}

{% tab title="Example Output" %}
```text
volumesnapshot.snapshot.storage.k8s.io/mayastor-pvc-snap-1 created
```
{% endtab %}
{% endtabs %}

---------

## List Snapshots 

To retrieve the details of the created snapshots, use the following command:


{% tabs %}
{% tab title="Command" %}
```text
kubectl get volumesnapshot 
```
{% endtab %}

{% tab title="Example Output" %}
```text
NAME                READYTOUSE   SOURCEPVC         SOURCESNAPSHOTCONTENT                RESTORESIZE                            SNAPSHOTCLASS    SNAPSHOTCONTENTCREATIONTIME    AGE
mayastor-pvc-snap-1   false     ms-volume-claim     1Gi csi-mayastor-snapshotclass  snapcontent-88aa857e-3c07-4ad9-b4f8-2a9d    2f209a4d              4m5s                   4m5s
```
{% endtab %}
{% endtabs %}

    
----

## Delete a Snapshot


To delete a snapshot, use the following command:


{% tabs %}
{% tab title="Command" %}
```text
kubectl delete volumesnapshot mayastor-pvc-snap-1
    
```
{% endtab %}

{% tab title="Example Output" %}
```text
volumesnapshot.snapshot.storage.k8s.io "mayastor-pvc-snap-1" deleted
```
{% endtab %}
{% endtabs %}



