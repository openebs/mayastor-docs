---
Title: Scaling up ETCD members
---

By default, Mayastor allows the creation of three ETCD members. If you wish to increase the number of etcd replicas, you will encounter an error. However, you can make the necessary configuration changes discussed in this guide to make it work.




## Overview of StatefulSets


StatefulSets are Kubernetes resources designed for managing stateful applications. They provide stable network identities and persistent storage for pods. StatefulSets ensure ordered deployment and scaling, support persistent volume claims, and manage the state of applications. They are commonly used for databases, messaging systems, and distributed file systems. Here's how StatefulSets function:
* For a StatefulSet with N replicas, when pods are deployed, they are created sequentially in order from {0..N-1}.
* When pods are deleted, they are terminated in reverse order from {N-1..0}.
* Before a scaling operation is applied to a pod, all of its predecessors must be running and ready.
* Before a pod is terminated, all of its successors must be completely shut down.
* Currently, we have a three-node cluster with 3 ETCD replicas and three pools created in the cluster.

{% tabs %}
{% tab title="Command" %}
```text 
kubectl get dsp -n mayastor
```
{% endtab %}
{% tab title="Output" %}
```text 
NAME     NODE       STATE    POOL_STATUS   CAPACITY       USED          AVAILABLE
pool-0   worker-0   Online   Online        374710730752   22561161216   352149569536
pool-1   worker-1   Online   Online        374710730752   21487419392   353223311360
pool-2   worker-2   Online   Online        374710730752   21793603584   352917127168
```
{% endtab %}
{% endtabs %}

* From etcd-0/1/2, we can see that all the values are registered in the database. Once we scale up ETCD with "n" replicas, all the key-value pairs should be available across all the pods.

To scale up or scale down the ETCD members, the following steps can be performed:


1. Add a new ETCD member
2. Add a peer URL
3. Create a PV (Persistent Volume)
4. Validate key-value pairs

----------

## Step 1: Adding a New ETCD Member (Scaling Up ETCD Replica)


To increase the number of replicas to 4, use the following `kubectl scale` command:

{% tabs %}
{% tab title="Command" %}
```text 
kubectl scale sts mayastor-etcd -n mayastor --replicas=4
```
{% endtab %}
{% tab title="Output" %}
```text 
statefulset.apps/mayastor-etcd scaled
```
{% endtab %}
{% endtabs %}


> The new pod will be created on available nodes but will be in a **pending state** as there is no PV/PVC created to bind the volumes.

{% tabs %}
{% tab title="Command" %}
```text 
kubectl get pods -n mayastor -l app=etcd
```
{% endtab %}
{% tab title="Output" %}
```text 
NAME              READY   STATUS    RESTARTS   AGE
mayastor-etcd-0   1/1     Running   0          28d
mayastor-etcd-1   1/1     Running   0          28d
mayastor-etcd-2   1/1     Running   0          28d
mayastor-etcd-3   0/1     Pending   0          2m34s
```
{% endtab %}
{% endtabs %}

## Step 2: Add a New Peer URL

Before creating a PV, we need to add the new peer URL (mayastor-etcd-3=http://mayastor-etcd-3.mayastor-etcd-headless.mayastor.svc.cluster.local:2380) and change the cluster's initial state from "new" to "existing" so that the new member will be added to the existing cluster when the pod comes up after creating the PV. Since the new pod is still in a pending state, the changes will not be applied to the other pods as they will be restarted in reverse order from {N-1..0}. It is expected that all of its predecessors must be running and ready.


{% tabs %}
{% tab title="Command" %}
```text 
kubectl edit sts mayastor-etcd -n mayastor 
```
{% endtab %}
{% tab title="Output" %}
```text 
        - name: ETCD_INITIAL_CLUSTER_STATE
          value: existing
        - name: ETCD_INITIAL_CLUSTER
          value: mayastor-etcd-0=http://mayastor-etcd-0.mayastor-etcd-headless.mayastor.svc.cluster.local:2380,mayastor-etcd-1=http://mayastor-etcd-1.mayastor-etcd-headless.mayastor.svc.cluster.local:2380,mayastor-etcd-2=http://mayastor-etcd-2.mayastor-etcd-headless.mayastor.svc.cluster.local:2380,mayastor-etcd-3=http://mayastor-etcd-3.mayastor-etcd-headless.mayastor.svc.cluster.local:2380
```
{% endtab %}
{% endtabs %}


## Step 3: Create a Persistent Volume

Create a PV with the following YAML. Change the pod name/claim name based on the pod's unique identity.


{% tabs %}
{% tab title="YAML" %}
```text 

apiVersion: v1
kind: PersistentVolume
metadata:
  annotations:
    meta.helm.sh/release-name: mayastor
    meta.helm.sh/release-namespace: mayastor
    pv.kubernetes.io/bound-by-controller: "yes"
  finalizers:
  - kubernetes.io/pv-protection
  labels:
    app.kubernetes.io/managed-by: Helm
    statefulset.kubernetes.io/pod-name: mayastor-etcd-3
  name: etcd-volume-3
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 2Gi
  claimRef:
    apiVersion: v1
    kind: PersistentVolumeClaim
    name: data-mayastor-etcd-3
    namespace: mayastor
  hostPath:
    path: /var/local/mayastor/etcd/pod-3
    type: ""
  persistentVolumeReclaimPolicy: Delete
  storageClassName: manual
  volumeMode: Filesystem
```
{% endtab %}
{% tab title="Output" %}
```text 
kubectl apply -f pv-etcd.yaml -n mayastor
persistentvolume/etcd-volume-3 created
```
{% endtab %}
{% endtabs %}




## Step 4: Validate Key-Value Pairs


Run the following command from the new ETCD pod and ensure that the values are the same as those in etcd-0/1/2. Otherwise, it indicates a data loss issue.


{% tabs %}
{% tab title="Command" %}
```text 
root@543-master:~# kubectl exec -it mayastor-etcd-3 -n mayastor -- bash
Defaulted container "etcd" out of: etcd, volume-permissions (init)
I have no name!@mayastor-etcd-3:/opt/bitnami/etcd$ ETCDCTL_API=3
I have no name!@mayastor-etcd-3:/opt/bitnami/etcd$ etcdctl get --prefix ""
```
{% endtab %}
{% tab title="Output" %}
```text 
/openebs.io/mayastor/apis/v0/clusters/ce05eb25-50cc-400a-a57f-37e6a5ed9bef/namespaces/mayastor/CoreRegistryConfig/db98f8bb-4afc-45d0-85b9-24c99cc443f2
{"id":"db98f8bb-4afc-45d0-85b9-24c99cc443f2","registration":"Automatic"}
/openebs.io/mayastor/apis/v0/clusters/ce05eb25-50cc-400a-a57f-37e6a5ed9bef/namespaces/mayastor/NexusSpec/d69c29f6-b2cf-4834-9e82-733189358b8d
{"uuid":"d69c29f6-b2cf-4834-9e82-733189358b8d","name":"94e50361-1533-44df-96fb-4852aeadef06","node":"worker-2","children":[{"Replica":{"uuid":"4b3c3899-d832-4159-b795-7a83370e699b","share_uri":"bdev:///4b3c3899-d832-4159-b795-7a83370e699b?uuid=4b3c3899-d832-4159-b795-7a83370e699b"}},{"Replica":{"uuid":"9f0eea06-2bf8-4e57-b30f-515b3c93275b","share_uri":"nvmf://10.20.30.57:8420/nqn.2019-05.io.openebs:9f0eea06-2bf8-4e57-b30f-515b3c93275b?uuid=9f0eea06-2bf8-4e57-b30f-515b3c93275b"}},{"Replica":{"uuid":"cf85027a-08b2-4278-a886-5fe8bfe1b6ed","share_uri":"nvmf://10.20.30.56:8420/nqn.2019-05.io.openebs:cf85027a-08b2-4278-a886-5fe8bfe1b6ed?uuid=cf85027a-08b2-4278-a886-5fe8bfe1b6ed"}}],"size":21474836480,"spec_status":{"Created":"Online"},"share":"nvmf","managed":true,"owner":"94e50361-1533-44df-96fb-4852aeadef06","operation":null,"nvmf_config":{"controllerIdRange":{"start":13,"end":14},"reservationKey":11421818261557578637,"reservationType":"ExclusiveAccess","preemptPolicy":"Holder"},"status_info":{"shutdown_failed":false},"allowed_hosts":["nqn.2019-05.io.openebs:node-name:worker-1"]}
/openebs.io/mayastor/apis/v0/clusters/ce05eb25-50cc-400a-a57f-37e6a5ed9bef/namespaces/mayastor/NexusSpec/e44c195f-8ab6-474b-8035-07ec3e7664c6
{"uuid":"e44c195f-8ab6-474b-8035-07ec3e7664c6","name":"d9e86e31-e8d5-4374-8651-6d44ade3dd05","node":"worker-0","children":[{"Replica":{"uuid":"6aa8e5e2-b4c9-4762-8f58-7d7b16d63266","share_uri":"bdev:///6aa8e5e2-b4c9-4762-8f58-7d7b16d63266?uuid=6aa8e5e2-b4c9-4762-8f58-7d7b16d63266"}},{"Replica":{"uuid":"580fd333-1275-4560-9c6c-0824564162e4","share_uri":"nvmf://10.20.30.64:8420/nqn.2019-05.io.openebs:580fd333-1275-4560-9c6c-0824564162e4?uuid=580fd333-1275-4560-9c6c-0824564162e4"}},{"Replica":{"uuid":"517db6a5-5df2-4144-a842-30c3c9839ead","share_uri":"nvmf://10.20.30.57:8420/nqn.2019-05.io.openebs:517db6a5-5df2-4144-a842-30c3c9839ead?uuid=517db6a5-5df2-4144-a842-30c3c9839ead"}}],"size":21474836480,"spec_status":{"Created":"Online"},"share":"nvmf","managed":true,"owner":"d9e86e31-e8d5-4374-8651-6d44ade3dd05","operation":null,"nvmf_config":{"controllerIdRange":{"start":1,"end":2},"reservationKey":9238298921862063302,"reservationType":"ExclusiveAccess","preemptPolicy":"Holder"},"status_info":{"shutdown_failed":false},"allowed_hosts":["nqn.2019-05.io.openebs:node-name:worker-0"]}
/openebs.io/mayastor/apis/v0/clusters/ce05eb25-50cc-400a-a57f-37e6a5ed9bef/namespaces/mayastor/NodeSpec/worker-0
{"id":"worker-0","endpoint":"10.20.30.56:10124","labels":{},"node_nqn":"nqn.2019-05.io.openebs:node-name:worker-0","draining_volumes":[]}
/openebs.io/mayastor/apis/v0/clusters/ce05eb25-50cc-400a-a57f-37e6a5ed9bef/namespaces/mayastor/NodeSpec/worker-1
{"id":"worker-1","endpoint":"10.20.30.57:10124","labels":{},"node_nqn":"nqn.2019-05.io.openebs:node-name:worker-1","draining_volumes":[]}
/openebs.io/mayastor/apis/v0/clusters/ce05eb25-50cc-400a-a57f-37e6a5ed9bef/namespaces/mayastor/NodeSpec/worker-2
{"id":"worker-2","endpoint":"10.20.30.64:10124","labels":{},"node_nqn":"nqn.2019-05.io.openebs:node-name:worker-2","draining_volumes":[]}
/openebs.io/mayastor/apis/v0/clusters/ce05eb25-50cc-400a-a57f-37e6a5ed9bef/namespaces/mayastor/PoolSpec/pool-0
{"node":"worker-0","id":"pool-0","disks":["/dev/nvme0n1"],"status":{"Created":"Online"},"labels":{"openebs.io/created-by":"operator-diskpool"},"operation":null}
/openebs.io/mayastor/apis/v0/clusters/ce05eb25-50cc-400a-a57f-37e6a5ed9bef/namespaces/mayastor/PoolSpec/pool-1
{"node":"worker-1","id":"pool-1","disks":["/dev/nvme0n1"],"status":{"Created":"Online"},"labels":{"openebs.io/created-by":"operator-diskpool"},"operation":null}
/openebs.io/mayastor/apis/v0/clusters/ce05eb25-50cc-400a-a57f-37e6a5ed9bef/namespaces/mayastor/PoolSpec/pool-2
{"node":"worker-2","id":"pool-2","disks":["/dev/nvme0n1"],"status":{"Created":"Online"},"labels":{"openebs.io/created-by":"operator-diskpool"},"operation":null}
/openebs.io/mayastor/apis/v0/clusters/ce05eb25-50cc-400a-a57f-37e6a5ed9bef/namespaces/mayastor/ReplicaSpec/4b3c3899-d832-4159-b795-7a83370e699b
{"name":"4b3c3899-d832-4159-b795-7a83370e699b","uuid":"4b3c3899-d832-4159-b795-7a83370e699b","size":21474836480,"pool":"pool-2","share":"none","thin":true,"status":{"Created":"online"},"managed":true,"owners":{"volume":"94e50361-1533-44df-96fb-4852aeadef06","disown_all":false},"operation":null,"allowed_hosts":["nqn.2019-05.io.openebs:node-name:worker-1"]}
/openebs.io/mayastor/apis/v0/clusters/ce05eb25-50cc-400a-a57f-37e6a5ed9bef/namespaces/mayastor/ReplicaSpec/517db6a5-5df2-4144-a842-30c3c9839ead
{"name":"517db6a5-5df2-4144-a842-30c3c9839ead","uuid":"517db6a5-5df2-4144-a842-30c3c9839ead","size":21474836480,"pool":"pool-1","share":"nvmf","thin":true,"status":{"Created":"online"},"managed":true,"owners":{"volume":"d9e86e31-e8d5-4374-8651-6d44ade3dd05","disown_all":false},"operation":null,"allowed_hosts":["nqn.2019-05.io.openebs:node-name:worker-0"]}
/openebs.io/mayastor/apis/v0/clusters/ce05eb25-50cc-400a-a57f-37e6a5ed9bef/namespaces/mayastor/ReplicaSpec/580fd333-1275-4560-9c6c-0824564162e4
{"name":"580fd333-1275-4560-9c6c-0824564162e4","uuid":"580fd333-1275-4560-9c6c-0824564162e4","size":21474836480,"pool":"pool-2","share":"nvmf","thin":true,"status":{"Created":"online"},"managed":true,"owners":{"volume":"d9e86e31-e8d5-4374-8651-6d44ade3dd05","disown_all":false},"operation":null,"allowed_hosts":["nqn.2019-05.io.openebs:node-name:worker-0"]}
/openebs.io/mayastor/apis/v0/clusters/ce05eb25-50cc-400a-a57f-37e6a5ed9bef/namespaces/mayastor/ReplicaSpec/6aa8e5e2-b4c9-4762-8f58-7d7b16d63266
{"name":"6aa8e5e2-b4c9-4762-8f58-7d7b16d63266","uuid":"6aa8e5e2-b4c9-4762-8f58-7d7b16d63266","size":21474836480,"pool":"pool-0","share":"none","thin":true,"status":{"Created":"online"},"managed":true,"owners":{"volume":"d9e86e31-e8d5-4374-8651-6d44ade3dd05","disown_all":false},"operation":null,"allowed_hosts":[]}
/openebs.io/mayastor/apis/v0/clusters/ce05eb25-50cc-400a-a57f-37e6a5ed9bef/namespaces/mayastor/ReplicaSpec/9f0eea06-2bf8-4e57-b30f-515b3c93275b
{"name":"9f0eea06-2bf8-4e57-b30f-515b3c93275b","uuid":"9f0eea06-2bf8-4e57-b30f-515b3c93275b","size":21474836480,"pool":"pool-1","share":"nvmf","thin":true,"status":{"Created":"online"},"managed":true,"owners":{"volume":"94e50361-1533-44df-96fb-4852aeadef06","disown_all":false},"operation":null,"allowed_hosts":["nqn.2019-05.io.openebs:node-name:worker-2"]}
/openebs.io/mayastor/apis/v0/clusters/ce05eb25-50cc-400a-a57f-37e6a5ed9bef/namespaces/mayastor/ReplicaSpec/cf85027a-08b2-4278-a886-5fe8bfe1b6ed
{"name":"cf85027a-08b2-4278-a886-5fe8bfe1b6ed","uuid":"cf85027a-08b2-4278-a886-5fe8bfe1b6ed","size":21474836480,"pool":"pool-0","share":"nvmf","thin":true,"status":{"Created":"online"},"managed":true,"owners":{"volume":"94e50361-1533-44df-96fb-4852aeadef06","disown_all":false},"operation":null,"allowed_hosts":["nqn.2019-05.io.openebs:node-name:worker-2"]}
/openebs.io/mayastor/apis/v0/clusters/ce05eb25-50cc-400a-a57f-37e6a5ed9bef/namespaces/mayastor/ReplicaSpec/e9c9b95b-75de-430a-82e4-9861c22beef8
{"name":"e9c9b95b-75de-430a-82e4-9861c22beef8","uuid":"e9c9b95b-75de-430a-82e4-9861c22beef8","size":10737418240,"pool":"pool-2","share":"none","thin":true,"status":{"Created":"online"},"managed":true,"owners":{"volume":"9cb5b575-966a-47d2-aac9-8b90df96f65e","disown_all":false},"operation":null,"allowed_hosts":[]}
/openebs.io/mayastor/apis/v0/clusters/ce05eb25-50cc-400a-a57f-37e6a5ed9bef/namespaces/mayastor/StoreLeaseLock/CoreAgent/c1588aeb1969907

/openebs.io/mayastor/apis/v0/clusters/ce05eb25-50cc-400a-a57f-37e6a5ed9bef/namespaces/mayastor/StoreLeaseOwner/CoreAgent
{"kind":"CoreAgent","lease_id":"c1588aeb1969907","instance_name":"mayastor-agent-core-7c594ff676-2ph69"}
/openebs.io/mayastor/apis/v0/clusters/ce05eb25-50cc-400a-a57f-37e6a5ed9bef/namespaces/mayastor/VolumeSpec/94e50361-1533-44df-96fb-4852aeadef06
{"uuid":"94e50361-1533-44df-96fb-4852aeadef06","size":21474836480,"labels":null,"num_replicas":3,"status":{"Created":"Online"},"policy":{"self_heal":true},"topology":{"node":null,"pool":{"Labelled":{"exclusion":{},"inclusion":{"openebs.io/created-by":"operator-diskpool"}}}},"last_nexus_id":null,"operation":null,"thin":true,"target":{"node":"worker-2","nexus":"d69c29f6-b2cf-4834-9e82-733189358b8d","protocol":"nvmf","active":true,"config":{"controllerIdRange":{"start":13,"end":14},"reservationKey":11421818261557578637,"reservationType":"ExclusiveAccess","preemptPolicy":"Holder"},"frontend":{"host_acl":[{"node_name":"worker-1","node_nqn":"nqn.2019-05.io.openebs:node-name:worker-1"}]}},"publish_context":{"ioTimeout":"30"},"affinity_group":null}
/openebs.io/mayastor/apis/v0/clusters/ce05eb25-50cc-400a-a57f-37e6a5ed9bef/namespaces/mayastor/VolumeSpec/9cb5b575-966a-47d2-aac9-8b90df96f65e
{"uuid":"9cb5b575-966a-47d2-aac9-8b90df96f65e","size":10737418240,"labels":null,"num_replicas":1,"status":{"Created":"Online"},"policy":{"self_heal":true},"topology":{"node":null,"pool":{"Labelled":{"exclusion":{},"inclusion":{"openebs.io/created-by":"operator-diskpool"}}}},"last_nexus_id":null,"operation":null,"thin":true,"target":{"node":"worker-2","nexus":"c29a2e54-592f-498e-9142-cbea47176630","protocol":"nvmf","active":false,"config":{"controllerIdRange":{"start":1,"end":2},"reservationKey":10467152691037955632,"reservationType":"ExclusiveAccess","preemptPolicy":"Holder"},"frontend":{"host_acl":[{"node_name":"worker-2","node_nqn":"nqn.2019-05.io.openebs:node-name:worker-2"}]}},"publish_context":null,"affinity_group":{"id":"minio-operator/data0-tenant-pool-0"}}
/openebs.io/mayastor/apis/v0/clusters/ce05eb25-50cc-400a-a57f-37e6a5ed9bef/namespaces/mayastor/VolumeSpec/d9e86e31-e8d5-4374-8651-6d44ade3dd05
{"uuid":"d9e86e31-e8d5-4374-8651-6d44ade3dd05","size":21474836480,"labels":null,"num_replicas":3,"status":{"Created":"Online"},"policy":{"self_heal":true},"topology":{"node":null,"pool":{"Labelled":{"exclusion":{},"inclusion":{"openebs.io/created-by":"operator-diskpool"}}}},"last_nexus_id":null,"operation":null,"thin":true,"target":{"node":"worker-0","nexus":"e44c195f-8ab6-474b-8035-07ec3e7664c6","protocol":"nvmf","active":true,"config":{"controllerIdRange":{"start":1,"end":2},"reservationKey":9238298921862063302,"reservationType":"ExclusiveAccess","preemptPolicy":"Holder"},"frontend":{"host_acl":[{"node_name":"worker-0","node_nqn":"nqn.2019-05.io.openebs:node-name:worker-0"}]}},"publish_context":{"ioTimeout":"30"},"affinity_group":null}
/openebs.io/mayastor/apis/v0/clusters/ce05eb25-50cc-400a-a57f-37e6a5ed9bef/namespaces/mayastor/volume/94e50361-1533-44df-96fb-4852aeadef06/nexus/d69c29f6-b2cf-4834-9e82-733189358b8d/info
{"children":[{"healthy":true,"uuid":"4b3c3899-d832-4159-b795-7a83370e699b"},{"healthy":true,"uuid":"9f0eea06-2bf8-4e57-b30f-515b3c93275b"},{"healthy":true,"uuid":"cf85027a-08b2-4278-a886-5fe8bfe1b6ed"}],"clean_shutdown":false}
/openebs.io/mayastor/apis/v0/clusters/ce05eb25-50cc-400a-a57f-37e6a5ed9bef/namespaces/mayastor/volume/9cb5b575-966a-47d2-aac9-8b90df96f65e/nexus/c29a2e54-592f-498e-9142-cbea47176630/info
{"children":[{"healthy":true,"uuid":"e9c9b95b-75de-430a-82e4-9861c22beef8"}],"clean_shutdown":true}
/openebs.io/mayastor/apis/v0/clusters/ce05eb25-50cc-400a-a57f-37e6a5ed9bef/namespaces/mayastor/volume/d9e86e31-e8d5-4374-8651-6d44ade3dd05/nexus/e44c195f-8ab6-474b-8035-07ec3e7664c6/info
{"children":[{"healthy":true,"uuid":"6aa8e5e2-b4c9-4762-8f58-7d7b16d63266"},{"healthy":true,"uuid":"580fd333-1275-4560-9c6c-0824564162e4"},{"healthy":true,"uuid":"517db6a5-5df2-4144-a842-30c3c9839ead"}],"clean_shutdown":false}
```
{% endtab %}
{% endtabs %}