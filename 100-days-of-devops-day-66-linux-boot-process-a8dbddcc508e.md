
# 100 Days of DevOps — Day 66-Linux Boot Process

Welcome to Day 66 of 100 Days of DevOps, Focus for today is Linux Boot Process

## Linux Boot Process
> # UEFI/BIOS → Bootable Disk → BootLoader(GRUB2) → Kernel → Initramfs → init/systemd → Services

* *When the System Boot from Firmware Interface(UEFI/BIOS) it work is to find the bootable disk*

* *Bootable Disk can be anything(hardrive,cd-rom,usb stick)*

* *Then on the bootable disk we need bootloader(**default:grub2**)*

* *The function of Grub2 is to boot Kernel*

* *Now kernel from the initial stage need drivers and that is provided by initramfs*

* *Init is the first process which kernel looks for(On modern Linux system it’s systemd)*

* *Systemd/Init is going to start all services*

*Now let’s go deeper into each point I mentioned earlier*

***UEFI(Unified Extensible Firmware Interface)**: Introduced to replace BIOS and it’s specially useful when we want to access larger disk(**>2TB**) in case of GPT(GUID Partition Table)*

*In BIOS based system, MBR(Master Boot Record) is read from disk and the stage1 boot loader is activated(446bytes) and it’s task is to load stage2 boot loader that resides in the first MB of the disk*

*In case of UEFI system, **GPT** partition table is used(/efi/grub)(MBR is for backward compatibility)*

    *# gdisk /dev/sda*

    *GPT fdisk (gdisk) version 0.8.6*

    *Partition table scan:*

    *MBR: protective*

    *BSD: not present*

    *APM: not present*

    *GPT: present*

    *Found valid GPT with protective MBR; using GPT.*

    *Command (? for help): p*

    *Disk /dev/xvda: 20971520 sectors, 10.0 GiB*

    *Logical sector size: 512 bytes*

    *Disk identifier (GUID): 7D648EBF-FE14–4BC0-AB70-F42E6705D11F*

    *Partition table holds up to 128 entries*

    *First usable sector is 34, last usable sector is 20971486*

    *Partitions will be aligned on 2048-sector boundaries*

    *Total free space is 2014 sectors (1007.0 KiB)*

    *Number Start (sector) End (sector) Size Code Name*

    *1 2048 4095 1024.0 KiB EF00   EFI System Partition*

    *2 4096 20971486 10.0 GiB 0700*

*Now to get more information about first 512bytes*

    *# xxd -l 512 /dev/xvda*

    *0000000: eb63 9000 0000 0000 0000 0000 0000 0000  .c..............*

    *0000010: 0000 0000 0000 0000 0000 0000 0000 0000  ................*

    *0000020: 0000 0000 0000 0000 0000 0000 0000 0000  ................*

    *0000030: 0000 0000 0000 0000 0000 0000 0000 0000  ................*

    *0000040: 0000 0000 0000 0000 0000 0000 0000 0000  ................*

    *0000050: 0000 0000 0000 0000 0000 0080 0008 0000  ................*

    *0000060: 0000 0000 fffa 9090 f6c2 8074 05f6 c270  ...........t...p*

    *0000070: 7402 b280 ea79 7c00 0031 c08e d88e d0bc  t....y|..1......*

    *0000080: 0020 fba0 647c 3cff 7402 88c2 52be 057c  . ..d|<.t...R..|*

    *0000090: b441 bbaa 55cd 135a 5272 3d81 fb55 aa75  .A..U..ZRr=..U.u*

    *00000a0: 3783 e101 7432 31c0 8944 0440 8844 ff89  7...t21..D.@.D..*

    *00000b0: 4402 c704 1000 668b 1e5c 7c66 895c 0866  D.....f..\|f.\.f*

    *00000c0: 8b1e 607c 6689 5c0c c744 0600 70b4 42cd  ..`|f.\..D..p.B.*

    *00000d0: 1372 05bb 0070 eb76 b408 cd13 730d 5a84  .r...p.v....s.Z.*

    *00000e0: d20f 83de 00be 857d e982 0066 0fb6 c688  .......}...f....*

    *00000f0: 64ff 4066 8944 040f b6d1 c1e2 0288 e888  d.@f.D..........*

    *0000100: f440 8944 080f b6c2 c0e8 0266 8904 66a1  .@.D.......f..f.*

    *0000110: 607c 6609 c075 4e66 a15c 7c66 31d2 66f7  `|f..uNf.\|f1.f.*

    *0000120: 3488 d131 d266 f774 043b 4408 7d37 fec1  4..1.f.t.;D.}7..*

    *0000130: 88c5 30c0 c1e8 0208 c188 d05a 88c6 bb00  ..0........Z....*

    *0000140: 708e c331 dbb8 0102 cd13 721e 8cc3 601e  p..1......r...`.*

    *0000150: b900 018e db31 f6bf 0080 8ec6 fcf3 a51f  .....1..........*

    *0000160: 61ff 265a 7cbe 807d eb03 be8f 7de8 3400  a.&Z|..}....}.4.*

    *0000170: be94 7de8 2e00 cd18 ebfe 4752 5542 2000  ..}.......**GRUB** .*

    *0000180: 4765 6f6d 0048 6172 6420 4469 736b 0052  Geom.Hard Disk.R*

    *0000190: 6561 6400 2045 7272 6f72 0d0a 00bb 0100  ead. Error......*

    *00001a0: b40e cd10 ac3c 0075 f4c3 0000 0000 0000  .....<.u........*

    *00001b0: 0000 0000 0000 0000 0000 0000 0000 8000  ................*

    *00001c0: 0100 eefe ffff 0100 0000 ffff 3f01 0000  ............?...*

    *00001d0: 0000 0000 0000 0000 0000 0000 0000 0000  ................*

    *00001e0: 0000 0000 0000 0000 0000 0000 0000 0000  ................*

    *00001f0: 0000 0000 0000 0000 0000 0000 0000 55aa  ..............U.*

*In case of grub2 if we want to make any change,go to*

    *# ls -l /etc/sysconfig/grub*

    *lrwxrwxrwx. 1 root root 17 Oct 20 12:52 /etc/sysconfig/grub -> /etc/default/grub*

    *GRUB_CMDLINE_LINUX=”console=ttyS0,115200n8 console=tty0 net.ifnames=0 crashkernel=auto*

*At the end of this line I want to change default scheduler from deadline to noop*

    *# cat /sys/block/xvda/queue/scheduler*

    *noop [deadline] cfq*

*To do that*

    *GRUB_CMDLINE_LINUX=”console=ttyS0,115200n8 console=tty0 net.ifnames=0 crashkernel=auto elevator=noop”*

*To make these changes persistent*
> # -o, — output=FILE output generated config to FILE [default=stdout]

    *# grub2-mkconfig -o /boot/grub2/grub.cfg*

    *Generating grub configuration file …*

    *Found linux image: /boot/vmlinuz-3.10.0–514.el7.x86_64*

    *Found initrd image: /boot/initramfs-3.10.0–514.el7.x86_64.img*

    *Found linux image: /boot/vmlinuz-0-rescue-be7a44b4b98e404a8cdcd062c4733c10*

    *Found initrd image: /boot/initramfs-0-rescue-be7a44b4b98e404a8cdcd062c4733c10.img*

    *done*

*Looking forward from you guys to join this journey and spend a minimum an hour every day for the next 100 days on DevOps work and post your progress using any of the below medium.*

* *Twitter: [@100daysofdevops](http://twitter.com/100daysofdevops) OR @[lakhera2015](https://twitter.com/lakhera2015)*

* *Facebook: [https://www.facebook.com/groups/795382630808645/](https://www.facebook.com/groups/795382630808645/)*

* *Medium: [https://medium.com/@devopslearning](https://medium.com/@devopslearning)*

* *Slack: [https://devops-myworld.slack.com/messages/CF41EFG49/](https://devops-myworld.slack.com/messages/CF41EFG49/)*

* *GitHub Link:[https://github.com/100daysofdevops](https://github.com/100daysofdevops)*

***Reference***
[**100 Days of DevOps — Day 0**
*D-day is just one day away and finally, this is a continuation of the post(I posted a month earlier)*medium.com](https://medium.com/@devopslearning/100-days-of-devops-day-0-4f2c9750542d)
[**100 Days of DevOps**
*Motivation*medium.com](https://medium.com/@devopslearning/100-days-of-devops-81faf13bf772)
