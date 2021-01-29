---
description: 'The native NVMe-oF CAS engine for OpenEBS, by OpenEBS'
---

# Welcome to Mayastor!

## What is Mayastor?

**Mayastor** is currently under development as a storage engine option of the Open Source CNCF project [**OpenEBS**](https://openebs.io/).  OpenEBS is a "Container Attached Storage" solution which extends Kubernetes with a declarative data plane, providing flexible persistent storage for stateful applications.

Design goals for Mayastor include:

* Highly available, durable persistence of data.
* To be readily deployable and easily managed by autonomous SRE or development teams.
* To be a low-overhead abstraction.

Mayastor incorporates Intel's [Storage Performance Development Kit](https://spdk.io/).  It has been designed from the ground up to leverage the protocol and compute efficiency of NVMe-oF semantics,  and the performance capabilities of the latest generation of solid-state storage devices, in order to deliver a storage abstraction with performance overhead measured to be within the range of single-digit percentages.

## Where can I find Mayastor?

The Mayastor GitHub repository can be found [here](https://github.com/openebs/Mayastor).

## Project Status

{% hint style="warning" %}
**Mayastor is beta software**.  It is considered largely, if not entirely, feature complete and substantially without major known defects.  Minor and unknown defects can be expected; **please deploy accordingly**.
{% endhint %}

## "Who" are Mayastor?

Mayastor is a component of  OpenEBS and is actively developed and maintained by [MayaData](https://mayadata.io/), as part of the Kubera platform for data agility.  The OpenEBS family of storage engines are fully Open Source and "free forever" - feedback and contributions are welcomed.

![](.gitbook/assets/openebs-stacked-color.png)

![](.gitbook/assets/mayadata-logo-1d5e6edb8a36beb68572ffc65dfe7a4e.svg)

