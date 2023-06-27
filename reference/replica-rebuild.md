## Replica Rebuilds

<!--

With the previous versions, the control plane ensured replica redundancy by monitoring all volume targets and checking for any volume targets that were in `Degraded` state, indicating that one or more replicas of that volume targets were faulty. When a matching volume targets is found, the faulty replica is removed. Then, a new replica is created and added to the volume targets object. As part of adding the new child data-plane, a full rebuild was initiated from one of the existing `Online` replicas.
However, the drawback to the above approach was that even if a replica was inaccessible for a short period (e.g., due to a node restart), a full rebuild was triggered. This may not have a significant impact on replicas with small sizes, but it is not desirable for large replicas.

The partial rebuild feature, overcomes the above problem and helps in achieving faster rebuild times. When volume target encounters IO error on a child/replica, it marks the child as `Faulted` (removing it from the I/O path) and begins to maintain a write log for all subsequent writes. The Core agent starts a default 10 minute wait for the replica to come back. If the child's replica is online again within timeout, the control-plane requests the volume target to online the child and add it to the IO path along with a partial rebuild process using the aforementioned write log.
-->

**Full rebuild** and **partial rebuild** are two approaches used in the context of replica redundancy and data recovery in a distributed storage system.

### Full rebuild

A full rebuild refers to the process of completely reconstructing a faulty replica of a volume target in a distributed storage system. In this process, when the control plane detects a replica has faulted, possibly due to node failure, connectivity or other issues (**degraded state**), it removes the faulty replica and initiates the creation of a new replica. The new replica is then added to the volume. Once the new replica is added, the data-plane internally triggers a full rebuild by copying entire data from an already existing healthy replica into the newly added one.

:::note
A drawback of the full rebuild approach is that even if a replica becomes temporarily inaccessible due to reasons like a node restart, a full rebuild is triggered. This means that the entire replica, regardless of its size, is reconstructed from scratch. This can be time-consuming and inefficient, especially for large replicas.
:::

To address the drawback of full rebuild issue, a partial rebuild feature is introduced. 

### Partial rebuild

In the partial rebuild approach, when a volume target encounters an I/O error on a child/replica, it marks that child as **faulted**, effectively removing it from the I/O path. Simultaneously, the volume target starts maintaining a write log for all subsequent writes.

The Core agent, responsible for managing the replica status, waits for a pre-configured period, known as the **faulted child wait period**. During this time, it expects the replica to come back online. If the faulted replica becomes online within the timeout, the control plane instructs the volume target to bring the child online and include it in the I/O path. Additionally, a partial rebuild process is initiated using the previously recorded write log. The partial rebuild focuses only on the changes made since the replica was marked as faulty, thereby reducing the time and resources required compared to a full rebuild.

By implementing a partial rebuild strategy, the storage system can achieve faster rebuild times and avoid unnecessary full rebuilds for replicas that experience temporary interruptions or failures.

{% hint style="info %}
The control plane waits for 10 minutes before initiating the full rebuild process, as the `--faulted-child-wait-period` is set to 10 minutes. To configure this parameter, edit values.yaml.
{% endhint %}

## Replica rebuild history 

The data-plane handles both full and partial replica rebuilds. To view the logs of these rebuilds, you can use either the `kubectl` command or make an API call.


{% tabs %}
{% tab title="Command" %}
```text
kubectl dummy
```
{% endtab %}

{% tab title="CURL" %}
```text
curl -X 'GET' \  'http://{your_localhost}/v0/volumes/{your_volume_UUID}/rebuild' \  -H 'accept: application/json'
```
{% endtab %}
{% endtabs %}

After hitting the curl command with the appropriate localhost and volume UUID (for example: `curl -X 'GET' \  'http://localhost:8081/v0/volumes/bd53de62-e6bb-4b72-a01a-4dcb7aa4d98b/rebuild' \  -H 'accept: application/json'`), you will receive a sample response like the following:

{% tabs %}
{% tab title="Response" %}
```text
{
  "targetUuid": "3c222a50-68df-4e5c-8e5a-33535009fc25",
  "records": [
    {
      "childUri": "nvmf://10.1.0.9:8420/nqn.2019-05.io.openebs:e5631b79-3223-4eeb-8df6-512cb6dc9b54?uuid=e5631b79-3223-4eeb-8df6-512cb6dc9b54",
      "srcUri": "bdev:///4bc36f32-fc83-4786-bc1a-58b6d5cefb81?uuid=4bc36f32-fc83-4786-bc1a-58b6d5cefb81",
      "rebuildJobState": "Completed",
      "blocksTotal": 14302,
      "blocksRecovered": 14302,
      "blocksTransferred": 0,
      "blockSize": 512,
      "isPartial": true,
      "startTime": "2023-06-27T05:09:13.680866230Z",
      "endTime": "2023-06-27T05:09:13.681847385Z"
    }
  ]
}
```
{% endtab %}
{% endtabs %}

:::note
The volume's rebuild history records are stored and maintained as long as the volume target remains intact without any disruptions caused by node failures or recreation.
:::