# Known Issues

## Live Issue Tracker

Mayastor is currently considered to be beta software.

> "\(it\) will generally have many more bugs in it than completed software and speed or performance issues, and may still cause crashes or data loss."

The project's maintainers operate a live issue tracking dashboard for defects which they have under active triage and investigation. It can be accessed [here](https://mayadata.atlassian.net/secure/Dashboard.jspa?selectPageId=10015). You are strongly encouraged to familiarise yourself with the issues identified there before using Mayastor and when raising issue reports in order to limit to the extent possible redundant issue reporting.

## How is Mayastor Tested?

Mayastor's maintainers perform integration and end-to-end testing on nightly builds and named releases. Clusters used to perform this testing are composed of worker nodes running Ubuntu 20.04.2 LTS, using the docker runtime 20.10.5 under Kubernetes version 1.19.8. Other testing efforts are underway including soak testing and failure injection testing.

We periodically access the labs of partners and community members for scale and performance testing and would welcome offers of any similar or other testing assistance.

## Common Installation Issues

### A Mayastor pod restarts unexpectedly with exit code 132 whilst mounting a PVC

The Mayastor process has been sent the SIGILL signal as the result of attempting to execute an illegal instruction. This indicates that the host node's CPU does not satisfy the prerequisite instruction set level for Mayastor \(SSE4.2 on x86-64\).

### Deploying Mayastor on RKE & Fedora CoreOS

In addition to ensuring that the general prerequisites for installation are met, it is necessary to add the following directory mapping to the `services_kublet->extra_binds` section of the cluster's`cluster.yml file.`

```text
/opt/rke/var/lib/kubelet/plugins:/var/lib/kubelet/plugins
```

If this is not done, CSI socket paths won't match expected values and the Mayastor CSI driver registration process will fail, resulting in the inability to provision Mayastor volumes on the cluster.

## Other Issues

### Lengthy worker node reboot times

When rebooting a node that runs applications mounting Mayastor volumes, this can take tens of minutes. The reason is the long default NVMe controller timeout \(`ctrl_loss_tmo`\). The solution is to follow the best k8s practices and cordon the node ensuring there aren't any application pods running on it before the reboot. Setting `ioTimeout` storage class parameter can be used to fine-tune the timeout.

### Node restarts on scheduling an application 

Deploying an application on a K8s environment which hosts Mayastor along with Prometheus exporter causes the application node to restart.
The issue originated because of a kernel bug. Once the nexus disconnects, the entries under `/host/sys/class/hwmon/` should get removed, which does not happen in this case(The issue was fixed via this [kernel patch](https://www.mail-archive.com/linux-kernel@vger.kernel.org/msg2413147.html)).

**Fix:** Since the issue got fixed in the next kernel version, i.e, `linux-modules-extra-5.13.0-22-generic`, upgrading ubuntu-based OS kernel to >= extra-5.13.0 should fix node restarts issue.
