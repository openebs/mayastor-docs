---
description: 'The native NVMe-oF CAS engine of OpenEBS'
---

# Welcome to Mayastor!

{% hint style="danger" %}
This is not the latest version of the documentation. Mayastor documentation up to version 2.5 is available on this website.
Mayastor is now referred to as OpenEBS Replicated Storage. The latest Mayastor documentation (v2.6 and above) is published as part of [OpenEBS Documentation](https://openebs.io/docs).
{% endhint %}

## What is Mayastor?

**Mayastor** is a performance optimised "Container Attached Storage" (CAS) solution of the CNCF project [**OpenEBS**](https://openebs.io/). The goal of OpenEBS is to extend Kubernetes with a declarative data plane, providing flexible persistent storage for stateful applications.

Design goals for Mayastor include:

* Highly available, durable persistence of data
* To be readily deployable and easily managed by autonomous SRE or development teams
* To be a low-overhead abstraction for NVMe-based storage devices 

Mayastor incorporates Intel's [Storage Performance Development Kit](https://spdk.io/). It has been designed from the ground up to leverage the protocol and compute efficiency of NVMe-oF semantics, and the performance capabilities of the latest generation of solid-state storage devices, in order to deliver a storage abstraction with performance overhead measured to be within the range of single-digit percentages.

By comparison, most "shared everything" storage systems are widely thought to impart an overhead of at least 40% (and sometimes as much as 80% or more) as compared to the capabilities of the underlying devices or cloud volumes; additionally traditional shared storage scales in an unpredictable manner as I/O from many workloads interact and compete for resources.

While Mayastor utilizes NVMe-oF it does not require NVMe devices or cloud volumes to operate and can work well with other device types.

## Where can I find Mayastor?

Mayastor's source code and documentation are distributed amongst a number of GitHub repositories under the OpenEBS organisation. The following list describes some of the main repositories but is not exhaustive. 

- [openebs/mayastor](https://github.com/openebs/mayastor) : contains the source code of the data plane components
- [openebs/mayastor-control-plane](https://github.com/openebs/mayastor-control-plane) : contains the source code of the control plane components
- [openebs/mayastor-api](https://github.com/openebs/mayastor-api) : contains common protocol buffer definitions and OpenAPI specifications for Mayastor components
- [openebs/mayastor-dependencies](https://github.com/openebs/mayastor-dependencies) : contains dependencies common to the control and data plane repositories
- [openebs/mayastor-extensions](https://github.com/openebs/mayastor-extensions) : contains components and utilities that provide extended functionalities like ease of installation, monitoring and observability aspects
- [openebs/mayastor-docs](https://github.com/openebs/mayastor-docs) : contains Mayastor's user documenation 




