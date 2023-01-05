# Monitoring

## Pool metrics exporter

The Mayastor pool metrics exporter runs as a sidecar container within every io-engine pod and exposes pool usage metrics in Prometheus format. These metrics are exposed on port 9502 using an HTTP endpoint /metrics and are refreshed every five minutes.

### Supported metrics

| Name | Type | Unit | Description |
| :--- | :--- | :--- | :--- |
| disk_pool_total_size_bytes | Gauge | Integer | Total size of the pool |
| disk_pool_used_size_bytes | Gauge | Integer | Used size of the pool |
| disk_pool_status | Gauge | Integer | Status of the pool (0, 1, 2, 3) = {"Unknown", "Online", "Degraded", "Faulted"} |


{% tab title="Example metrics" %}
```text
# HELP disk_pool_status mayastor name status
# TYPE disk_pool_status gauge
disk_pool_status{node="worker-0",name="mayastor-disk-pool"} 1
# HELP disk_pool_total_size_bytes mayastor name total size in bytes
# TYPE disk_pool_total_size_bytes gauge
disk_pool_total_size_bytes{node="worker-0",name="mayastor-disk-pool"} 5.360320512e+09
# HELP disk_pool_used_size_bytes mayastor name used size in bytes
# TYPE disk_pool_used_size_bytes gauge
disk_pool_used_size_bytes{node="worker-0",name="mayastor-disk-pool"} 2.147483648e+09
```
{% endtab %}


### Integrating exporter with Prometheus monitoring stack

1. To install, add the Prometheus-stack helm chart and update the repo.

{% tab title="Command" %}
```text
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```
{% endtab %}

Then, install the Prometheus monitoring stack and set prometheus.prometheusSpec.serviceMonitorSelectorNilUsesHelmValues to false. This enables Prometheus to discover custom ServiceMonitor for Mayastor.

{% tab title="Command" %}
```text
helm install mayastor prometheus-community/kube-prometheus-stack -n mayastor --set prometheus.prometheusSpec.serviceMonitorSelectorNilUsesHelmValues=false
```
{% endtab %}

2. Next, install the ServiceMonitor resource to select services and specify their underlying endpoint objects.

{% tab title="ServiceMonitor YAML" %}
```text
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: mayastor-monitoring
  labels:
    app: mayastor
spec:
  selector:
    matchLabels:
      app: mayastor
  endpoints:
  - port: metrics
```
{% endtab %}

{% hint style="info" %}
Upon successful integration of the exporter with the Prometheus stack, the metrics will be available on the port 9090 and all the endpoints will be able to make HTTP calls.
{% endhint %}


## CSI metrics exporter

| Name | Type | Unit | Description |
| :--- | :--- | :--- | :--- |
| kubelet_volume_stats_available_bytes | Gauge | Integer | Size of the available/usable volume (in bytes) | 
| kubelet_volume_stats_capacity_bytes | Gauge | Integer | The total size of the volume (in bytes) |
| kubelet_volume_stats_used_bytes | Gauge | Integer | Used size of the volume (in bytes) |
| kubelet_volume_stats_inodes | Gauge |	Integer | The total number of inodes |
| kubelet_volume_stats_inodes_free | Gauge | Integer | The total number of usable inodes. |
| kubelet_volume_stats_inodes_used | Gauge | Integer | The total number of inodes that have been utilized to store metadata. |


[Learn more](https://kubernetes.io/docs/concepts/storage/volume-health-monitoring/)