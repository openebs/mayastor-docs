## Manual Upgrade

Upgrading Mayastor from a version prior to 2.0.0 needs manual intervention. Follow the steps given below.
To get started, take an etcd snapshot. The detailed steps for taking a snapshot can be found in the etcd [documentation](https://etcd.io/docs/v3.3/op-guide/recovery/).


{% hint style="warning" %}
The following changes are breaking changes when upgrading from a Mayastor version prior to 2.0.0. 
**ETCD:**
  - Control Plane: The prefixes for control plane have changed from `/namespace/$NAMESPACE/control-plane/` to `/openebs.io/mayastor/apis/v0/clusters/$KUBE_SYSTEM_UID/namespaces/$NAMESPACE/`
  - Data Plane: The Data Plane nexus information containing a list of healthy children has been moved from `$nexus_uuid` to `/openebs.io/mayastor/apis/v0/clusters/$KUBE_SYSTEM_UID/namespaces/$NAMESPACE/volume/$volume_uuid/nexus/$nexus_uuid/info`

**RPC:**
  - Control Plane: The RCP for the control plane has been changed from NATS to gRPC.
  - Data Plane: The registration heartbeat has been changed from NATS to gRPC. 

**Pool CRDs:**
  - The pool CRDs have been renamed `DiskPools` (previously, MayastorPools).
  {% endhint %}



1. In order to start the upgrade process, all the previously deployed components have to be deleted. 

 - To delete the control-plane components, execute: 
    {% tab title="Commands" %}
    ```text
    kubectl delete deploy core-agents -n mayastor
    kubectl delete deploy csi-controller -n mayastor
    kubectl delete deploy msp-operator -n mayastor
    kubectl delete deploy rest -n mayastor
    kubectl delete ds mayastor-csi -n mayastor
    ```
    {% endtab %}
 
- Next, delete the associated RBAC operator. To do so, execute:
    {% tab title="Commands" %}
    ```text
    kubectl delete -f operator-rbac.yaml
    ```
    {% endtab %}


2. Once all the above components have been successfully removed, pull the latest helm chart from [Mayastor-extension repo](https://github.com/openebs/mayastor-extensions) and save it to a file, say `helm_templates.yaml`. To do so, execute:

   {% tab title="Command" %}
    ```text
    helm template mayastor . -n mayastor --set etcd.persistence.storageClass="manual" --set loki-stack.loki.persistence.storageClassName="manual" > helm_templates.yaml
    ```
    {% endtab %}

- Next, update the `helm_template.yaml` file, add the following helm label to all the resources that are being created.

    {% tab title="Helm label" %}
    ```text
    metadata: 
      annotations: 
        meta.helm.sh/release-name: $RELEASE_NAME 
        meta.helm.sh/release-namespace: $RELEASE_NAMESPACE 
      labels: 
        app.kubernetes.io/managed-by: Helm
    ```
    {% endtab %}

- Remove the `etcd` and `io-engine` spec from the `helm_templates.yaml`. These components will be upgraded separately. 

3.  Install the new control-plane components using the `helm-templates.yaml` file. 

    {% tab title="Command" %}
    ```text
    kubectl apply -f helm_templates.yaml -n mayastor
    ```
    {% endtab %}


    {% hint style="info" %}
    In the above method of installation, HA is disabled by default. The steps to enable HA are described later in this document.
    {% endhint %} 

    - Verify the status of the pods. Upon successful deployment, all the pods will be in a running state.
    {% tab title="Command" %}
    ```text
    kubectl get pods -n mayastor
    ```
    {% endtab %}

    - Verify the etcd prefix and compat mode.
    {% tab title="Command" %}
    ```text
    kubectl exec -it mayastor-etcd-0 -n mayastor -- bash
    Defaulted container "etcd" out of: etcd, volume-permissions (init)
    I have no name!@mayastor-etcd-0:/opt/bitnami/etcd$ export ETCDCTL_API=3
    I have no name!@mayastor-etcd-0:/opt/bitnami/etcd$ etcdctl get --prefix ""
    ```
    {% endtab %}    

    - Verify if the DiskPools are online.
    {% tab title="Command" %}
    ```text
    kubectl get dsp -n mayastor
    ```
    {% endtab %} 

    - Next, verify the status of the volumes.
    {% tab title="Command" %}
    ```text
    kubectl get volumes
    ```
    {% endtab %}       

4. After upgrading control-plane components, the data-plane pods have to be upgraded. To do so, deploy the `io-engine` DaemonSet from Mayastor's new version. 

_Using the command given below, the data-plane pods (now io-engine pods) will be upgraded to Mayastor v2.0._

    {% tab title="Command" %}
    ```text
    kubectl apply -f mayastor_io_v2.0.yaml -n mayastor
    ```
    {% endtab %}

-    Delete the previously deployed data-plane pods (`mayastor-xxxxx`). The data-plane pods need to be manually deleted as their update-strategy is set to `delete`. Upon successful deletion, the new `io-engine` pods will be up and running.   

- NATS has been replaced by gRPC for Mayastor versions 2.0 or later. Hence, the NATS components (StatefulSets and services) have to be removed from the cluster.

    {% tab title="Command" %}
    ```text
    kubectl delete sts nats -n mayastor
    kubectl delete svc nats -n mayastor
    ```
    {% endtab %}

5. After `control-plane` and `io-engine`, the etcd has to be upgraded. Before starting the etcd upgrade, label the etcd PV and PVCs with helm. This will be needed to make them helm compatible.


    {% tab title="Command to patch etcd PVC" %}
    ```text
    kubectl patch pvc <data-mayastor-etcd-x> --patch-file labels.yaml n mayastor 
    ```
    {% endtab %}


    {% tab title="Command to patch etcd PV" %}
    ```text
    kubectl patch pv <etcd-volume-x> --patch-file labels.yaml -n mayastor
    ```
    {% endtab %}

  - Next, update the `values.yaml` file and set the state variable to `existing`. 

    To deploy the new etcd YAML, execute:
    {% tab title="Command" %}
    ```text
    kubectl apply -f mayastor_2.0_etcd.yaml -n mayastor
    ```
    {% endtab %}

    Now, verify the etcd space and compat mode, execute:

    {% tab title="Command" %}
    ```text
    kubectl exec -it mayastor-etcd-0 -n mayastor -- bash
    Defaulted container "etcd" out of: etcd, volume-permissions (init)
    I have no name!@mayastor-etcd-0:/opt/bitnami/etcd$ export ETCDCTL_API=3
    I have no name!@mayastor-etcd-0:/opt/bitnami/etcd$ etcdctl get --prefix ""
    ```
    {% endtab %}


6. Once all the components have been upgraded, the HA module can now be enabled via the helm upgrade command.  

    {% tab title="Command to check the helm list" %}
    ```text
    helm upgrade --install mayastor . -n mayastor --set etcd.persistence.storageClass="manual" --set loki-stack.loki.persistence.storageClassName="manual" --set agents.ha.enabled="true"
    ```
    {% endtab %} 

7. This concludes the process of manual upgrade. Run the below commands to verify the upgrade,

    {% tab title="Command to check the helm list" %}
    ```text
    helm list -n mayastor
    ```
    {% endtab %} 


    {% tab title="Command to check the status of pods" %}
    ```text
    kubectl get pods -n mayastor
    ```
    {% endtab %}      


    {% tab title="Command to check the application and volume" %}
    ```text
    kubectl mayastor get volumes
    kubectl get pods
    ```
    {% endtab %}   


