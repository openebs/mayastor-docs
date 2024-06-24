# Call-home metrics 

{% hint style="danger" %}
This website/page will be End-of-life (EOL) after 31 August 2024. We recommend you to visit [OpenEBS Documentation](https://openebs.io/docs/user-guides/replicated-storage-user-guide/replicated-pv-mayastor/rs-installation) for the latest Mayastor documentation (v2.6 and above).
 
Mayastor is now also referred to as OpenEBS Replicated PV Mayastor.
{% endhint %}

 By default, Mayastor collects some basic information related to the number and scale of user-deployed instances. The collected data is anonymous and is encrypted at rest. This data is used to understand storage usage trends, which in turn helps maintainers prioritize their contributions to maximize the benefit to the community as a whole. 

{% hint style="info" %} 

No user-identifiable information, hostnames, passwords, or volume data are collected. **ONLY** the below-mentioned information is collected from the cluster. 

{% endhint %} 


A summary of the information collected is given below.

| **Cluster information** | 
| :--- | 
|**K8s cluster ID**: This is a SHA-256 hashed value of the UID of your Kubernetes cluster's `kube-system` namespace.| 
|**K8s node count**: This is the number of nodes in your Kubernetes cluster.| 
|**Product name**: This field displays the name Mayastor | 
|**Product version**: This is the deployed version of Mayastor.| 
|**Deploy namespace**: This is a SHA-256 hashed value of the  name of the  Kubernetes namespace where Mayastor Helm chart is deployed.| 
|**Storage node count**: This is the number of nodes on which the Mayastor I/O engine is scheduled.| 

 
 

|**Pool information**| 
| :--- | 
|**Pool count**: This is the number of Mayastor DiskPools in your cluster.| 
|**Pool maximum size**: This is the capacity of the Mayastor DiskPool with the highest capacity.| 
|**Pool minimum size**: This is the capacity of the Mayastor DiskPool with the lowest capacity.| 
|**Pool mean size**: This is the average capacity of the Mayastor DiskPools in your cluster.| 
|**Pool capacity percentiles**: This calculates and returns the capacity distribution of Mayastor DiskPools for the 50th, 75th and the 90th percentiles.| 

 
 
 

|**Volume information**| 
| :--- | 
|**Volume count**: This is the number of Mayastor Volumes in your cluster.| 
|**Volume minimum size**: This is the capacity of the Mayastor Volume with the lowest capacity.| 
|**Volume mean size**: This is the average capacity of the Mayastor Volumes in your cluster.| 
|**Volume capacity percentiles**: This calculates and returns the capacity distribution of Mayastor Volumes for the 50th, 75th and the 90th percentiles.| 

 
 

|**Replica Information**| 
| :--- | 
|**Replica count**: This is the number of Mayastor Volume replicas in your cluster.| 
|**Average replica count per volume**: This is the average number of replicas each Mayastor Volume has in your cluster.| 


### How to disable the collection of usage data

To disable the collection of data metrics from the cluster, add `--set obs.callhome.enabled=false` flag to the Helm install command. The Helm command, along with the flag, can either be executed during installation or can be re-executed post-installation.

### Where is the collected data stored?

The collected information is stored on behalf of the OpenEBS project by DataCore Software Inc. in data centers located in Texas, USA.