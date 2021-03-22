# Known Limitations


## Volume and Pool Capacity Expansion

Once provisioned, neither Mayastor Disk Pools nor Mayastor Volumes can be re-sized.  A Mayastor Pool can have only a single block device as a member.  Mayastor Volumes are exclusively thick-provisioned.

## Snapshots and Clones

Mayastor has no snapshot or cloning capabilities.

## Volumes are "Highly Durable" but not "Highly Available"

Mayastor Volumes can be configured (or subsequently re-configured) to be composed of 2 or more "children" or "replicas"; causing synchronously mirrored copies of the volumes's data to be maintained on more than one worker node and Disk Pool.  This contributes additional "durability" at the persistence layer, ensuring that viable copyies of a volume's data remain even if a Disk Pool device is lost.

However a Mayastor volume is accessible to an application only via a single target instance (NVMe-oF, or iSCSI) of a single Mayastor pod.  If that pod terminates (through the loss of the worker node on which it's scheduled, excution failure, pod eviction etc.) then there will be no viable I/O path to any remaining healthy replicas and access to data on the volume cannot be maintained.

