
# 100 Days of DevOps — Day 68-Introduction to Systemd

Welcome to Day 68 of 100 Days of DevOps, Focus for today is Introduction to Systemd

*If you ask me the most important new feature introduced in RHEL7/Centos7 I would say that is Systemd*
> *Units in Systemd*

* *Unit is the basic building block in Systemd*

* *These are the things you can manage using Systemd*

    *# It shows the unit types*

    *# systemctl -t help*

    *Available unit types:*

    *service*

    *socket*

    *busname*

    *target --> group of services that belong together*

    *snapshot*

    *device*

    *mount --> mounts are preferably done using systemd rather /etc/fstab*

    *automount*

    *swap*

    *timer*

    *path*

    *slice*

    *scope*

* *Behind every unit, there is a file and they are located in /usr/lib/systemd/system*

    *# cd /usr/lib/systemd/system
    # ls*

    *arp-ethers.service                      fstrim.timer                       nss-user-lookup.target             runlevel5.target                               systemd-localed.service*

    *auditd.service                          getty-pre.target                   paths.target                       runlevel5.target.wants                         systemd-logind.service*

    *auth-rpcgss-module.service              getty@.service                     polkit.service                     runlevel6.target                               systemd-machined.service*

    *autovt@.service                         getty.target                       postfix.service                    selinux-policy-migrate-local-changes@.service  systemd-machine-id-commit.service*

    *basic.target                            graphical.target                   poweroff.target                    serial-getty@.service                          systemd-modules-load.service*

    *basic.target.wants                      graphical.target.wants             poweroff.target.wants              shutdown.target                                systemd-nspawn@.service*

    *blk-availability.service                gssproxy.service                   printer.target                     shutdown.target.wants                          systemd-poweroff.service*

    *bluetooth.target                        halt-local.service                 proc-fs-nfsd.mount                 sigpwr.target                                  systemd-quotacheck.service*

    *brandbot.path                           halt.target                        proc-sys-fs-binfmt_misc.automount  sleep.target                                   systemd-random-seed.service*

    *brandbot.service                        hibernate.target                   proc-sys-fs-binfmt_misc.mount      -.slice                                        systemd-readahead-collect.service*

    *chrony-dnssrv@.service                  htcacheclean.service               qemu-guest-agent.service           slices.target                                  systemd-readahead-done.service*

    *chrony-dnssrv@.timer                    httpd.service                      quotaon.service                    smartcard.target                               systemd-readahead-done.timer*

    *chronyd.service                         hybrid-sleep.target                rc-local.service                   sockets.target                                 systemd-readahead-drop.service*

    *chrony-wait.service                     initrd-cleanup.service             rdisc.service                      sockets.target.wants                           systemd-readahead-replay.service*

    *cloud-config.service                    initrd-fs.target                   reboot.target                      sound.target                                   systemd-reboot.service*

    *cloud-config.target                     initrd-parse-etc.service           reboot.target.wants                sshd-keygen.service                            systemd-remount-fs.service*

    *cloud-final.service                     initrd-root-fs.target              remote-cryptsetup.target           sshd.service                                   systemd-rfkill@.service*

    *cloud-init-local.service                initrd-switch-root.service         remote-fs-pre.target               sshd@.service                                  systemd-shutdownd.service*

    *cloud-init.service                      initrd-switch-root.target          remote-fs.target                   sshd.socket                                    systemd-shutdownd.socket*

    *console-getty.service                   initrd.target                      rescue.service                     suspend.target                                 systemd-suspend.service*

    *console-shell.service                   initrd.target.wants                rescue.target                      swap.target                                    systemd-sysctl.service*

    *container-getty@.service                initrd-udevadm-cleanup-db.service  rescue.target.wants                sys-fs-fuse-connections.mount                  systemd-timedated.service*

    *cpupower.service                        irqbalance.service                 rhel-autorelabel-mark.service      sysinit.target                                 systemd-tmpfiles-clean.service*

    *crond.service                           kdump.service                      rhel-autorelabel.service           sysinit.target.wants                           systemd-tmpfiles-clean.timer*

    *cryptsetup-pre.target                   kexec.target                       rhel-configure.service             sys-kernel-config.mount                        systemd-tmpfiles-setup-dev.service*

    *cryptsetup.target                       kmod-static-nodes.service          rhel-dmesg.service                 sys-kernel-debug.mount                         systemd-tmpfiles-setup.service*

    *ctrl-alt-del.target                     local-fs-pre.target                rhel-domainname.service            syslog.socket                                  systemd-udevd-control.socket*

    *dbus-org.freedesktop.hostname1.service  local-fs.target                    rhel-import-state.service          syslog.target.wants                            systemd-udevd-kernel.socket*

    *dbus-org.freedesktop.import1.service    local-fs.target.wants              rhel-loadmodules.service           sysstat.service                                systemd-udevd.service*

    *dbus-org.freedesktop.locale1.service    machine.slice                      rhel-readonly.service              systemd-ask-password-console.path              systemd-udev-settle.service*

    *dbus-org.freedesktop.login1.service     machines.target                    rpcbind.service                    systemd-ask-password-console.service           systemd-udev-trigger.service*

    *dbus-org.freedesktop.machine1.service   messagebus.service                 rpcbind.socket                     systemd-ask-password-wall.path                 systemd-update-done.service*

    *dbus-org.freedesktop.timedate1.service  microcode.service                  rpcbind.target                     systemd-ask-password-wall.service              systemd-update-utmp-runlevel.service*

    *dbus.service                            multi-user.target                  rpc-gssd.service                   systemd-backlight@.service                     systemd-update-utmp.service*

    *dbus.socket                             multi-user.target.wants            rpcgssd.service                    systemd-binfmt.service                         systemd-user-sessions.service*

    *dbus.target.wants                       network-online.target              rpcidmapd.service                  systemd-bootchart.service                      systemd-vconsole-setup.service*

    *debug-shell.service                     network-pre.target                 rpc_pipefs.target                  systemd-firstboot.service                      system.slice*

    *default.target                          network.target                     rpc-rquotad.service                systemd-fsck-root.service                      system-update.target*

    *default.target.wants                    nfs-blkmap.service                 rpc-statd-notify.service           systemd-fsck@.service                          teamd@.service*

    *dev-hugepages.mount                     nfs-client.target                  rpc-statd.service                  systemd-halt.service                           timers.target*

    *dev-mqueue.mount                        nfs-config.service                 rsyncd.service                     systemd-hibernate-resume@.service              timers.target.wants*

    *dracut-cmdline.service                  nfs-idmapd.service                 rsyncd@.service                    systemd-hibernate.service                      time-sync.target*

    *dracut-initqueue.service                nfs-idmap.service                  rsyncd.socket                      systemd-hostnamed.service                      tmp.mount*

    *dracut-mount.service                    nfs-lock.service                   rsyslog.service                    systemd-hwdb-update.service                    tuned.service*

    *dracut-pre-mount.service                nfslock.service                    runlevel0.target                   systemd-hybrid-sleep.service                   umount.target*

    *dracut-pre-pivot.service                nfs-mountd.service                 runlevel1.target                   systemd-importd.service                        user.slice*

    *dracut-pre-trigger.service              nfs-rquotad.service                runlevel1.target.wants             systemd-initctl.service                        var-lib-nfs-rpc_pipefs.mount*

    *dracut-pre-udev.service                 nfs-secure.service                 runlevel2.target                   systemd-initctl.socket                         vsftpd.service*

    *dracut-shutdown.service                 nfs-server.service                 runlevel2.target.wants             systemd-journal-catalog-update.service         vsftpd@.service*

    *emergency.service                       nfs.service                        runlevel3.target                   systemd-journald.service                       vsftpd.target*

    *emergency.target                        nfs-utils.service                  runlevel3.target.wants             systemd-journald.socket                        wpa_supplicant.service*

    *final.target                            nginx.service                      runlevel4.target                   systemd-journal-flush.service*

    *fstrim.service                          nss-lookup.target                  runlevel4.target.wants             systemd-kexec.service*

* *These are all the files, installed with the installation of server*

* *Let’s take a look at a simple example*

    *# cat vsftpd.service*

    *[Unit]*

    *Description=Vsftpd ftp daemon*

    *After=network.target *

    *[Service]*

    *Type=forking*

    ***ExecStart=/usr/sbin/vsftpd /etc/vsftpd/vsftpd.conf** <---------*

    *[Install]*

    *WantedBy=multi-user.target*

* *Don't worry about all the other parameters and pay attention to how we are starting vsftpd service*

* *If as an administrator you want to make changes to the unit file, go to /etc/systemd/system*

    *# cd /etc/systemd/system
    # ls*

    *basic.target.wants  default.target.wants                                     getty.target.wants     multi-user.target.wants  sockets.target.wants  system-update.target.wants*

    *default.target      dev-virtio\x2dports-org.qemu.guest_agent.0.device.wants  local-fs.target.wants  remote-fs.target.wants   sysinit.target.wants  tmp.mount*
> *Systemd Targets*

* *Targets are basically a collection of Systemd units*

* *If you want to look at the default targets*

    *# cd /usr/lib/systemd/system
    # ls *target*

    *basic.target           emergency.target  hybrid-sleep.target        local-fs.target        nss-lookup.target         remote-fs-pre.target  runlevel2.target  sleep.target      sysinit.target*

    *bluetooth.target       final.target      initrd-fs.target           machines.target        nss-user-lookup.target    remote-fs.target      runlevel3.target  slices.target     system-update.target*

    *cloud-config.target    getty-pre.target  initrd-root-fs.target      multi-user.target      paths.target              rescue.target         runlevel4.target  smartcard.target  timers.target*

    *cryptsetup-pre.target  getty.target      initrd-switch-root.target  network-online.target  poweroff.target           rpcbind.target        runlevel5.target  sockets.target    time-sync.target*

    *cryptsetup.target      graphical.target  initrd.target              network-pre.target     printer.target            rpc_pipefs.target     runlevel6.target  sound.target      umount.target*

    *ctrl-alt-del.target    halt.target       kexec.target               network.target         reboot.target             runlevel0.target      shutdown.target   suspend.target    vsftpd.target*

    *default.target         hibernate.target  local-fs-pre.target        nfs-client.target      remote-cryptsetup.target  runlevel1.target      sigpwr.target     swap.target*

* *If you open any one of these targets, you will see runlevel1.target requires other things which are group together(**Requires=sysinit.target rescue.service)***

    *# cat runlevel1.target*

    *#  This file is part of systemd.*

    *#*

    *#  systemd is free software; you can redistribute it and/or modify it*

    *#  under the terms of the GNU Lesser General Public License as published by*

    *#  the Free Software Foundation; either version 2.1 of the License, or*

    *#  (at your option) any later version.*

    *[Unit]*

    *Description=Rescue Mode*

    *Documentation=man:systemd.special(7)*

    ***Requires=sysinit.target rescue.service***

    *After=sysinit.target rescue.service*

    *AllowIsolate=yes*

    *[Install]*

    *Alias=kbrequest.target*

* *If you dig further you will see the comparison with runlevels*

    *# ls -ld runlevel**

    *lrwxrwxrwx. 1 root root 15 Jan 28 12:53 runlevel0.target -> poweroff.target*

    *lrwxrwxrwx. 1 root root 13 Jan 28 12:53 runlevel1.target -> rescue.target*

    *drwxr-xr-x. 2 root root 50 Jan 28 12:53 runlevel1.target.wants*

    *lrwxrwxrwx. 1 root root 17 Jan 28 12:53 runlevel2.target -> multi-user.target*

    *drwxr-xr-x. 2 root root 50 Jan 28 12:53 runlevel2.target.wants*

    *lrwxrwxrwx. 1 root root 17 Jan 28 12:53 runlevel3.target -> multi-user.target*

    *drwxr-xr-x. 2 root root 50 Jan 28 12:53 runlevel3.target.wants*

    *lrwxrwxrwx. 1 root root 17 Jan 28 12:53 runlevel4.target -> multi-user.target*

    *drwxr-xr-x. 2 root root 50 Jan 28 12:53 runlevel4.target.wants*

    *lrwxrwxrwx. 1 root root 16 Jan 28 12:53 runlevel5.target -> graphical.target*

    *drwxr-xr-x. 2 root root 50 Jan 28 12:53 runlevel5.target.wants*

    *lrwxrwxrwx. 1 root root 13 Jan 28 12:53 runlevel6.target -> reboot.target*

* *eg: runlevel0.target is for poweroff and runlevel5.target is for graphical mode etc..*

* *Now when working with targets the important point to remember that targets will depend on each other*

    *# systemctl list-dependencies graphical.target |grep -i target*

    *graphical.**target***

    *● └─multi-user.**target***

    *●   ├─basic.**target***

    *●   │ ├─selinux-policy-migrate-local-changes@**target**ed.service*

    *●   │ ├─paths.**target***

    *●   │ ├─slices.**target***

    *●   │ ├─sockets.**target***

    *●   │ ├─sysinit.**target***

    *●   │ │ ├─cryptsetup.**target***

    *●   │ │ ├─local-fs.**target***

    *●   │ │ └─swap.**target***

    *●   │ └─timers.**target***

    *●   ├─getty.**target***

    *●   ├─nfs-client.**target***

    *●   │ └─remote-fs-pre.**target***

    *●   └─remote-fs.**target***

    *●     └─nfs-client.**target***

    *●       └─remote-fs-pre.**target***

* *To list out all the targets in the system*

    *# systemctl list-units --type=target --all*

    *UNIT                   LOAD      ACTIVE   SUB    DESCRIPTION*

    *basic.target           loaded    active   active Basic System*

    *cloud-config.target    loaded    active   active Cloud-config availability*

    *cryptsetup.target      loaded    active   active Local Encrypted Volumes*

    *emergency.target       loaded    inactive dead   Emergency Mode*

    *final.target           loaded    inactive dead   Final Step*

    ***●** firewalld.target       **not-found** inactive dead   firewalld.target*

    *getty-pre.target       loaded    inactive dead   Login Prompts (Pre)*

    *getty.target           loaded    active   active Login Prompts*

    *graphical.target       loaded    inactive dead   Graphical Interface*

    *local-fs-pre.target    loaded    active   active Local File Systems (Pre)*

    *local-fs.target        loaded    active   active Local File Systems*

    *multi-user.target      loaded    active   active Multi-User System*

    *network-online.target  loaded    active   active Network is Online*

    *network-pre.target     loaded    active   active Network (Pre)*

    *network.target         loaded    active   active Network*

    *nfs-client.target      loaded    active   active NFS client services*

    *nss-lookup.target      loaded    inactive dead   Host and Network Name Lookups*

    *nss-user-lookup.target loaded    inactive dead   User and Group Name Lookups*

    *paths.target           loaded    active   active Paths*

    *remote-fs-pre.target   loaded    active   active Remote File Systems (Pre)*

    *remote-fs.target       loaded    active   active Remote File Systems*

    *rescue.target          loaded    inactive dead   Rescue Mode*

    *rpc_pipefs.target      loaded    active   active rpc_pipefs.target*

    *rpcbind.target         loaded    active   active RPC Port Mapper*

    *shutdown.target        loaded    inactive dead   Shutdown*

    *slices.target          loaded    active   active Slices*

    *sockets.target         loaded    active   active Sockets*

    *swap.target            loaded    active   active Swap*

    *sysinit.target         loaded    active   active System Initialization*

    ***●** syslog.target          **not-found** inactive dead   syslog.target*

    *time-sync.target       loaded    inactive dead   System Time Synchronized*

    *timers.target          loaded    active   active Timers*

    *umount.target          loaded    inactive dead   Unmount All Filesystems*

    *LOAD   = Reflects whether the unit definition was properly loaded.*

    *ACTIVE = The high-level unit activation state, i.e. generalization of SUB.*

    *SUB    = The low-level unit activation state, values depend on unit type.*

    ***33 loaded units listed.***

    *To show all installed unit files use 'systemctl list-unit-files'.*

* *To find out the default target*

    *# systemctl get-default*

    *graphical.target*

* *To change it some other target*

    *# systemctl set-default multi-user.target*

    *Removed symlink /etc/systemd/system/default.target.*

    *Created symlink from /etc/systemd/system/default.target to /usr/lib/systemd/system/multi-user.target.*

* *To get the list of all the services via systemd*

    *# systemctl --type=service*

    *UNIT                               LOAD   ACTIVE SUB     DESCRIPTION*

    *auditd.service                     loaded active running Security Auditing Service*

    *chronyd.service                    loaded active running NTP client/server*

    *cloud-config.service               loaded active exited  Apply the settings specified in cloud-config*

    *cloud-final.service                loaded active exited  Execute cloud user/final scripts*

    *cloud-init-local.service           loaded active exited  Initial cloud-init job (pre-networking)*

    *cloud-init.service                 loaded active exited  Initial cloud-init job (metadata service crawler)*

    *crond.service                      loaded active running Command Scheduler*

    *dbus.service                       loaded active running D-Bus System Message Bus*

    *getty@tty1.service                 loaded active running Getty on tty1*

    *gssproxy.service                   loaded active running GSSAPI Proxy Daemon*

    *httpd.service                      loaded active running The Apache HTTP Server*

    *jenkins.service                    loaded active running LSB: Jenkins Automation Server*

    ***●** **kdump.service                     ** loaded **failed failed ** Crash recovery kernel arming*

    *kmod-static-nodes.service          loaded active exited  Create list of required static device nodes for the current kernel*

    *network.service                    loaded active running LSB: Bring up/down networking*

    *polkit.service                     loaded active running Authorization Manager*

    *postfix.service                    loaded active running Postfix Mail Transport Agent*

    *rhel-dmesg.service                 loaded active exited  Dump dmesg to /var/log/dmesg*

    *rhel-domainname.service            loaded active exited  Read and set NIS domainname from /etc/sysconfig/network*

    *rhel-import-state.service          loaded active exited  Import network configuration from initramfs*

    *rhel-readonly.service              loaded active exited  Configure read-only root support*

    *rpcbind.service                    loaded active running RPC bind service*

    *rsyslog.service                    loaded active running System Logging Service*

    *serial-getty@ttyS0.service         loaded active running Serial Getty on ttyS0*

    *sshd.service                       loaded active running OpenSSH server daemon*

    *systemd-journal-flush.service      loaded active exited  Flush Journal to Persistent Storage*

    *systemd-journald.service           loaded active running Journal Service*

    *systemd-logind.service             loaded active running Login Service*

    *systemd-random-seed.service        loaded active exited  Load/Save Random Seed*

    *systemd-remount-fs.service         loaded active exited  Remount Root and Kernel File Systems*

    *systemd-sysctl.service             loaded active exited  Apply Kernel Variables*

    *systemd-tmpfiles-setup-dev.service loaded active exited  Create Static Device Nodes in /dev*

    *systemd-tmpfiles-setup.service     loaded active exited  Create Volatile Files and Directories*

    *systemd-udev-trigger.service       loaded active exited  udev Coldplug all Devices*

    *systemd-udevd.service              loaded active running udev Kernel Device Manager*

    *systemd-update-utmp.service        loaded active exited  Update UTMP about System Boot/Shutdown*

    *systemd-user-sessions.service      loaded active exited  Permit User Sessions*

    *systemd-vconsole-setup.service     loaded active exited  Setup Virtual Console*

    *tuned.service                      loaded active running Dynamic System Tuning Daemon*

    *LOAD   = Reflects whether the unit definition was properly loaded.*

    *ACTIVE = The high-level unit activation state, i.e. generalization of SUB.*

    *SUB    = The low-level unit activation state, values depend on unit type.*

    ***39 loaded units listed.** Pass --all to see loaded but inactive units, too.*

    *To show all installed unit files use 'systemctl list-unit-files'.*

* *If you are looking for a concise view(this is equivalent to chkconfig — list command)*

    *# systemctl list-unit-files --type=service*

    *UNIT FILE                                     STATE*

    *arp-ethers.service                            **disabled***

    *auditd.service                                **enabled***

    *auth-rpcgss-module.service                    static*

    *autovt@.service                               **enabled***

    *blk-availability.service                      **disabled***

    *brandbot.service                              static*

    *chrony-dnssrv@.service                        static*

    *chrony-wait.service                           **disabled***

    *chronyd.service                               **enabled***

    *cloud-config.service                          **enabled***

    *cloud-final.service                           **enabled***

    *cloud-init-local.service                      **enabled***

    *cloud-init.service                            **enabled***

    *console-getty.service                         **disabled***

    *console-shell.service                         **disabled***

    *container-getty@.service                      static*

    *cpupower.service                              **disabled***

    *crond.service                                 **enabled***

    *dbus-org.freedesktop.hostname1.service        static*

    *dbus-org.freedesktop.import1.service          static*

    *dbus-org.freedesktop.locale1.service          static*

    *dbus-org.freedesktop.login1.service           static*

    *dbus-org.freedesktop.machine1.service         static*

    *dbus-org.freedesktop.timedate1.service        static*

    *dbus.service                                  static*

    *debug-shell.service                           **disabled***

    *dracut-cmdline.service                        static*

    *dracut-initqueue.service                      static*

    *dracut-mount.service                          static*

    *dracut-pre-mount.service                      static*

    *dracut-pre-pivot.service                      static*

    *dracut-pre-trigger.service                    static*

    *dracut-pre-udev.service                       static*

    *dracut-shutdown.service                       static*

    *emergency.service                             static*

    *fstrim.service                                static*

    *getty@.service                                **enabled***

    *gssproxy.service                              **disabled***

    *halt-local.service                            static*

    *htcacheclean.service                          static*

    *httpd.service                                 **enabled***

    *initrd-cleanup.service                        static*

    *initrd-parse-etc.service                      static*

    *initrd-switch-root.service                    static*

    *initrd-udevadm-cleanup-db.service             static*

    *irqbalance.service                            **enabled***

    *kdump.service                                 **enabled***

    *kmod-static-nodes.service                     static*

    *messagebus.service                            static*

    *microcode.service                             **enabled***

    *nfs-blkmap.service                            **disabled***

    *nfs-config.service                            static*

    *nfs-idmap.service                             static*

    *nfs-idmapd.service                            static*

    *nfs-lock.service                              static*

    *nfs-mountd.service                            static*

    *nfs-rquotad.service                           **disabled***

    *nfs-secure.service                            static*

    *nfs-server.service                            **disabled***

    *nfs-utils.service                             static*

* *To check the status of any service*

    *# systemctl status httpd*

    *● httpd.service - The Apache HTTP Server*

    *Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)*

    *Active: inactive (dead) since Thu 2019-04-18 22:01:42 PDT; 2s ago*

    *Docs: man:httpd(8)*

    *man:apachectl(8)*

    *Process: 15963 ExecStop=/bin/kill -WINCH ${MAINPID} (code=exited, status=0/SUCCESS)*

    *Process: 410 ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND (code=exited, status=0/SUCCESS)*

    *Main PID: 410 (code=exited, status=0/SUCCESS)*

    *Status: "Total requests: 137; Current requests/sec: 0; Current traffic:   0 B/sec"*

    *Apr 18 22:01:41 ip-172-31-31-68.us-west-2.compute.internal systemd[1]: Stopping The Apache HTTP Server...*

    *Apr 18 22:01:42 ip-172-31-31-68.us-west-2.compute.internal systemd[1]: Stopped The Apache HTTP Server.*

* *To start/stop any service*

    *# systemctl start httpd
    # systemctl stop httpd*

* *To make sure it enable/disable at reboot*

    *# systemctl enable httpd
    # systemctl disable httpd*

* *To list out the dependency of a particular service*

    *# systemctl list-dependencies httpd*

    *httpd.service*

    ***●** ├─-.mount*

    ***●** ├─system.slice*

    ***●** └─basic.target*

    ***●**   ├─microcode.service*

    ***●**   ├─rhel-dmesg.service*

    ***●**   ├─selinux-policy-migrate-local-changes@targeted.service*

    ***●**   ├─paths.target*

    ***●**   ├─slices.target*

    ***●**   │ ├─-.slice*

    ***●**   │ └─system.slice*

    ***●**   ├─sockets.target*

    ***●**   │ ├─dbus.socket*

    ***●**   │ ├─rpcbind.socket*

    ***●**   │ ├─systemd-initctl.socket*

    ***●**   │ ├─systemd-journald.socket*

    ***●**   │ ├─systemd-shutdownd.socket*

    ***●**   │ ├─systemd-udevd-control.socket*

    ***●**   │ └─systemd-udevd-kernel.socket*

    ***●**   ├─sysinit.target*

    ***●**   │ ├─dev-hugepages.mount*

    ***●**   │ ├─dev-mqueue.mount*

    ***●**   │ ├─kmod-static-nodes.service*

    ***●**   │ ├─proc-sys-fs-binfmt_misc.automount*

    ***●**   │ ├─rhel-autorelabel.service*

    ***●**   │ ├─rhel-domainname.service*

    ***●**   │ ├─rhel-import-state.service*

    ***●**   │ ├─rhel-loadmodules.service*

    ***●**   │ ├─sys-fs-fuse-connections.mount*

    ***●**   │ ├─sys-kernel-config.mount*

    ***●**   │ ├─sys-kernel-debug.mount*

    ***●**   │ ├─systemd-ask-password-console.path*

    ***●**   │ ├─systemd-binfmt.service*

    ***●**   │ ├─systemd-firstboot.service*

    ***●**   │ ├─systemd-hwdb-update.service*

    ***●**   │ ├─systemd-journal-catalog-update.service*

    ***●**   │ ├─systemd-journal-flush.service*

    ***●**   │ ├─systemd-journald.service*

    ***●**   │ ├─systemd-machine-id-commit.service*

    ***●**   │ ├─systemd-modules-load.service*

    ***●**   │ ├─systemd-random-seed.service*

    ***●**   │ ├─systemd-sysctl.service*

    ***●**   │ ├─systemd-tmpfiles-setup-dev.service*

    ***●**   │ ├─systemd-tmpfiles-setup.service*

    ***●**   │ ├─systemd-udev-trigger.service*

    ***●**   │ ├─systemd-udevd.service*

    ***●**   │ ├─systemd-update-done.service*

    ***●**   │ ├─systemd-update-utmp.service*

    ***●**   │ ├─systemd-vconsole-setup.service*

    ***●**   │ ├─cryptsetup.target*

    ***●**   │ ├─local-fs.target*

    ***●**   │ │ ├─-.mount*

    ***●**   │ │ ├─rhel-readonly.service*

    ***●**   │ │ └─systemd-remount-fs.service*

    ***●**   │ └─swap.target*

    ***●**   └─timers.target*

    ***●**     └─systemd-tmpfiles-clean.timer*

* *Let discuss the important feature called Cgroups*

* *Cgroups allow you to allocate resources — such as CPU time, system memory, network bandwidth, or combinations of these resources — among user-defined groups of tasks (processes) running on a system*

    *# mount |grep -i cgroup*

    *tmpfs on /sys/fs/**cgroup** type tmpfs (ro,nosuid,nodev,noexec,mode=755)*

    ***cgroup** on /sys/fs/**cgroup**/systemd type **cgroup** (rw,nosuid,nodev,noexec,relatime,xattr,release_agent=/usr/lib/systemd/systemd-**cgroup**s-agent,name=systemd)*

    ***cgroup** on /sys/fs/**cgroup**/net_cls,net_prio type **cgroup** (rw,nosuid,nodev,noexec,relatime,net_prio,net_cls)*

    ***cgroup** on /sys/fs/**cgroup**/blkio type **cgroup** (rw,nosuid,nodev,noexec,relatime,blkio)*

    ***cgroup** on /sys/fs/**cgroup**/cpuset type **cgroup** (rw,nosuid,nodev,noexec,relatime,cpuset)*

    ***cgroup** on /sys/fs/**cgroup**/cpu,cpuacct type **cgroup** (rw,nosuid,nodev,noexec,relatime,cpuacct,cpu)*

    ***cgroup** on /sys/fs/**cgroup**/hugetlb type **cgroup** (rw,nosuid,nodev,noexec,relatime,hugetlb)*

    ***cgroup** on /sys/fs/**cgroup**/memory type **cgroup** (rw,nosuid,nodev,noexec,relatime,memory)*

    ***cgroup** on /sys/fs/**cgroup**/freezer type **cgroup** (rw,nosuid,nodev,noexec,relatime,freezer)*

    ***cgroup** on /sys/fs/**cgroup**/devices type **cgroup** (rw,nosuid,nodev,noexec,relatime,devices)*

    ***cgroup** on /sys/fs/**cgroup**/pids type **cgroup** (rw,nosuid,nodev,noexec,relatime,pids)*

    ***cgroup** on /sys/fs/**cgroup**/perf_event type **cgroup** (rw,nosuid,nodev,noexec,relatime,perf_event)*

* *Now in our use case, I want to restrict Apache to only use 500MB of memory*

    *cd /etc/systemd/system
    # Create a file
    # cat httpd.service*

    *.include /usr/lib/systemd/system/httpd.service*

    *[Service]*

    *MemoryLimit=500M*

    *# As we made changes to Unit file this is required*

    *# systemctl daemon-reload*

    *# systemctl restart httpd*

    *# systemctl status httpd*

    ***●** httpd.service - The Apache HTTP Server*

    *Loaded: loaded (/etc/systemd/system/httpd.service; enabled; vendor preset: disabled)*

    *Active: **active (running)** since Thu 2019-04-18 22:15:06 PDT; 2s ago*

    *Docs: man:httpd(8)*

    *man:apachectl(8)*

    *Process: 16580 ExecStop=/bin/kill -WINCH ${MAINPID} (code=exited, status=0/SUCCESS)*

    *Main PID: 16585 (httpd)*

    *Status: "Processing requests..."*

    ***Memory: 2.9M (limit: 500.0M)***

    *CGroup: /system.slice/httpd.service*

    *├─16585 /usr/sbin/httpd -DFOREGROUND*

    *├─16586 /usr/sbin/httpd -DFOREGROUND*

    *├─16587 /usr/sbin/httpd -DFOREGROUND*

    *├─16588 /usr/sbin/httpd -DFOREGROUND*

    *├─16589 /usr/sbin/httpd -DFOREGROUND*

    *└─16590 /usr/sbin/httpd -DFOREGROUND*

    *Apr 18 22:15:06 ip-172-31-31-68.us-west-2.compute.internal systemd[1]: Stopped The Apache HTTP Server.*

    *Apr 18 22:15:06 ip-172-31-31-68.us-west-2.compute.internal systemd[1]: Starting The Apache HTTP Server...*

    *Apr 18 22:15:06 ip-172-31-31-68.us-west-2.compute.internal systemd[1]: Started The Apache HTTP Server.*
[**2.3. Modifying Control Groups - Red Hat Customer Portal**
*Each persistent unit supervised by systemd has a unit configuration file in the /usr/lib/systemd/system/ directory. To…*access.redhat.com](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/resource_management_guide/sec-modifying_control_groups)
> *Mounting FileSystem using Systemd*

* *If we look at the /etc/fstab file, it becomes too short nowadays*

    *# cat /etc/fstab*

    *#*

    *# /etc/fstab*

    *# Created by anaconda on Mon Jan 28 20:51:49 2019*

    *#*

    *# Accessible filesystems, by reference, are maintained under '/dev/disk'*

    *# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info*

    *#*

    *UUID=f41e390f-835b-4223-a9bb-9b45984ddf8d /                       xfs     defaults        0 0*

* *One of the reasons for that, all the previous mounts which were handled by fstab earlier is now handled by Systemd*

    *# cd /usr/lib/systemd/system
    # ls *mount*

    *dev-hugepages.mount  proc-fs-nfsd.mount                 proc-sys-fs-binfmt_misc.mount  sys-kernel-config.mount  tmp.mount*

    *dev-mqueue.mount     proc-sys-fs-binfmt_misc.automount  sys-fs-fuse-connections.mount  sys-kernel-debug.mount   var-lib-nfs-rpc_pipefs.mount*

* *If you open any of the files*

    *# cat tmp.mount*

    *#  This file is part of systemd.*

    *#*

    *#  systemd is free software; you can redistribute it and/or modify it*

    *#  under the terms of the GNU Lesser General Public License as published by*

    *#  the Free Software Foundation; either version 2.1 of the License, or*

    *#  (at your option) any later version.*

    *[Unit]*

    *Description=Temporary Directory*

    *Documentation=man:hier(7)*

    *Documentation=http://www.freedesktop.org/wiki/Software/systemd/APIFileSystems*

    *ConditionPathIsSymbolicLink=!/tmp*

    *DefaultDependencies=no*

    *Conflicts=umount.target*

    *Before=local-fs.target umount.target*

    ***[Mount]***

    ***What=tmpfs***

    ***Where=/tmp***

    ***Type=tmpfs***

    ***Options=mode=1777,strictatime***

    *# Make 'systemctl enable tmp.mount' work:*

    *[Install]*

    *WantedBy=local-fs.target*

* *The important section is*

    ** What to mount
    * Where to mount
    * Type of filesystem
    * Options you can pass to this mount *

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
