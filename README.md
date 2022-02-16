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

By comparison most pre-CAS shared everything storage systems are widely thought to impart an overhead of at least 40% and sometimes as much as 80% or more as compared to the capabilities of the underlying devices or cloud volumes; additionally pre-CAS shared storage scales in an unpredictale manner as I/O from many workloads interact and compete for the capabilities of the shared storage system.

While Mayastor utilizes NVMe-oF it does not require NVMe devices or cloud volumes to operate.

## Where can I find Mayastor?

The Mayastor product is associated with multiple repositories under the OpenEBS project. The list of repositories are as follows:
- [openebs/mayastor](https://github.com/openebs/mayastor) : This repository contains specifications for Mayastor Data Plane components.
- [openebs/mayastor-control-plane](https://github.com/openebs/mayastor-control-plane) : This repository contains specifications for Mayastor Control Plane components.
- [openebs/mayastor-api](https://github.com/openebs/mayastor-api) : This repository contains protocol buffer definitions and OpenAPI specifications for Mayastor




