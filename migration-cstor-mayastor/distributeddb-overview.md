# Migration from CStor to Mayastor for Distributed Databases (Cassandra)

{% hint style="danger" %}
This website/page will be End-of-life (EOL) after 31 August 2024. We recommend you to visit [OpenEBS Documentation](https://openebs.io/docs/user-guides/replicated-storage-user-guide/replicated-pv-mayastor/rs-installation) for the latest Mayastor documentation (v2.6 and above).
 
Mayastor is now also referred to as OpenEBS Replicated PV Mayastor.
{% endhint %}

This documentation outlines the process of migrating application volumes from CStor to Mayastor. We will leverage Velero for backup and restoration, facilitating the transition from a CStor cluster to a Mayastor cluster. This example specifically focuses on a Google Kubernetes Engine (GKE) cluster.

**Velero Support**: Velero supports the backup and restoration of Kubernetes volumes attached to pods through File System Backup (FSB) or Pod Volume Backup. This process involves using modules from popular open-source backup tools like Restic (which we will utilize).

- For **cloud provider plugins**, refer to the [Velero Docs - Providers section](https://velero.io/docs/main/supported-providers/).
- **Velero GKE Configuration (Prerequisites)**: You can find the prerequisites and configuration details for Velero in a Google Kubernetes Engine (GKE) environment on the GitHub [here](https://github.com/vmware-tanzu/velero-plugin-for-gcp#setup).
- **Object Storage Requirement**: To store backups, Velero necessitates an object storage bucket. In our case, we utilize a Google Cloud Storage (GCS) bucket. Configuration details and setup can be found on the GitHub [here](https://github.com/vmware-tanzu/velero-plugin-for-gcp#setup). 
- **Velero Basic Installation**: For a step-by-step guide on the basic installation of Velero, refer to the [Velero Docs - Basic Install section](https://velero.io/docs/v1.11/basic-install/).

