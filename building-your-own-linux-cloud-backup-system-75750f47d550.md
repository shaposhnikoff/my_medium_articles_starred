Unknown markup type 10 { type: [33m10[39m, start: [33m178[39m, end: [33m193[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m338[39m, end: [33m352[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m130[39m, end: [33m134[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m174[39m, end: [33m195[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m46[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m129[39m, end: [33m142[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m34[39m, end: [33m83[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m113[39m, end: [33m419[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m0[39m, end: [33m186[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m51[39m, end: [33m53[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m199[39m, end: [33m201[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m209[39m, end: [33m213[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m211[39m, end: [33m219[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m272[39m, end: [33m280[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m31[39m, end: [33m38[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m142[39m, end: [33m158[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m254[39m, end: [33m268[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m285[39m, end: [33m290[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m64[39m, end: [33m100[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m226[39m, end: [33m243[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m24[39m, end: [33m37[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m46[39m, end: [33m50[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m154[39m, end: [33m160[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m37[39m, end: [33m41[39m }

# Building Your Own Linux Cloud Backup System

Automatic, secure, and cheap! A complete guide from start to finish.

![](https://cdn-images-1.medium.com/max/12032/1*m2I-UXbUMJw6iZH8RY-foA.jpeg)

Photos of your loved ones. Important documents. That screenplay that you’re totally going to finish some day. We all know we should be **backing up our important digital assets**, but knowing that and actually doing it are two different things.

This guide will show you how to set up a **completely automatic and encrypted** cloud backup system that costs about 50¢/month per 100GB and can backup as many desktops, laptops and servers as you want.

The first few sections cover the “why” and the “what” of this DIY approach; you can skip down to “Step 1” if you just want to get started. I’ve also created a [GitHub repo](https://github.com/markormesher/example-backup-scripts) with examples of the scripts we’ll create.

## Why Build Your Own?

![](https://cdn-images-1.medium.com/max/4000/1*AdUIhheAo_7oEl_GgXWnkg.jpeg)

Because you can. Because you’ll learn something. Because you want to guarantee security. Because it’s cheaper. All perfectly valid reasons. For me it was a mix of everything, but mostly I wanted to know exactly what was happening when I backed up something important.

## What Do You Need?

* **BorgBackup** (Borg for short) to create, deduplicate, compress and encrypt our backups.

* **Rclone** to sync your backups to the cloud.

* **Backblaze B2** for super-cheap and reliable cloud storage.

We’re using **Borg** in this guide because it is, simply put, freaking awesome. This wonderful piece of open source goodness will split your backed up files into chunks and only store the ones that are different, so you can run backups as often as you like without worrying about wasting space with hundreds of copies of your files. On top of that it also handles encrypting your files and compressing them, making it an ideal tool for managing backups that will be synced to a cloud storage system.

**Rclone** is another brilliant bit of open source magic that allows you to sync almost anything to almost anything else with no fuss. We’ll be syncing local files to Backblaze B2, but as of May 2020 it can also handle nearly 50 storage systems including Wasabi, Amazon S3, Digital Ocean Spaces, Google Cloud Storage and Azure Blob Storage.

Finally, **Backblaze B2** will be our cloud storage provider for this guide. You could use any of the other systems that Rclone supports, but I’d recommend B2 for a couple of reasons:

* It’s really cheap — just $0.005 per GB at the time of writing, with your first 10GB free. The pricing model is also very simple: one flat rate per GB stored, per GB downloaded and per 1000 actions.

* Their generous daily allowances on free transactions and downloads mean one less thing to worry about. Even when seeding my initial several-hundred GB backup I didn’t get close to using the transaction allowance.

* They’re well-trusted, with [more than an exabyte](https://www.backblaze.com/blog/exabyte-unlocked/) of customer data in storage.

* The tooling is easy to use, even if you’ve never dealt with “bare bones” cloud storage before.

*Side note: I don’t get paid to promote any of these services, I just think they really are the best options!*

## Step 1: Setting up Borg

The first step is deciding where your local backup server is going to live. You want to pick somewhere with enough space and a good Internet connection. If you’re only going to backup one system it’s okay for it to live right there, but ideally it should live somewhere separate. A Raspberry Pi with an external hard drive makes a great backup server.

Now, lets install Borg:

    sudo apt update
    sudo apt install borgbackup

*The commands above assume Debian (or Raspbian, Ubuntu, etc), but you can find details for almost all Linux OSes in the [Borg documentation](https://borgbackup.readthedocs.io/en/stable/installation.html).*

Once Borg is installed we need to initialise a backup repository — this is where your backed up files will live. Throughout this guide we’re going to assume your backups live at /hdd/borg-repo0 but this could be anywhere with enough space. Make sure the directory exists before you run this command.

    borg init --encryption=repokey-blake2 /hdd/borg-repo0

You will be asked to create a passphrase (make it a good one) and then Borg will create the encryption key for you. You should store your passphrase and a copy of your key somewhere safe because **if you lose either of them your backups are gone forever. **You can read more about Borg encryption modes in their [documentation](https://borgbackup.readthedocs.io/en/stable/usage/init.html#encryption-modes); we’ll be using repokey-blake2 for its security and speed.

    # optionally add --paper for a print-friendly output
    borg key export /hdd/borg-repo0 ~/borg-key-backup.txt

That’s it! You now have a Borg repo on your hard drive that is ready to start accepting files.

## Step 2: Creating Backups

Each backup in Borg is called an “archive”. Creating a backup archive is super-simple: you just need to supply a name for the archive and a list of files to backup. You could one-liner this command, but for ease of reading it’s presented here as a short script:

    #! /usr/bin/env bash

    archive_name="$(hostname)-$(date -Iseconds | cut -d '+' -f 1)"

    borg_options="--stats --compression zlib"

    borg create ${borg_options} /hdd/borg-repo0::${archive_name} \
      <file/dir to backup> \
      <file/dir to backup> \
      <...>

Run this script and it will create a new backup archive containing all of the files you listed. The files will be compressed with zlib (you can pick other methods — just run borg help compression) and a summary of the backup will be printed at the end.

If you make a change to your backed up files and run the script again you will see Borg’s super-power in action:

    **# create a backup of ~/screenplay
    **root@84f4c21fb19b:~# ./create-backup.sh 
    Enter passphrase for key /hdd/borg-repo0: 
    --------------------------------------------------------------------
    Archive name: 84f4c21fb19b-2020-05-10T10:31:29
    Archive fingerprint: add814...
    Time (start): Sun, 2020-05-10 10:31:31
    Time (end):   Sun, 2020-05-10 10:31:31
    Duration: 0.09 seconds
    Number of files: 15
    Utilization of max. archive size: 0%
    --------------------------------------------------------------------
                    Original size  Compressed size   Deduplicated size
    This archive:         1.51 MB        404.48 kB           404.48 kB
    All archives:         1.51 MB        404.48 kB           404.48 kB

                           Unique chunks         Total chunks
    Chunk index:                      17                   17
    --------------------------------------------------------------------

    **# add another 100KB file to ~/screenplay**
    root@84f4c21fb19b:~# mv ~/drafts/chapter-16.md ~/screenplay/.

    **# create another backup of ~/screenplay**
    root@84f4c21fb19b:~# ./create-backup.sh 
    Enter passphrase for key /hdd/borg-repo0: 
    --------------------------------------------------------------------
    Archive name: 84f4c21fb19b-2020-05-10T10:33:13
    Archive fingerprint: 153fdd...
    Time (start): Sun, 2020-05-10 10:33:18
    Time (end):   Sun, 2020-05-10 10:33:18
    Duration: 0.02 seconds
    Number of files: 16
    Utilization of max. archive size: 0%
    --------------------------------------------------------------------
                   Original size  Compressed size   Deduplicated size
    This archive:        1.61 MB        431.49 kB            28.69 kB
    All archives:        3.11 MB        835.97 kB           433.17 kB

                           Unique chunks         Total chunks
    Chunk index:                      20                   35
    --------------------------------------------------------------------

Look at the sizes on the second backup. We took a backup of the entire folder, which is just over 1.6MB, but because most of it was backed up already Borg only had to store 28.68kB of new information. Exactly how Borg does this is [really clever](https://borgbackup.readthedocs.io/en/stable/internals.html), but that’s detail for another time; for now you can just throw files at Borg and it will happily do all the hard work for you.

## Step 3: Syncing Backups to the Cloud

We’ll do this in two stages: setting up storage space in Backblaze B2, then syncing your backups from Borg.

### Setting up Backblaze B2 Storage

This part is easy: head to the [Backblaze B2 sign-up page](https://www.backblaze.com/b2/sign-up.html) and create a free account. Hopefully this step doesn’t need explaining!

Once you’re logged in, go to “Buckets” and create a new storage bucket. Make sure the bucket is set to private and give it a unique name (names have to be *globally* unique, so prefixing with your username is a good start).

Now head to “App Keys” and create a new key. It will need read and write access to the bucket you just created. Save the key somewhere safe because it’ll only be displayed once.

That’s it! Backblaze B2 is ready to store your files.

### Syncing from Borg to Backblaze B2

First, we’ll install Rclone on our backup server:

    curl https://rclone.org/install.sh | sudo bash

*See [the documentation](https://rclone.org/install/) for alternative install methods.*

Now we need to tell Rclone about the Backblaze B2 bucket we just created, so it knows how to sync to it in the future. We’ll run rclone config to start the interactive setup tool and go through the prompts to configure a new remote location. **Bold** indicates fields you should complete.

    root@84f4c21fb19b:~# rclone config
    No remotes found - make a new one
    n) New remote
    s) Set configuration password
    q) Quit config
    n/s/q> **n**
    name> **b2**
    Type of storage to configure.
    Choose a number from below, or type in your own value
    <...many options will be listed...>
    ?? / Backblaze B2
       \ "b2"
    <...>
    Storage> **b2**
    Account ID or Application Key ID
    account> **your-application-key-id**
    Application Key
    key> **your-application-key**

    **<You may be prompted for more config options now, you can leave everything with its default value. Eventually you'll be asked to confirm your remote config, as below.>**

    --------------------
    [b2]
    type = b2
    account = your-application-key-id
    key = your-application-key
    --------------------
    y) Yes this is OK
    e) Edit this remote
    d) Delete this remote
    y/e/d> **y**

Rclone now has a remote location configured called b2 that it can sync files to, and syncing your backups to the cloud is as easy as running one command:

    rclone sync /hdd/borg-repo0 b2:<your bucket name>

This might take a while, depending on how much data you have backed up. The good news is that Borg and Rclone both work on *differences* in data, so subsequent backups will be much faster. You can add -P to the sync command to see a live progress meter.

## Step 4: Automation

### Running Regular Backups

Running your backups on a schedule is as easy as putting the commands and scripts into cron, but what about supplying the passphrase? Borg allows you to do this from an environment variable, so you can create a .secrets file to load before running your backup script. The .secrets file should only be readable by your user and should contain the following:

    export BORG_PASSPHRASE='**your passphrase**'

Now you can source that file before running your backup script and Borg will use the key from the environment variable. This a full example of a cron job that backs up every day at 8AM:

    0 8 * * *  bash -l -c "cd ~/backup-scripts; source .secrets; ./create-backup.sh"

Create another cron job for your Rclone sync command (which doesn’t need credentials) and you’re all set! Check out [this repository](https://github.com/markormesher/example-backup-scripts) for full examples of cron jobs and backup scripts.

### [Optional] Backing Up Multiple Machines

This one is easy — any machine that can reach your backup server over SSH can send backups there, you just need to import the backup key you saved a copy of earlier:

    borg key import username@host:/hdd/borg-repo0 ~/borg-key-backup.txt

Once your key is imported (add --paper if you exported it in that format) you can backup as normal, just update the backup script to use the remote location:

    # local location format:
    /hdd/borg-repo0::<archive name>

    # remote location format:
    username@host:/hdd/borg-repo0::<archive-name>

### [Optional] Pruning Old Backups

Borg is great at minimising disk space used for backups, so you *could* decide to keep everything forever. However, if you want to gradually drop old backups, Borg provides a highly-configurable way to do it:

    borg prune \
      --stats \
      --keep-daily 14 \
      --keep-weekly 4 \
      --keep-monthly 6 \
      --keep-yearly -1 \
      /hdd/borg-repo0

This configuration will keep one backup archive per day for the last 14 days, plus one per week for the last 4 weeks, etc. The last argument, --keep-yearly -1, tells Borg to keep one backup per year going back as far as possible. Obviously you can tweak the retention rules to suit your needs; the [documentation for the prune method](https://borgbackup.readthedocs.io/en/stable/usage/prune.html) is very comprehensive and explains exactly how Borg decides which archives to keep.

What if you’re backing up **multiple machines** and you want a different retention policy for each one? Borg can handle that too! In our backup script we gave each backup archive a name starting with the host name; that was so we could take advantage of the --prefix <...> argument to the prune command. When you specify a prefix it will only affect files starting with that string.

### [Optional] Keeping Backblaze B2 Tidy

If you’re backing up and syncing to the cloud regularly (which you should be!) you will eventually run into the occasional failed upload. That’s fine — Rclone will just try again next time — but Backblaze B2 might keep holding on to those partially uploaded failed files and charging you to store them.

Fortunately, there’s an easy way to deal with this. You can run rclone cleanup b2:<your bucket name> to delete partially uploaded files that are more than 1 day old; a good practise is to run this after your scheduled runs of rclone sync <...>.

### [Optional] Tips for Running Rclone on a Raspberry Pi

Raspberry Pis are brilliant little bits of kit, but they aren’t the most powerful machines around. If you’re using one as your backup server (like I am), there are two things you will want to tweak to get the best out of Rclone:

First, add the argument --transfers=1 to your sync command. This will force Rclone to send one file at a time rather than the default of 4, which prevents it from overwhelming your Pi (especially the weaker models).

Second, create a folder on your large storage device that can be used a temporary space (this should NOT be inside your Borg directory) and set it as the TMPDIR environment variable just before running the sync, like this:

    export TMPDIR=/hdd/tmp
    rclone sync <...>

Rclone’s default behaviour is to use /tmp for temporary storage, but if that’s on your Pi’s SD card you may find it too slow and/or too small to work effectively.

## That’s it!

You’re done, you now have automated encrypted backups stored in the cloud — well done! If you want to take things further, here are some suggestions:

* Check out [this repository](https://github.com/markormesher/example-backup-scripts) where I have created example scripts for everything covered here.

* Dig into the [internals](https://borgbackup.readthedocs.io/en/stable/internals.html) of how Borg does its magic — it really is interesting stuff.

* Set spending limits on your Backblaze B2 account, just in case you accidentally try to store more than you intended.

## Limitations

One important caveat with what we’ve built here is that we’re not backing up directly to the cloud in one move — we’re backing up to a hard drive on our backup host, then regularly syncing *that* into the cloud. For 99% of users this is completely fine, but there are a few drawbacks:

* You backup hard drive needs to be at least as big as everything you plan to backup. With multi-TB external drives readily available this should be fine for most users, but it may be a limiting factor for a few use cases.

* Your backup machine needs to be accessible from wherever you might want to interact with your backups, both adding or recovering files. Again, for most use cases this should be fine, as long as you set it up to be accessible over your network or over the Internet (and secure it appropriately!), but if you’re on the road a lot it could be an issue.

* If your backup machine or its storage dies, you’ll need to clone your whole backup down to a new host/drive before you can carry on backing up to it, or recovering files from it. This is probably the biggest weakness of this system.

So, how do you overcome these limitations?

Simple: **skip the middleman and backup to the cloud directly**. Doing this comes with a few complications and concerns, which is why it’s not covered here, but watch this space for part 2 where we’ll do exactly that.
