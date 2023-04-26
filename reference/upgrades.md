## Upgrading Mayastor

Upgrade to the latest Mayastor version is supported only from 2.1.0 or later. To upgrade from a previous version, visit [Manual steps to upgrade Mayastor](manual-steps-to-upgrade-mayastor). 

### Supported upgrade paths

- From 2.0.x to 2.1.0

{% hint style="info" %}
- The process of upgrade utilises the [Mayastor Kubectl Plugin](https://mayastor.gitbook.io/introduction/advanced-operations/kubectl-plugin).
- The upgrade process is generally non-disruptive for volumes with a replication factor greater than 1 and all replicas being healthy, prior to starting the upgrade.
{% endhint %}

To upgrade Mayastor deployment on the Kubernetes cluster,

{% tab title="Command" %}
```text
kubectl mayastor upgrade
```
{% endtab %}

Mayastor 2.1.0 has the capability to perform selective-component upgrades. To view the flags that can be passed in order to achieve a non-disruptive selective upgrade, execute the following command:

{% tabs %}
{% tab title="Command" %}
```text
kubectl mayastor upgrade -h
```
{% endtab %}

{% tab title="Expected Output" %}
```text
`Upgrade` the deployment

Usage: kubectl-mayastor upgrade [OPTIONS]

Options:
  -d, --dry-run
          Display all the validations output but will not execute upgrade
  -r, --rest <REST>
          The rest endpoint to connect to
  -D, --skip-data-plane-restart
          If set then upgrade will skip the io-engine pods restart
  -k, --kube-config-path <KUBE_CONFIG_PATH>
          Path to kubeconfig file
  -S, --skip-single-replica-volume-validation
          If set then it will continue with upgrade without validating singla replica volume
  -R, --skip-replica-rebuild
          If set then upgrade will skip the repilca rebuild in progress validation
  -C, --skip-cordoned-node-validation
          If set then upgrade will skip the cordoned node validation
  -o, --output <OUTPUT>
          The Output, viz yaml, json [default: none]
  -j, --jaeger <JAEGER>
          Trace rest requests to the Jaeger endpoint agent
  -n, --namespace <NAMESPACE>
          Kubernetes namespace of mayastor service, defaults to mayastor [default: mayastor]
  -h, --help
          Print help
```
{% endtab %}
{% endtabs %}


To view the status of upgrade, execute:

{% tabs %}
{% tab title="Command" %}
```text
kubectl mayastor get upgrade-status
```
{% endtab %}

{% tab title="Expected Output" %}
```text
Upgrade From: 2.0.0
Upgrade To: 2.1.0
Upgrade Status: Successfully upgraded Mayastor
```
{% endtab %}
{% endtabs %}

To view the logs of upgrade job, execute:

{% tab title="Command" %}
```text
kubectl get logs job/job-name -n <namespace>
```
{% endtab %}


{% hint style="info" %}
1. The time taken to upgrade is directly proportional to the number of Mayastor nodes and Mayastor volumes.
2. To upgrade to a particular Mayastor version, ensure you are using the same version of kubectl plugin.
3. The above process of upgrade creates one ClusterRole, one ClusterRoleBinding and one ServiceAccount in the namespace where Mayastor is installed.
{% endhint %}

