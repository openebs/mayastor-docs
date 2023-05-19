## Replica Rebuilds

With the previous versions, the control plane ensured replica redundancy by monitoring all nexuses and checking for any nexus that were in `Degraded` state, indicating that one or more replicas of that nexus were faulty. When a matching nexus is found, the faulty replica is removed. Then, a new replica is created and added to the nexus object. As part of adding the new child data-plane, a full rebuild was initiated from one of the existing `Online` replicas.
However, the drawback to the above approach was that even if a replica was inaccessible for a short period (e.g., due to a node restart), a full rebuild was triggered. This may not have a significant impact on replicas with small sizes, but it is not desirable for large replicas.

The partial rebuild feature, overcomes the above problem and helps in achieving faster rebuild times. When nexus encounters IO error on a child/replica, It marks child as `Faulted`, it begins to maintain a write log for all subsequent writes and removes child from IO path. Core agent starts a default 10 minute wait for the replica to come back. If the child's replica is brought online again within timeout, the control-plane marks child as `Online`, after which nexus adds child to the IO path and initiates a partial rebuild process for the replica using the write log.

{% hint style="info %}
The control plane waits for 10 minutes before initiating the full rebuild process, as the `--faulted-child-wait-period` is set to 10 minutes. To configure this parameter, edit values.yaml.
{% endhint %}