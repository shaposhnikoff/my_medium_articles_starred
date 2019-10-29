Unknown markup type 10 { type: [33m10[39m, start: [33m37[39m, end: [33m56[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m67[39m, end: [33m72[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m77[39m, end: [33m87[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m125[39m, end: [33m131[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m166[39m, end: [33m172[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m160[39m, end: [33m183[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m279[39m, end: [33m297[39m }

# Using cloud-init to extend the Root Partition on a new Linux Instance on OCI



Since beginning of March 2018, when you launch a Virtual Machine or a Bare Metal instance on Oracle Cloud Infrastructure — OCI, you can specify whether to use the selected image's default boot volume size (~46.6GB), or you can specify a custom size up to 16 TB.

Oracle recommends you to [resize the partition](https://docs.us-phoenix-1.oraclecloud.com/Content/Block/Tasks/extendingpartition.htm) using a partitioning tool like **growpart, gdisk, parted, **or **fdisk**. And for growing the file system, you have to use utilities like **xfs_growfs **or **resize2fs**, depending on your image's file system type and the selected **Linux distribution**.

![](https://cdn-images-1.medium.com/max/2000/1*HjkbxQHAc3E3xKFaeCufXw.png)

On Oracle Linux & CentOS you can use cloud-init-growpartalong with gdisk and cloud-init to completely automate this process. libicu is a library required for running gdisk and is installed automatically on OL7.x/CentOS 7.x. On Release 6, it needs to be installed separately.

Next, you will create a [cloud-init userdata script](http://cloudinit.readthedocs.io/en/latest/topics/format.html) (shell script) at the time you are launching a new instance in the OCI console (or through API/SDK/Terraform), and that will automatically install these packages and reboot your instance automatically to update the root partition (GPT).

![](https://cdn-images-1.medium.com/max/2000/1*PgTgwmd67mF09qtuzIb9mw.png)

To illustrate that, create a new instance on OCI, specify a custom boot volume size and copy and paste the snippet below into the box available when you select PASTE CLOUD-INIT SCRIPT. This box is available after you clicking on the Show Advanced Options link, located below the Add SSH KEY button:

![Show Advanced Options link](https://cdn-images-1.medium.com/max/2404/1*kaqnscDQzhca0XF5VXgs1A.png)*Show Advanced Options link*

    #!/bin/sh

    sudo yum -y install cloud-utils-growpart

    sudo yum -y install gdisk

    #Required for OL6.x/CentOS6.x
    sudo yum -y install libicu

    sudo reboot

![Custom Boot Volume size along with Cloud-init script at Launch Instance page on OCI](https://cdn-images-1.medium.com/max/2492/1*scxP9mz8pefa_Qw2Tr1wKA.png)*Custom Boot Volume size along with Cloud-init script at Launch Instance page on OCI*

Now you can Launch your instance and wait for all that process to complete (it may take from 2' to 5' depending on the selected compute shape). Once it's completed (after reboot), you can verify that your root partition should have the exactly size you specified for the custom boot volume.

![](https://cdn-images-1.medium.com/max/2000/1*Nf2uqy-NQK8HBBOQTZ0PpA.png)
