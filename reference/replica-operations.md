# Replica Operations

## Basics

When a Mayastor volume is provisioned based on a StorageClass which has a replication factor greater than one \(set by its `repl` parameter)\, the control plane will attempt to maintain through a 'Kubernetes-like' reconciliation loop that number of identical copies of the volume's data \("replicas" or "children"\) at any point in time.  When a volume is first provisioned the control plane will attempt to create the required number of replicas, whilst adhering to its internal heuristics for their location within the cluster \(which will be discussed shortly\).  If it succeeds, the volume will become available and will bind with the PVC.  If the control plane cannot identify a sufficient number of eligble Mayastor Pools in which to create required replicas at the time of provisioning, the operation will fail; the Mayastor Volume will not be created and the associated PVC will not be bound.  Kubernetes will periodically re-try the volume creation and if at any time the appropriate number of pools can be selected, the volume provisioning should succeed.

Once a volume is processing I/O, each of its replicas will also receive I/O.  Reads are round-robin distributed across replicas, whilst writes must be written to all.  In a real world environment this is attended by the possibility that I/O to one or more replicas might fail at any time.  Possible reasons include transient loss of network connectivity, node reboots, node or disk failure.  If a volume's nexus \(NVMe controller\) encounters 'too many' failed I/Os for a replica, then that replica's status will be marked `CHILD_FAULTED` and it will no longer receive I/O requests from the nexus.  It will remain a member of the volume, whose departure from the desired state with respect to replica count will reflected with a volume status of `degraded`.  How many I/O failures are considered "too many" in this context is outside the scope of this discussion.

The control plane will attempt to restore the desired state \(replica count\) by creating a new replica, following its replica placement rules.  If it succeeds, the nexus will "rebuild" that new replica - performing a full copy of all data from a healthy replica `CHILD_ONLINE`, i.e. the source.  This process can proceed whilst the volume continues to process application I/Os although it will contend for disk throughput at both the source and destination disks.  It will also 'retire' the old, faulted one which will then no longer be associated to the volume.  Once retired, a replica will become available for garbage collection (deletion from the Mayastor Pool containing it), assuming that the nature of the failure was such that the pool itself is still viable (i.e. the underlying disk device is still accessible).  N.B.: For a replica to be retired from a volume, it must have been possible to create a suitable replacement first.  In the absence of a replacement, the replica remains a member of the nexus, albeit faulted and thus unusable.

If a nexus is restarted, i.e. the Mayastor pod hosting it restarts, with the assistance of the control plane it will 'recompos' itself; all of the previous member replicas will be re-attached to it, if possible, including any that were in the state `CHILD_FAULTED`.  If faulted replicas can be re-connected successfully, then the control plane will attempt to rebuild them directly, rather than seek replacements for them first.  This edge-case therefore does not result in the retirement of the affected replicas; they are simply reused.  If they are successfully re-attached but then continue to encounter I/O failures the rebuild will fail and will not be attempted again.

Once provisioned, the replica count of a volume can be changed by editing its corresponding MayastorVolume \(MSV\) custom resource object.  The value of the `Replica Count` field may be either increased or decreased and the control plane will attempt to reconcile the desired and actual state, following the same replica placement rules as described herein.  If the replica count is reduced, faulted replicas will be selected for removal in preference to healthy ones.

Whenever attempting to reconcile the replica count of an existing volume, should the control plane fail to locate an eligible pool for a required replica, or should the replica creation call fail, the reconciliation 'loop' for that Volume will cease; the desired vs actual replica count will remain diverged.

## Replica Placement Heuristics

Accurate predictions of the behaviour of Mayastor with respect to replica placement and management of replica faults can be made by reference to these 'rules', which are a simplified representation of the actual logic:

* "Rule 1": A volume can only be provisioned if the replica count \(and capacity\) of its StorageClass can be satisfied at the time of creation
* "Rule 2": Every replica of a volume must be placed on a different Mayastor Node \(MSN\)
* "Rule 3": A replica remains a member of a volume even when faulted, until it is retired
* "Rule 4": A replica can only be retired if a replacement can be created FIRST
* "Rule 5": Replicas with the state `CHILD_FAULTED` are always selected for retirement in preference to those with state `CHILD_ONLINE`

N.B.: By application of the 2nd rule, replicas of the same volume cannot exist within different pools on the same Mayastor Node. 

## Example Scenarios

### Scenario One

A cluster has two Mayastor Nodes \(MSN\) deployed, "Node-1" and "Node-2".  Each MSN hosts two MayastorPools \(MSP\) and currently, no MayastorVolumes \(MSV\) have been defined.  MSN Node-1 hosts MSPs "Pool-1-A" and "Pool-1-B", whilst MSN Node-2 hosts "Pool-2-A and "Pool-2-B".  When a user creates a PVC from a StorageClass which defines a replica count of 2, the Mayastor control plane will seek to place one replica on each MSN (it 'follows' Rule 2).  Since in this example it can find a suitable candidate pool with sufficient free capacity on each node, the MSV is provisioned and becomes "healthy" (Rule 1).  Pool-1-A is selected on Node-1, and Pool-2-A selected on Node-2 (all pools being of equal capacity and replica count, in this initial 'clean' state).

Sometime later, the physical disk of Pool-2-A encounters a hardware failure and goes offline.  The MSV is in use at the time, so its nexus \(NVMe controller\) starts to receive I/O errors for the replica hosted in that Pool.  The MSV state becomes `degraded` and the replica associated with Pool-2-A enters the `CHILD_FAULTED` state (as seen in the MSV custom resouce).

Expected Behaviour:  The volume will maintain read/write access for the application via the remaining healthy replica.  The volume's status will remain `degraded` as it is not currently possible for the control plane to create a new replica - Rule 2 applies; the faulted replica is situated on Node-2 and is still a member of the volume's nexus (Rule 3), so a new replica cannot be placed here.  Even though that node has a second, unused pool, the requirement is that ALL replicas of a volume must be placed on different nodes.  There are no other MSNs in the cluster, so the selection of a new replica location fails.

### Scenario Two

A cluster has three MSNs deployed, "Node-1", "Node-2" and "Node-3".  Each MSN hosts one MSP: "Pool-1" on Node-1, "Pool-2" on Node-2 and "Pool-3" on Node-3.  No MSVs have yet been defined; the cluster is 'clean'.  A user creates a PVC from a StorageClass which defines a replica count of 2.  The control plane determines that it is possible to accommodate one replica within the available capacity of each of Pool-1 and Pool-2, and so the MSV is created.  An application is deployed on the cluster which uses the PVC, so the volume receives I/O.

Unfortunately, due to user error the SAN LUN which is used to persist Pool-2 becomes detached from Node-2, causing I/O failures in the replica which it hosts for the volume.  As with scenario one, the volume state becomes `degraded` and the faulted replica's becomes `CHILD_FAULTED`.

Expected Behaviour: Since there is a MSP on Node-3 which has sufficient capacity to host a replacement replica, a new replica can be created (Rule 2: this 'third' incoming replica isn't located on either of the MSNs that the two original ones are).  A new replica is created on Pool-3 and added to the nexus, and the faulted replica in Pool-2 is retired from the nexus.  The new replica is rebuilt and eventually the state of the MSV returns to `healthy`.

### Scenario Three

In the cluster from Scenario three, sometime after the MSV has returned to the `healthy` state, a user edits the MSV's `Replica Count:` field, increasing the value from 2 to 3.  Before doing so they corrected the SAN misconfiguration and ensured that the MSP on Node-2 was `online`.

Expected Behaviour:  The control plane will attempt to reconcile the difference in current (replicas = 2) and desired (replicas = 3) states.  Since MSN Node-2 no longer hosts a replica for the volume (the previously faulted replica was successfully retired and is no longer a member of the volume's nexus), the control plane will select it to host the new replica required (Rule 2 permits this).  The volume state will become initially `degraded` to reflect the difference in actual vs required redundant data copies but a rebuild of the new replica will be performed and eventually the volume state will be `healthy` again.

### Scenario Four

A cluster has three Mayastor Nodes \(MSN\) deployed; "Node-1", "Node-2" and "Node-3".  Each MSN hosts two MayastorPools \(MSP\) and currently, no MayastorVolumes \(MSV\) have been defined.  MSN Node-1 hosts MSPs "Pool-1-A" and "Pool-1-B", whilst MSN Node-2 hosts "Pool-2-A and "Pool-2-B" and Node-3 hosts "Pool-3-A" and "Pool-3-B".  A single MSV exists in the cluster, which has a replica count of 3.  The volume's replicas are all healthy and are located on Pool-1-A, Pool-2-A and Pool-3-A.  An application is using the volume, so all replicas are receiving I/O.

The physical disk of Pool-3-A suffers a permanent failure, causing all failure of all I/O sent to the replica it hosts.

Expected Behaviour:  The volume will enter and remain in the `degraded` state.  The replica in Pool-3-A will be in the state `CHILD_FAULTED`, as observed in the volume's MSV custom resource.  No replica replacement \(nor subsequent rebuild\) will occur, since Rule 3 states that the faulted replica hosted in Pool-3-A on Node-3 remains a part of that volume's nexus and therefore the other pool on Node-3 \(Pool-3-B)\ cannot be selected as a location for its replacement because of Rule 2.

### Scenario Five

Given the post-disk failure situation of Scenario four, the user edits the MSV custom resource for the volume, reducing the value of `Replica Count` from 3 to 2

Expected Behaviour:  The control plane will reconcile the actual \(replicas=3\) vs desired \(replicas=2\) state of the volume by removing a replica from its nexus.  The replica located on Node-3 will be the one selected, since it is in the `CHILD_FAULTED` state whilst the other two replicas are healthy \(`CHILD_ONLINE`)\.  This is Rule 5 in action.  Once the faulted replica has been retired, the volume state will become `healthy` again.

### Scenario Six

In scenario Five, after editing the MSV the user waits for the volume state to become `healthy` again.  The desired and actual replica count are now 2.  The volume's replicas are located in MSPs on both Node-1 and Node-2. The user then edits the MSV custom resource again, increasing  the `Replica Count:` from 2 to 3 again.

Expected Behaviour:  The volume's state will become `degraded`, reflecting the difference in desired vs actual replica count.  The control plane will select a pool on Node-3 as the location for the new replica required.  It is following Rule 2; the replica previously in Pool-3-A, which failed, has already been retired from the membership of the volume's nexus as a result of the user's actions in Scenario Five.  Node-3 is therefore again a suitable candidate and has online pools with sufficient capacity.  If the control plane selects Pool-3-B, which was unaffected by the previous disk failure in Pool-3-A, then once the new replica has been created, a rebuild will take place and eventually, the volume state will return to `healthy` but now with a `Replica Count:` of 3.  However, if Pool-3-A were to be selected (which still has a permanent disk fault and is unable to complete any I/O) then the replica creation will fail; the control plane will not attempt further reconciliation and the volume state will remain `degraded`, with a `Replica Count:` of 2.





