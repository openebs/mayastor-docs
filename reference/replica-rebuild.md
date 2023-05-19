## Replica Rebuilds

With the previous versions, the control plane ensured replica redundancy by monitoring all volume targets and checking for any volume targets that were in `Degraded` state, indicating that one or more replicas of that volume targets were faulty. When a matching volume targets is found, the faulty replica is removed. Then, a new replica is created and added to the volume targets object. As part of adding the new child data-plane, a full rebuild was initiated from one of the existing `Online` replicas.
However, the drawback to the above approach was that even if a replica was inaccessible for a short period (e.g., due to a node restart), a full rebuild was triggered. This may not have a significant impact on replicas with small sizes, but it is not desirable for large replicas.

The partial rebuild feature, overcomes the above problem and helps in achieving faster rebuild times. When volume target encounters IO error on a child/replica, it marks the child as `Faulted` (removing it from the I/O path) and begins to maintain a write log for all subsequent writes. The Core agent starts a default 10 minute wait for the replica to come back. If the child's replica is online again within timeout, the control-plane requests the volume target to online the child and add it to the IO path along with a partial rebuild process using the aforementioned write log.

{% hint style="info %}
The control plane waits for 10 minutes before initiating the full rebuild process, as the `--faulted-child-wait-period` is set to 10 minutes. To configure this parameter, edit values.yaml.
{% endhint %}