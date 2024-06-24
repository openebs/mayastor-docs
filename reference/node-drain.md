## Mayastor Node Drain

{% hint style="danger" %}
This website/page will be End-of-life (EOL) after 31 August 2024. We recommend you to visit [OpenEBS Documentation](https://openebs.io/docs/user-guides/replicated-storage-user-guide/replicated-pv-mayastor/rs-installation) for the latest Mayastor documentation (v2.6 and above).
 
Mayastor is now also referred to as OpenEBS Replicated PV Mayastor.
{% endhint %}

The node drain functionality marks the node as unschedulable and then gracefully moves all the volume targets off the drained node. 
This feature is in line with the [node drain functionality of Kubernetes](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/).


To start the drain operation, execute:
{% tab title="Command" %}
```text
kubectl-mayastor drain node <node_name> <label>
```
{% endtab %}

To get the list of nodes on which the drain operation has been performed, execute:
{% tab title="Command" %}
```text
kubectl-mayastor get drain nodes
```
{% endtab %}

To halt the drain operation or to make the node schedulable again, execute:

{% tab title="Command" %}
```text
kubectl-mayastor uncordon node <node_name> <label>
```
{% endtab %}