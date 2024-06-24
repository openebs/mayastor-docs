# Scope

{% hint style="danger" %}
This website/page will be End-of-life (EOL) after 31 August 2024. We recommend you to visit [OpenEBS Documentation](https://openebs.io/docs/user-guides/replicated-storage-user-guide/replicated-pv-mayastor/rs-installation) for the latest Mayastor documentation (v2.6 and above).
 
Mayastor is now also referred to as OpenEBS Replicated PV Mayastor.
{% endhint %}

This quickstart guide describes the actions necessary to perform a basic installation of Mayastor on an existing Kubernetes cluster, sufficient for evaluation purposes. It assumes that the target cluster will pull the Mayastor container images directly from OpenEBS public container repositories. Where preferred, it is also possible to [build Mayastor locally from source](https://github.com/openebs/Mayastor/blob/develop/doc/build.md) and deploy the resultant images but this is outside of the scope of this guide.

Deploying and operating Mayastor in production contexts requires a foundational knowledge of Mayastor internals and best practices, found elsewhere within this documentation. 



