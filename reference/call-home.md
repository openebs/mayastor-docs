# Call-home metrics

In the interest of gaining perspective on storage usage trends, Mayastor collects several anonymous data metrics from Mayastor instances deployed by its users. 



In the interest of gaining perspective on storage usage trends, Mayastor collects several anonymous data metrics from Mayastor instances deployed by its users. When you use the Mayastor helm chart (v2.0.0 onwards) to deploy Mayastor on to your Kubernetes cluster, a Kubernetes deployment called `obs-callhome` is deployed along with it. A Kubernetes Pod from this deployment collects the data points from your Mayastor cluster.

The data is anonymous. The data is encrypted at rest, before being transmitted over the Internet. A summary of the data points collected is furnished below. 


| **Cluster information** |
| :--- |
|**K8s cluster ID**:This is a SHA-256 hashed value of the UID of your Kubernetes cluster's `kube-system` namespace.|
|**K8s node count**:The number of nodes in your Kubernetes cluster.|
|**Product name**:The name given during helm install. |
|**Product version**: The deployed version of Mayastor.|
|**Deploy namespace**:This is a SHA-256 hashed value of the name of the Kubernetes namespace where you've deployed the Mayastor Helm chart.|
|**Storage node count**:This is the number of nodes which Mayastor is configured to use for volume provisioning.|

|**Pool information**|
| :--- |
|**Pool count**:This is the number of Mayastor DiskPools in your cluster.|
|**Pool maximum size**:This is the capacity of the Mayastor DiskPool with the highest capacity.|
|**Pool minimum size**:his is the capacity of the Mayastor DiskPool with the lowest capacity.|
|**Pool mean size**:This is the average capacity of the Mayastor DiskPools in your cluster.|
|**Pool capacity percentiles**:his calculates and returns the capacity distribution of Mayastor DiskPools for the 50th, 75th and the 90th percentiles.|


|**Volume information**|
| :--- |
||


