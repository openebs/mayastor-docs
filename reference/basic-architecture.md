# Basic Architecture

The objective of this section is to provide the user and evaluator of Mayastor with a topological view of the gross anatomy of a Mayastor deployment. A description will be made of the expected pod inventory of a correctly deployed cluster, the roles and functions of the constituent pods and related Kubernetes resource types, and of the high level interactions between them and the orchestration thereof.

More detailed guides to Mayastor's components, their design and internal structure, and instructions for building Mayastor from source, are maintained within the [project's GitHub repository](https://github.com/openebs/Mayastor).

## Dramatis Personae

| Name | Resource Type | Function | Frequency in Cluster |
| :--- | :--- | :--- | :--- |
| **Control Plane** |
| agent-core | Deployment | Principle control plane actor | Single |
| csi-controller | Deployment | Hosts Mayastor's CSI controller implementation and CSI provisioner side car| Single |
| api-rest | Pod | Hosts the public API REST server | Single |
| *api-rest*  | Service | Exposes the REST API server via NodePort |
| operator-diskpool | Deployment | Hosts DiskPool operator | Single |
| csi-node| DaemonSet | Hosts CSI Driver node plugin containers | All worker nodes |
| etcd | StatefulSet | Hosts etcd Server container | Configurable(<i>Recommended: Three replicas)</i> |
| etcd | Service | Exposes etcd DB endpoint | Single |
| etcd-headless | Service | Exposes etcd DB endpoint | Single |
| io-engine| DaemonSet | Hosts Mayastor I/O engine| User-selected nodes |
| DiskPool | CRD | Declares a DiskPool's desired state and reflects its current state | User-defined, one or many |
| **Additional components**  |
| metrics-exporter-pool | Sidecar container (within io-engine DaemonSet)| Exports pool related metrics in Prometheus format | All worker nodes |
| pool-metrics-exporter | Service| Exposes exporter API endpoint to Prometheus | Single |
| promtail | DaemonSet| Scrapes logs of Mayastor-specific pods and exports them to Loki| All worker nodes |
| loki | StatefulSet| Stores the historical logs exported by promtail pods | Single |
| loki | Service| Exposes the Loki API endpoint via ClusterIP | Single |

## Component Roles

### io-engine

The io-engine pods encapsulate Mayastor containers, which implement the I/O path from the block devices at the persistence layer, up to the relevant initiators on the worker nodes mounting volume claims.
The instance of the Mayastor running inside the container performs four major classes of functions:
- Presents a gRPC interface to the components which allow it to orchestrate the creation, configuration, and deletion of Mayastor-managed objects hosted by that instance.
- Creates and manages DiskPools hosted on that node.
- Creates, exports, and manages volume controller objects hosted on that node.
- Creates and exposes replicas from DiskPools hosted on that node over NVMe-TCP.

When an io-engine pod starts running, an init container attempts to verify connectivity to the agent-core in the namespace where Mayastor has been deployed. If a connection is established, the Mayastor container is started, and the instance performs registration with the control plane. In this way, the agent-core maintains a registry of nodes and their current states.

The scheduling of these pods is determined declaratively by using a DaemonSet specification. By default, a nodeSelector field is used within the pod spec to select all worker nodes to which the user has attached the label `openebs.io/engine=mayastor` as recipients of an io-engine pod. In this way, the node count and location are set appropriately to the hardware configuration of the worker nodes, and the capacity and performance demands of the cluster.

### csi-node

The csi-node pods within a cluster implement the node plugin component of Mayastor's CSI driver. As such, their function is to orchestrate the mounting of Mayastor-provisioned volumes on worker nodes on which application pods consuming those volumes are scheduled. By default, a csi-node pod is scheduled on every node in the target cluster, as determined by a DaemonSet resource of the same name. Each of these pods encapsulates two containers, csi-node, and csi-driver-registrar.

The node plugin does not need to run on every worker node within a cluster and this behavior can be modified, if desired, through the application of appropriate node labeling and the addition of a corresponding nodeSelector entry within the pod spec of the csi-node DaemonSet. However, it should be noted that if a node does not host a plugin pod, then it will not be possible to schedule an application pod on it, which is configured to mount Mayastor volumes.

### etcd
   
etcd is a distributed reliable key-value store for the critical data of a distributed system. Mayastor uses etcd as a reliable persistent store for its configuration and state data.

### Supportability:

The supportability tool is used to create support bundles (archive files) by interacting with multiple services present in the system. These bundles contain information about the entire Mayastor system, as well as specific Mayastor resources like volumes, pools and nodes, and can be used for debugging. It can collect the following information:
- Topological information of Mayastor's resource(s) by interacting with the REST service

- Historical logs by interacting with Loki. If Loki is unavailable, it interacts with the kube-apiserver to fetch logs.

- Mayastor-specific Kubernetes resources by interacting with the kube-apiserver

- Mayastor-specific information from etcd (internal) by interacting with the etcd server.

### Loki:

    [Loki](https://grafana.com/oss/loki/) aggregates and centrally stores logs from all Mayastor containers which are deployed in the cluster.

### Promtail:
     
    Promtail is a log collector built specifically for Loki. It uses the configuration file for target discovery and includes analogous features for labeling, transforming, and filtering logs from containers before ingesting them to Loki.