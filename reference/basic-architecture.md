# Basic Architecture

The objective of this section is to provide the user and evaluator of Mayastor with a topological view of the gross anatomy of a Mayastor deployment.   A description will be made of the expected pod inventory of a correctly deployed cluster, the roles and functions of the constituent pods and related Kubernetes resource types, and of the high level interactions between them and the orchestration thereof.

More detailed guides to Mayastor's components, their design and internal structure, and instructions for building Mayastor from source, are maintained within the [project's GitHub repository](https://github.com/openebs/Mayastor).

## Topology

![Figure 1. Example cluster deployment configured with three Mayastor Storage Nodes](../.gitbook/assets/basic_cluster_topology.png)

## Dramatis Personae

| Name | Resource Type | Function | Occurrence in Cluster |
| :--- | :--- | :--- | :--- |
| **moac** | Pod | Hosts control plane containers | Single |
| moac | Service | Exposes MOAC REST service end point | Single |
| moac | Deployment | Declares desired state for the MOAC pod | Single |
|   |   |   |   |
| **mayastor-csi** | Pod | Hosts CSI Driver node plugin containers | All worker nodes |
| mayastor-csi | DaemonSet | Declares desired state for mayastor-csi pods | Single |
|   |    |   |   |
| **mayastor** | Pod | Hosts Mayastor I/O engine container | User-selected nodes |
| mayastor | DaemonSet | Declares desired state for Mayastor pods | Single |
|   |   |   |   |
| **nats** | Pod | Hosts NATS Server container | Single |
| nats | Deployment | Declares desired state for NATS pod | Single |
| nats | Service | Exposes NATS message bus end point | Single |
|   |   |   |   |
| **mayastornodes** | CRD | Inventories and reflects the state of Mayastor pods | One per Mayastor Pod |
| **mayastorpools** | CRD | Declares a Mayastor pool's desired state and reflects its current state | User-defined, zero to many |
| **mayastorvolumes** | CRD | Inventories and reflects the state of Mayastor-provisioned Persistent Volumes | User-defined, zero to many |

## Component Roles

### MOAC

A Mayastor deployment features a single MOAC pod, declared via a Deployment resource of the same name and has its API's gRPC endpoint exposed via a cluster Service, also of the same name.  The MOAC pod is the principle control plane actor and encapsulates containers for both the Mayastor CSI Driver's controller implementation \(and its external-attacher sidecar\) and the MOAC component itself.

The MOAC component implements the bulk of the Mayastor-specific control plane.  It is called by the CSI Driver controller in response to dynamic volume provisioning events, to orchestrate the creation of a nexus at the Mayastor pod of an appropriate node and also the creationion of any additional data replicas on other nodes as may be required to satisfy the desired configuration state of the PVC  \(i.e. replication factor &gt; 1\).  MOAC is also responsible for the creation and status reporting of Storage Pools, for which purpose it implements a watch on the Kubernetes API server for relevant custom resource objects \(mayastorpools.openebs.io\).

MOAC exposes a REST API endpoint on the cluster using a Kubernetes Service of the same name.  This is currently used to support the export of volume metrics to Prometheus/Grafana, although this mechanism will change in later releases.

### Mayastor

The Mayastor pods of a deployment are its principle data plane actors, encapsulating the Mayastor containers which implement the I/O path from the block devices at the persistence layer, up to the relevant initiators on the worker nodes mounting volume claims.

The instance of the `mayastor` binary running inside the container performs four major classes of functions:

* Present a gRPC interface to the MOAC control plane component, to allow the latter to orchestrate creation, configuration and deletion of Mayastor managed objects hosted by that instance
* Create and manage storage pools hosted on that node
* Create, export and manage nexus objects \(and by extension, volumes\) hosted on that node
* Create and share "replicas" from storage pools hosted on that node
  * Local replica -&gt; loopback -&gt; Local Nexus
  * Local replica - &gt; NVMe-F TCP -&gt;  Remote Nexus \(hosted by a Mayastor container on another node\)
  * Remote replicas are employed by a Nexus as synchronous data copies, where replication is in use

When a Mayastor pod starts running, an init container attempts to verify connectivity to the NATS message bus in the Mayastor namespace.  If a connection can be established the Mayastor container is started, and the Mayastor instance performs registration with MOAC over the message bus.  In this way, MOAC maintains a registry of nodes \(specifically, running Mayastor instances\) and their current state.  For each registered Mayastor container/instance, MOAC creates a MayastorNode custom resource within the Kubernetes API of the cluster.

The scheduling of Mayastor pods is determined declaratively by using a DaemonSet specification.  By default, a `nodeSelector` field is used within the pod spec to select all worker nodes to which the user has attached the label `openebs.io/engine=mayastor` as recipients of a Mayastor pod.   It is in this way that the MayastorNode count and location is set appropriate to the hardware configuration of the worker nodes \(i.e. which nodes host the block storage devices to be used\), and capacity and performance demands of the cluster.

### Mayastor-CSI

The mayastor-csi pods within a cluster implement the node plugin component of Mayastor's CSI driver.  As such, their function is to orchestrate the mounting of Maystor provisioned volumes on worker nodes on which application pods consuming those volumes are scheduled.  By default a mayastor-csi pod is scheduled on every node in the target cluster, as determined by a DaemonSet resource of the same name.  These pods each encapsulate two containers, `mayastor-csi` and `csi-driver-registrar`

 It is not necessary for the node plugin to run on every worker node within a cluster and this behaviour can be modified if so desired through the application of appropriate node labeling and the addition of a corresponding  `nodeSelector` entry within the pod spec of the mayastor-csi DaemonSet.  It should be noted that if a node does not host a plugin pod, then it will not be possible to schedule pod on it which is configured to mount Mayastor volumes.

Further detail regarding the implementation of CSI driver components and their function can be found within the Kubernetes CSI Developer Documentation.

### NATS

NATS is a high performance open source messaging system.  It is used within Mayastor as the transport mechanism for registration messages passed between Mayastor I/O engine pods running in the cluster and the MOAC component which maintains an inventory of active Mayastor nodes and reflects this via CRUD actions on MayastorNode custom resources.

In future releases of Mayastor, the control plane will transition towards a more microservice-like architecture following the saga pattern, whereupon a highly available NATS deployment within the Mayastor namespace will be employed as an event bus. 

## 



