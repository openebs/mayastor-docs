# Call-home metrics 

 In the interest of gaining perspective on storage usage trends, Mayastor collects some data metrics from the deployed Mayastor instances. Installing Mayastor using Helm chart (v2.0.0 onwards) deploys a Kubernetes deployment called `obs-callhome`on to the cluster. A Kubernetes pod from this deployment collects the data points from your Mayastor cluster. The collected data is anonymous and is encrypted at rest. 

{% hint style="info" %} 

No identities, host names, passwords, or volume data are collected. **ONLY** the below-mentioned information is collected from the cluster.  

{% endhint %} 


A summary of the data points collected is given below.  

| **Cluster information** | 

| :--- | 

|**K8s cluster ID**: This is a SHA-256 hashed value of the UID of your Kubernetes cluster's `kube-system` namespace.| 

|**K8s node count**: This is the number of nodes in your Kubernetes cluster.| 

|**Product name**: This is the name given during Helm install. | 

|**Product version**: This is the deployed version of Mayastor.| 

|**Deploy namespace**: This is a SHA-256 hashed value of the  name of the  Kubernetes namespace where Mayastor Helm chart is deployed.| 

|**Storage node count**: This is the number of nodes which Mayastor is configured to use for volume provisioning.| 

 
 

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



To disable the collection of data metrics from the cluster, use the following command: 


{% tabs %} 

{% tab title="To disable during installation" %} 

```text 

helm install mayastor mayastor/mayastor -n mayastor --set obs.callhome.enabled=false --create-namespace --version 2.0.0 

``` 

{% endtab %} 
{% tab title="To disable post installation" %} 

```text 

helm upgrade mayastor mayastor/mayastor -n mayastor --set obs.callhome.enabled=false --version 2.0.0 

``` 

{% endtab %} 

{% endtabs %} 

 
 
 
 
 
 
 

 