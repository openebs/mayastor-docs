# Known Limitations

## Volume and Pool Capacity Expansion

Once provisioned, neither Mayastor Disk Pools nor Mayastor Volumes can be re-sized. A Mayastor Pool can have only a single block device as a member. Mayastor Volumes are exclusively thick-provisioned.

## Volumes are "Highly Durable" but without multipathing are not "Highly Available"

Mayastor Volumes can be configured \(or subsequently re-configured\) to be composed of 2 or more "children" or "replicas"; causing synchronously mirrored copies of the volumes's data to be maintained on more than one worker node and Disk Pool. This contributes additional "durability" at the persistence layer, ensuring that viable copies of a volume's data remain even if a Disk Pool device is lost.

A Mayastor volume is currently accessible to an application only via a single target instance \(NVMe-oF\) of a single Mayastor pod. However, if that Mayastor pod ceases to run \(through the loss of the worker node on which it's scheduled, execution failure, crashloopbackoff etc.\) the [HA switch-over module](https://mayastor.gitbook.io/introduction/advanced-operations/ha) detects the failure and moves the target to a healthy worker node to ensure I/O continuity.

