# Basic Architecture

The objective of this section is to provide the user and evaluator of Mayastor with a topological view of the gross anatomy of a Mayastor deployment. A description will be made of the expected pod inventory of a correctly deployed cluster, the roles and functions of the constituent pods and related Kubernetes resource types, and of the high level interactions between them and the orchestration thereof.

More detailed guides to Mayastor's components, their design and internal structure, and instructions for building Mayastor from source, are maintained within the [project's GitHub repository](https://github.com/openebs/Mayastor).

## Dramatis Personae

| Name | Resource Type | Function | Frequency in Cluster |
| :--- | :--- | :--- | :--- |
| **Control Plane** |
| core-agent | Pod | Principle control plane actor | Single |
| csi-controller | Pod | Hosts Mayastor's CSI controller implementation and CSI provisioner side car| Single |
| rest | Pod | Hosts the public API REST server | Single |
| *rest*  | Service | Exposes the REST API server via NodePort |
| msp-operator | Pod | Hosts Mayastor's pool operator | Single |
| mayastor-csi| Pod | Hosts CSI Driver node plugin containers | All worker nodes |
| nats | Pod | Hosts NATS Server container | Single |
| *nats* | Service | Exposes NATS message bus end point | Single |
| *nats-config* | ConfigMap | NATS cluster configuration data |
| etcd | Pod | Hosts etcd Server container | Single |
| *mayastor-etcd* | Service | Exposes NATS message bus end point | Single |
| *mayastor-etcd-headless* | Service | Exposes NATS message bus end point | Single |
| **Data Plane**|
| mayastor| Pod | Hosts Mayastor I/O engine| User-selected nodes |
| **k8s Resource Types**  |
| mayastorpools | CRD | Declares a Mayastor pool's desired state and reflects its current state | User-defined, one or many |

## Component Roles

### Control Plane

A microservices patterned control plane, centered around a core agent with a publically exposed RESTful API.  This is extended by a dedicated operator responsible for managing the life cycle of "Mayastor Pools" (an abstraction for devices supplying the cluster with persistent backing storage), and a CSI compliant external provisioner (controller).  The source for the control plane components is located in its [own repository](https://github.com/mayadata-io/mayastor-control-plane)

 The control plane uses dedicated, clustered instances of etcd and NATS to persist configuration and state data, and as a message bus / transport for communcation with the data plane components, respectively.

### Mayastor

The Mayastor pods of a deployment are its principle data plane actors, encapsulating the Mayastor containers which implement the I/O path from the block devices at the persistence layer, up to the relevant initiators on the worker nodes mounting volume claims.

The instance of the `mayastor` binary running inside the container performs four major classes of functions:

* Present a gRPC interface to the control plane components, to allow the latter to orchestrate creation, configuration and deletion of Mayastor managed objects hosted by that instance
* Create and manage storage pools hosted on that node
* Create, export and manage nexus objects \(and by extension, volumes\) hosted on that node
* Create and shares "replicas" from storage pools hosted on that node over NVMe-TCP

When a Mayastor pod starts running, an init container attempts to verify connectivity to the NATS message bus in the Mayastor namespace. If a connection can be established the Mayastor container is started, and the Mayastor instance performs registration with MOAC over the message bus. In this way, the core agent maintains a registry of nodes \(specifically, running Mayastor instances\) and their current state.

The scheduling of Mayastor pods is determined declaratively by using a DaemonSet specification. By default, a `nodeSelector` field is used within the pod spec to select all worker nodes to which the user has attached the label `openebs.io/engine=mayastor` as recipients of a Mayastor pod. It is in this way that the MayastorNode count and location is set appropriate to the hardware configuration of the worker nodes \(i.e. which nodes host the block storage devices to be used\), and the capacity and performance demands of the cluster.

### Mayastor-CSI

The mayastor-csi pods within a cluster implement the node plugin component of Mayastor's CSI driver. As such, their function is to orchestrate the mounting of Maystor provisioned volumes on worker nodes on which application pods consuming those volumes are scheduled. By default a mayastor-csi pod is scheduled on every node in the target cluster, as determined by a DaemonSet resource of the same name. These pods each encapsulate two containers, `mayastor-csi` and `csi-driver-registrar`

It is not necessary for the node plugin to run on every worker node within a cluster and this behaviour can be modified if so desired through the application of appropriate node labeling and the addition of a corresponding `nodeSelector` entry within the pod spec of the mayastor-csi DaemonSet. It should be noted that if a node does not host a plugin pod, then it will not be possible to schedule pod on it which is configured to mount Mayastor volumes.

Further detail regarding the implementation of CSI driver components and their function can be found within the Kubernetes CSI Developer Documentation.

### NATS

[NATS](https://nats.io/) is a high performance open source messaging system. It is used by Mayastor as the transport mechanism for registration messages passed between Mayastor I/O engine pods running in the cluster and the core agent component, which maintains an inventory of active Mayastor nodes.  It is also used as the transport abstraction for inter data-plane / control-plane gRPC calls.

### etcd

"[etcd](https://github.com/etcd-io/etcd) is a distributed reliable key-value store for the most critical data of a distributed system"

Mayastor uses etcd as a reliable persistent store for its configuration and state data.

