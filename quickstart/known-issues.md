# Known Issues

## A Mayastor pod restarts unexpectedly with exit code 132 whilst mounting a PVC

The Mayastor process has been sent the SIGKILL signal as the result of attempting to execute an illegal instruction.  This indicates that the host node's CPU does not satisfy the prerequisite instruction set level for Mayastor \(SSE4.2 on x86-64\).

## Mayastor lacks k8s finalizers

Finalizers have yet to implemented on key structures, therefore it is important to ensure that the correct tear down sequences are followed, if unintended data loss is to be avoided.

For example, deleting a Mayastor Pool \(MSP\) whilst it is hosting Volume replicas is possible and will result in the faulting of those replicas.  If the replication factor of an implicated volume is equal to one \(i.e. there is _only_ one data replica\), that volume's data will be irrecoverably deleted along with the pool.

This will be resolved in a future version.

## Deploying Mayastor on RKE & Fedora CoreOS

In addition to ensuring that the general prerequisites for installation are met, it is necessary to add the following directory mapping to the `services_kublet->extra_binds` section of the ckuster's`cluster.yml file.`

```text
/opt/rke/var/lib/kubelet/plugins:/var/lib/kubelet/plugins
```

If this is not done, CSI socket paths won't match expected values and the Mayastor CSI driver registration process will fail, resulting in the inability to provision Mayastor volumes on the cluster.

