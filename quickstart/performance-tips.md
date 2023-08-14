# Performance Tips

## CPU isolation

Mayastor will fully utilize each CPU core that it was configured to run on. It will spawn a thread on each and the thread will run in an endless loop serving tasks dispatched to it without sleeping or blocking. There are also other Mayastor threads that are not bound to the CPU and those are allowed to block and sleep. However, the bound threads \(also called reactors\) rely on being interrupted by the kernel and other userspace processes as little as possible. Otherwise, the latency of IO may suffer.

Ideally, the only thing that interrupts Mayastor's reactor would be only kernel time-based interrupts responsible for CPU accounting. However, that is far from trivial. `isolcpus` option that we will be using does not prevent:

* kernel threads and
* other k8s pods to run on the isolated CPU

However, it prevents system services including kubelet from interfering with Mayastor.

### Set Linux kernel boot parameter

Note that the best way to accomplish this step may differ, based on the Linux distro that you are using.

Add the `isolcpus` kernel boot parameter to `GRUB_CMDLINE_LINUX_DEFAULT` in the grub configuration file, with a value which identifies the CPUs to be isolated \(indexing starts from zero here\). The location of the configuration file to change is typically `/etc/default/grub` but may vary. For example when running Ubuntu 20.04 in AWS EC2 Cloud boot parameters are in `/etc/default/grub.d/50-cloudimg-settings.cfg`.

In the following example we assume a system with 4 CPU cores in total, and that the third and the fourth CPU cores are to be dedicated to Mayastor.

```text
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash isolcpus=2,3"
```

### Update grub

{% tabs %}
{% tab title="Command" %}
```bash
sudo update-grub
```
{% endtab %}

{% tab title="Output example" %}
```text
Sourcing file `/etc/default/grub'
Sourcing file `/etc/default/grub.d/40-force-partuuid.cfg'
Sourcing file `/etc/default/grub.d/50-cloudimg-settings.cfg'
Sourcing file `/etc/default/grub.d/init-select.cfg'
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-5.8.0-29-generic
Found initrd image: /boot/microcode.cpio /boot/initrd.img-5.8.0-29-generic
Found linux image: /boot/vmlinuz-5.4.0-1037-aws
Found initrd image: /boot/microcode.cpio /boot/initrd.img-5.4.0-1037-aws
Found Ubuntu 20.04.2 LTS (20.04) on /dev/xvda1
done
```
{% endtab %}
{% endtabs %}

### Reboot the system

{% tabs %}
{% tab title="Command" %}
```bash
sudo reboot
```
{% endtab %}
{% endtabs %}

### Verify isolcpus

Basic verification is by outputting the boot parameters of the currently running kernel:

{% tabs %}
{% tab title="Command" %}
```bash
cat /proc/cmdline
```
{% endtab %}

{% tab title="Example output" %}
```text
BOOT_IMAGE=/boot/vmlinuz-5.8.0-29-generic root=PARTUUID=7213a253-01 ro console=tty1 console=ttyS0 nvme_core.io_timeout=4294967295 isolcpus=2,3 panic=-1
```
{% endtab %}
{% endtabs %}

You can also print a list of isolated CPUs:

{% tabs %}
{% tab title="Command" %}
```bash
cat /sys/devices/system/cpu/isolated
```
{% endtab %}

{% tab title="Example output" %}
```text
2-3
```
{% endtab %}
{% endtabs %}

### Deploy IO-engine daemonset

To edit the `io-engine` daemonset and set the `-l` parameter of mayastor to specify CPU cores that Mayastor reactors should run on, follow these steps:

{% hint style="note" %}
You can modify the `-l` parameter value to specify the CPU cores where Mayastor reactors should run. In the below example, Mayastor runs on the third and fourth CPU cores.
{% endhint %}


1. Verify the current status of the `mayastor-io-engine` DaemonSet using the following command:

{% tabs %}
{% tab title="Command" %}
```text
kubectl get ds mayastor-io-engine -n mayastor
```
{% endtab %}

{% tab title="Output" %}
```text
NAME                 DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR                                         AGE
mayastor-io-engine   3         3         3       0            3           kubernetes.io/arch=amd64,openebs.io/engine=mayastor   22d
```
{% endtab %}
{% endtabs %}

2.  Edit the `mayastor-io-engine` DaemonSet configuration using the `kubectl edit` command:

```
kubectl edit ds mayastor-io-engine -n mayastor
```

> This will open the DaemonSet configuration in your default text editor.

3. Locate the section that defines the `io-engine` container within the DaemonSet configuration. If this section doesn't exist, add it within the containers list.

```
...
containers:
  - name: io-engine
    ...
    args:
      ...
      - "-l2,3"

```

4. Save the changes and exit the text editor.