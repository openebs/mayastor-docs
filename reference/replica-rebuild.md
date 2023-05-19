## Replica Rebuilds

With the previous versions, the control plane ensured replica redundancy by monitoring all nexuses and checking for any nexus that were in `Degraded` state, indicating that one or more replicas of that nexus were faulty. When a matching nexus is found, the faulty replica is removed. Then, a new replica is created and added to the nexus object. As part of adding the new child data-plane, a full rebuild was initiated from one of the existing `Online` replicas.
However, the drawback to the above approach was that even if a replica was inaccessible for a short period (e.g., due to a node restart), a full rebuild was triggered. This may not have a significant impact on replicas with small sizes, but it is not desirable for large replicas.

The partial rebuild feature, overcomes the above problem and helps in achieving faster rebuild times. When a nexus identifies a replica as `Faulted`, it begins to maintain a write log for all subsequent writes. This write log is crucial for the data-plane to utilise if the `Faulted` replica becomes operational again. Once the same replica is brought online, the control-plane marks it as `Online`, initiating a partial rebuild process for the replica using the write log.

{% hint style="info %}
The control plane waits for 10 minutes before initiating the full rebuild process, as the `--faulted-child-wait-period` is set to 10 minutes. To configure this parameter, edit values.yaml.
{% endhint %}