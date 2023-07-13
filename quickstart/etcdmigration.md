---
Title: ETCD Migration Procedure
---

By following the given steps, you can successfully migrate ETCD from one node to another during maintenance activities like node drain etc., ensuring the continuity and integrity of the ETCD data.

:::note
Take a snapshot of the ETCD. Click [here](https://etcd.io/docs/v3.3/op-guide/recovery/) for the detailed documentation.
:::


## Step 1: Draining the ETCD Node

1. Assuming we have a three-node cluster with three ETCD replicas and three pools created in the cluster, verify the ETCD pods and pools with the following commands:

**Command to verify pods**:

{% tabs %}
{% tab title="Command" %}

```text
kubectl get pods -n mayastor -l app=etcd -o wide
```
{% endtab %}
{% tab title="Output" %}

```text
NAME              READY   STATUS    RESTARTS   AGE     IP             NODE       NOMINATED NODE   READINESS GATES
mayastor-etcd-0   1/1     Running   0          4m9s    10.244.1.212   worker-1   <none>           <none>
mayastor-etcd-1   1/1     Running   0          5m16s   10.244.2.219   worker-2   <none>           <none>
mayastor-etcd-2   1/1     Running   0          6m28s   10.244.3.203   worker-0   <none>           <none>
```
{% endtab %}
{% endtabs %}

**Command to verify pools**:

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

2. From etcd-0/1/2 we could see all the values are registered in database, once we migrated etcd to new node, all the key-value pairs should be available across all the pods. Run the following commands from any etcd pod.

**Command to get etcd data**:

{% tabs %}
{% tab title="Command" %}

```text
root@543-master:~# kubectl exec -it mayastor-etcd-0 -n mayastor -- bash
Defaulted container "etcd" out of: etcd, volume-permissions (init)
I have no name!@mayastor-etcd-0:/opt/bitnami/etcd$ ETCDCTL_API=3
I have no name!@mayastor-etcd-0:/opt/bitnami/etcd$ etcdctl get --prefix ""
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

3. In this example, we drain the ETCD node **worker-0** and migrate it to the next available node (in this case, the worker-4 node), use the following command:

**Command to drain the node**:

{% tabs %}
{% tab title="Command" %}

```text
kubectl drain worker-0 --ignore-daemonsets --delete-emptydir-data
```
{% endtab %}
{% tab title="Output" %}

```text
node/worker-0 cordoned
Warning: ignoring DaemonSet-managed Pods: kube-system/kube-flannel-ds-pbm7r, kube-system/kube-proxy-jgjs4, mayastor/mayastor-agent-ha-node-jkd4c, mayastor/mayastor-csi-node-mb89n, mayastor/mayastor-io-engine-q2n28, mayastor/mayastor-promethues-prometheus-node-exporter-v6mfs, mayastor/mayastor-promtail-6vgvm, monitoring/node-exporter-fz247
evicting pod mayastor/mayastor-etcd-2
evicting pod mayastor/mayastor-agent-core-7c594ff676-2ph69
evicting pod mayastor/mayastor-operator-diskpool-c8ddb588-cgr29
pod/mayastor-operator-diskpool-c8ddb588-cgr29 evicted
pod/mayastor-agent-core-7c594ff676-2ph69 evicted
pod/mayastor-etcd-2 evicted
node/worker-0 drained
root@543-master:~#
```
{% endtab %}
{% endtabs %}

## Step 2: Migrating ETCD to the New Node

1. After draining the **worker-0** node, the ETCD pod will be scheduled on the next available node, which is the worker-4 node.
2. The pod may end up in a **CrashLoopBackOff status** with specific errors in the logs.
3. When the pod is scheduled on the new node, it attempts to bootstrap the member again, but since the member is already registered in the cluster, it fails to start the ETCD server with the error message **member already bootstrapped**.
4. To fix this issue, change the cluster's initial state from **new** to **existing** by editing the StatefulSet for ETCD:

**Command to check new etcd pod status**

{% tabs %}
{% tab title="Command" %}

```text
kubectl get pods -n mayastor -l app=etcd -o wide
```
{% endtab %}
{% tab title="Output" %}

```text
NAME              READY   STATUS             RESTARTS      AGE   IP             NODE       NOMINATED NODE   READINESS GATES
mayastor-etcd-0   1/1     Running            0   35m   10.244.1.212   worker-1   <none>           <none>
mayastor-etcd-1   1/1     Running            0   36m   10.244.2.219   worker-2   <none>           <none>
mayastor-etcd-2   0/1     CrashLoopBackOff   5 (44s ago)   10m   10.244.0.121   worker-4     <none>           <none>

```
{% endtab %}
{% endtabs %}

**Command to edit the StatefulSet**:

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
```
{% endtab %}
{% endtabs %}

## Step 3: Validating ETCD Key-Value Pairs

Run the appropriate command from the migrated ETCD pod to validate the key-value pairs and ensure they are the same as in the existing ETCD. This step is crucial to avoid any data loss during the migration process.


{% tabs %}
{% tab title="Command" %}

```text
root@543-master:~# kubectl exec -it mayastor-etcd-0 -n mayastor -- bash
Defaulted container "etcd" out of: etcd, volume-permissions (init)
I have no name!@mayastor-etcd-2:/opt/bitnami/etcd$ ETCDCTL_API=3
I have no name!@mayastor-etcd-2:/opt/bitnami/etcd$ etcdctl get --prefix ""
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