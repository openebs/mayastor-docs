## High Availability 

Mayastor 2.0 brings in the High Availability (HA) feature. In the event of a failure, this feature will guarantee I/O continuity. 
The HA feature consists of two components: the HA node agent and the cluster agent. The HA node agent looks for io-path failures from applications to their corresponding targets. If any such broken path is encountered, the HA node agent informs the cluster agent. The cluster-agent then creates a new target on a different (live) node. Once the target is created, the `csi-node` establishes a new path between the application and its corresponding target. The HA feature restores the broken path within seconds, ensuring negligible downtime. 

### How do I disable this feature? 

{% hint style="info" %}
**Note:**We strongly recommend keeping this feature enabled.
{% endhint %}

The HA feature is enabled by default; to disable it, edit the `charts.yaml` file and set  `agents.ha.enabled` as false. 
