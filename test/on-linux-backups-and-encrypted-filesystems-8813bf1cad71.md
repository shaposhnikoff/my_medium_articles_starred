Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m153[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m65[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m521[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m39[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m106[39m }

# On Linux, backups and encrypted filesystems

Source: https://nnc3.com/mags/LM10/Magazine/Archive/2005/61/065-071_encrypt/article.html

Today, I want to focus on backups. Specifically, how can I backup my Linux filesystems in a way, that I can:

1. Perform it fairly frequently without much of a hassle. More hassle means that it will not get done as frequently as I would like, and defeating the purpose of maintaining backups

1. Is secure. The media that I want to backup to is very portable, and has a high chance of being misplaced or lost. Encryption of the contents is therefore a requirement.

1. I would like the restoration process to be fairly painless and be able to perform it without many dependencies, preferably by booting into an Arch Linux installation image, for example.

As it is the case with most things in Linux, there are many ways that this could be done. In this case, I’m sticking to rsync as the tool for taking the backup. It can be used to create a replica of a given directory structure in a different location, either on a different filesystem or even on a different networked host fairly easily. In this example, I’ll be using a USB disk that I use for offline storage.

Next, encryption. I would really like to avoid the self encrypting disks especially with the disclosure of issues recently with several leading storage manufacturers that revealed serious vulnerabilities in how the encryption is performed that can lead to compromise of the data fairly trivially. The encryption keys, ciphers and technologies used is something that I would like to control for my backups. Which leads me to encrypted file systems on Linux. There are [several options](https://wiki.archlinux.org/index.php/disk_encryption), when it comes to this topic, and the ones I short-listed are dm-crypt and EncFS. The former is natively supported in most distros, is mature and works at the block level. The latter works in the user space and is simple to get started with. I quickly eliminated EncFS due to the many security issues surrounding it that was discovered as part of a [security audit](https://defuse.ca/audits/encfs.htm) of its implementation some time back.

So now, the tools we will be dealing with are rsync + dm-crypt on Linux with a simple script to make the process simple.

First, create a partition on the external storage to house the backup. Tools such as fdisk, gdisk and cfdisk can be used for this.

To encrypt the volume for use with the backup, follow the steps below:

    # cryptsetup -v --type luks --cipher aes-xts-plain64 --key-size 256 --hash sha256 --iter-time 2000 --use-urandom --verify-passphrase luksFormat /dev/sdcX

This sets up a new dm-crypt device in LUKS encryption mode and encrypts the master key with a key derivation algorithm that uses sha256 and 2000 rounds in the example above. It sets up the LUKS device header in the device with this configuration, but we still need to create the file system in order to start using it. We achieve this by:

    # cryptsetup open /dev/sdcX backup
    # mkfs.ext4 /dev/mapper/backup

As you can see, the path provided to mkfs.ext4 is the “mapped” device, which allows dm-crypt to intercept the calls to the file system and perform the encryption behind the scenes in a manner that is transparent to calling applications. At this point, the file system is created and can be mounted as follows:

    # mount -t ext4 /dev/mapper/backup /mnt

Now that we have an encrypted file system, let’s synchronize our root file system over to the /mnt file system with the following command:

    rsync -aAXv --delete --exclude=/dev/*        \
                         --exclude=/proc/*       \
                         --exclude=/sys/*        \
                         --exclude=/tmp/*        \
                         --exclude=/run/*        \
                         --exclude=/mnt/*        \
                         --exclude=/backup/*     \
                         --exclude=/lost+found/* \
                         --exclude="Downloads"   \
                         --exclude=.cache        \
                         --exclude=/shared/*     / /mnt

This will take a while initially, but in subsequent runs, rsync will only ship the differences and will be much quicker. It will keep the two file systems completely in sync quite efficiently.

You can find a more complete version of the backup script [here](https://github.com/aweeraman/arch_linux/blob/master/scripts/backup).

Once the backup is complete, the backup filesystem can be unmounted as follows:

    # umount /mnt
    # cryptsetup close backup

Any backup strategy is incomplete without confirmation that restoration is tested to be possible. For this, create a bootable USB or SD card with your favorite distro or rescue disk, and boot into it. Once booted, attempt to mount both the backup storage disk and the file system to restore to and run the following command, tweaking it to your environment as necessary:

    # rsync -aAXv --dry-run --delete --exclude="lost+found" --exclude="boot" --exclude="var" /backup/ /source/

For this I have included the dry-run argument, which prevents any actual changes to the file system. The changes to be performed will be output to the console, and if it includes the files that you’re looking to restore from the backup and you’re happy with what you see, you can re-run the command without the dry-run argument and let rsync synchronize the two file systems. You can also choose to cherry pick what you need from the backup file system if you just want to restore a specific file.

This is simple enough, and helps me achieve the three objectives that I laid out initially.
