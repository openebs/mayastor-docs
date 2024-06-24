# Known Issues

{% hint style="danger" %}
This website/page will be End-of-life (EOL) after 31 August 2024. We recommend you to visit [OpenEBS Documentation](https://openebs.io/docs/user-guides/replicated-storage-user-guide/replicated-pv-mayastor/rs-installation) for the latest Mayastor documentation (v2.6 and above).
 
Mayastor is now also referred to as OpenEBS Replicated PV Mayastor.
{% endhint %}

## Installation Issues

### A Mayastor pod restarts unexpectedly with exit code 132 whilst mounting a PVC

The Mayastor process has been sent the SIGILL signal as the result of attempting to execute an illegal instruction. This indicates that the host node's CPU does not satisfy the prerequisite instruction set level for Mayastor \(SSE4.2 on x86-64\).

### Deploying Mayastor on RKE & Fedora CoreOS

In addition to ensuring that the general prerequisites for installation are met, it is necessary to add the following directory mapping to the `services_kublet->extra_binds` section of the cluster's`cluster.yml file.`

```text
/opt/rke/var/lib/kubelet/plugins:/var/lib/kubelet/plugins
```

If this is not done, CSI socket paths won't match expected values and the Mayastor CSI driver registration process will fail, resulting in the inability to provision Mayastor volumes on the cluster.

## Other Issues

### Mayastor pod may restart if a pool disk is inaccessible

If the disk device used by a Mayastor pool becomes inaccessible or enters the offline state, the hosting Mayastor pod may panic.  A fix for this behaviour is under investigation.

### Lengthy worker node reboot times

When rebooting a node that runs applications mounting Mayastor volumes, this can take tens of minutes. The reason is the long default NVMe controller timeout \(`ctrl_loss_tmo`\). The solution is to follow the best k8s practices and cordon the node ensuring there aren't any application pods running on it before the reboot. Setting `ioTimeout` storage class parameter can be used to fine-tune the timeout.

### Node restarts on scheduling an application 

Deploying an application pod on a worker node which hosts Mayastor and Prometheus exporter causes that node to restart.
The issue originated because of a kernel bug. Once the nexus disconnects, the entries under `/host/sys/class/hwmon/` should get removed, which does not happen in this case(The issue was fixed via this [kernel patch](https://www.mail-archive.com/linux-kernel@vger.kernel.org/msg2413147.html)).

**Fix:** Use kernel version 5.13 or later if deploying Mayastor in conjunction with the Prometheus metrics exporter.
