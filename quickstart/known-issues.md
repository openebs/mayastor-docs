# Known Issues

## Live Issue Tracker

Mayastor is currently considered to be beta software.

> "(it) will generally have many more bugs in it than completed software and speed or performance issues, and may still cause crashes or data loss."

The project's maintainers operate a live issue tracking dashboard for defects which they have under active triage and investigation.  It can be accessed [here](https://mayadata.atlassian.net/secure/Dashboard.jspa?selectPageId=10015).  You are strongly encouraged to familisarise yourself with the issues identified there before deploying Mayastor and/or raising further issue reports.

## How is Mayastor Tested?

Mayastor's maintainers perform integration and end-to-end testing on nightly builds and named releases.  Clusters used to perform this testing are composed of worker nodes running Ubuntu 20.04.2 LTS, using the docker runtime 20.10.5 under Kubernetes version 1.19.8.  No other testing configurations are in regular use at this time.

## Common Installation Issues

### A Mayastor pod restarts unexpectedly with exit code 132 whilst mounting a PVC

The Mayastor process has been sent the SIGILL signal as the result of attempting to execute an illegal instruction.  This indicates that the host node's CPU does not satisfy the prerequisite instruction set level for Mayastor \(SSE4.2 on x86-64\).


### Deploying Mayastor on RKE & Fedora CoreOS

In addition to ensuring that the general prerequisites for installation are met, it is necessary to add the following directory mapping to the `services_kublet->extra_binds` section of the ckuster's`cluster.yml file.`

```text
/opt/rke/var/lib/kubelet/plugins:/var/lib/kubelet/plugins
```

If this is not done, CSI socket paths won't match expected values and the Mayastor CSI driver registration process will fail, resulting in the inability to provision Mayastor volumes on the cluster.

