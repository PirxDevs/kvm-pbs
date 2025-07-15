# KVM backup and restore scripts

These scripts enable you to back up and restore KVM virtual machines managed by libvirt to and from the Proxmox Backup Server.

Before using them, copy the file "kvm-pbs.example.conf" to "kvm-pbs.conf," and then review and edit it to match your configuration and needs.

## kvm2pbs

This script will back up all virtual machines except those listed as excluded in the kvm-pbs.conf file. It only supports machines that are running or powered off and will skip those in any other state.

Backups will include all virtual disks based on LVM volumes (thick or thin) and QCOW2 image files. If the machine is running, LVM/QCOW2 snapshots will be used to ensure the disks are in a consistent state. To accomplish this, all file system operations inside the VM must be temporarily paused. For this reason, only machines with a running QEMU Guest Agent can be backed up while running.

To run the backup, simply run the script, or preferrably, add it to the cron job.

## pbs2kvm

This script allows you to restore your virtual machines from a backup. Simply run the script, and it will return a list of available backup groups (VMs).

``
# ./pbs2kvm
Available backup groups:

host/myvm
host/othervm
``

Add the desired group name as an option to the script, then run it again. It will list the available snapshots for the given backup group.

``
# ./pbs2kvm host/myvm
Available backup snapshots for group host/myvm:

2025-05-25T05:20:25Z
2025-06-25T05:03:13Z
``

Once more, add the snapshot name as an option to the script and run it. This will list the contents of the given VM snapshot (backup).

``
# ./pbs2kvm host/myvm 2025-06-26T02:02:03Z
Contents of backup snapshot host/myvm/2025-06-26T02:02:03Z:

Virtual machine name: myvm

Object          Type   Format Device   Original path
--------------- ------ ------ -------- ---------------------------------------
vm.conf.blob    Config -      -        /etc/libvirt/qemu/myvm.xml
disk-sda.img    Disk   RAW    sda      /dev/vmvg/myvm-root
disk-sdb.img    Disk   RAW    sdb      /dev/vmvg/myvm-data
``

You must then restore each virtual disk separately, followed by the virtual machine configuration. For example:

`./pbs2kvm host/myvm 2025-05-25T05:20:25Z disk-sda.img /dev/vmvg/myvm-root`
`./pbs2kvm host/myvm 2025-05-25T05:20:25Z disk-sdb.img /dev/vmvg/myvm-data`
`./pbs2kvm host/myvm 2025-05-25T05:20:25Z vm.conf.blob /etc/libvirt/qemu/myvm.xml`
