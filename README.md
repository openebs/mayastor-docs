---
description: 'The native NVMe-oF CAS engine for OpenEBS, by OpenEBS'
---

# Welcome to Mayastor!

## What is Mayastor?

**Mayastor** is currently under development as a sub project of the Open Source CNCF project [**OpenEBS**](https://openebs.io/). OpenEBS is a "Container Attached Storage" or CAS solution which extends Kubernetes with a declarative data plane, providing flexible persistent storage for stateful applications.

Design goals for Mayastor include:

* Highly available, durable persistence of data.
* To be readily deployable and easily managed by autonomous SRE or development teams.
* To be a low-overhead abstraction.

OpenEBS Mayastor incorporates Intel's [Storage Performance Development Kit](https://spdk.io/). It has been designed from the ground up to leverage the protocol and compute efficiency of NVMe-oF semantics, and the performance capabilities of the latest generation of solid-state storage devices, in order to deliver a storage abstraction with performance overhead measured to be within the range of single-digit percentages.

By comparison most pre-CAS shared everything storage systems are widely thought to impart an overhead of at least 40% and sometimes as much as 80% or more as compared to the capabilities of the underlying devices or cloud volumes; additionally pre-CAS shared storage scales in an unpredictale manner as I/O from many workloads interact and complete for the capabilities of the shared storage system.

While Mayastor utilizes NVMe-oF it does not require NVMe devices or cloud volumes to operate as is explained below.

## Where can I find Mayastor?

The Mayastor GitHub repository can be found [here](https://github.com/openebs/Mayastor).

## Project Status

{% hint style="warning" %}
**Mayastor is beta software**. It is considered largely, if not entirely, feature complete and substantially without major known defects. Minor and unknown defects can be expected; **please deploy accordingly**.
{% endhint %}

## "Who" are Mayastor?

Mayastor is a component of OpenEBS and is primarily developed and maintained by [MayaData](https://mayadata.io/), which also provides enterprise support and management software for the use of OpenEBS Mayastor and Kubernetes for data. The OpenEBS family of storage engines are fully Open Source and "free forever" - feedback and contributions are welcomed. Other companies also provide support for OpenEBS and these offerings may include support for OpenEBS Mayadata. More information about these companies can be found on the web site for the OpenEBS project at: www.openEBS.io/support

![](.gitbook/assets/openebs-stacked-color.png)

![](.gitbook/assets/mayadata-logo-1d5e6edb8a36beb68572ffc65dfe7a4e.svg)

