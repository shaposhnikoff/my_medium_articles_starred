
# 100 Days of DevOps — Day 54-And You Thought You Knew RPM

Welcome to Day 54 of 100 Days of DevOps, Focus for today is Introduction to Regular Expression

* *Finally, I get a chance to write about this topic which I was planning to do for quite a long time. The common trend I am seeing now a days (your opinion might differ) with the growth of Cloud people are forgetting basic Linux concepts like rpm. It’s good to have knowledge about Cloud to survive in the current market but it's important we must know these basic concepts as these are the building blocks to debug any issues.*

* *Tomorrow I am going to discuss YUM which you can think right now as the front-end to rpm*
> What is package management?

* *The main goal behind package management is to install, remove and see information regarding software package*
> Different Linux Distro support different Linux package

* *RedHat: rpm,yum,dnf*

* *Debian: apt,dpkg*
> Installing software using rpm

    *# rpm -ivh vsftpd-3.0.2-25.el7.x86_64.rpm*

    *Preparing...                          ################################# [100%]*

    *Updating / installing...*

    *1:vsftpd-3.0.2-25.el7              ################################# [100%]*

* *I → install*

* *v → verbose*

* *h → hash to see the progress*

* *To upgrade the package*

    *# rpm -Uvh <new package name>*

* *-U → This upgrades or installs the package currently installed to a newer version. This is the same as install, except all other versions of the package are removed after the new package is*
> Display all installed packages(rpm -qa)

    *rpm -qa*
> Which package owns the file(rpm -qf)

    *# rpm -qf /etc/ssh/sshd_config*

    *openssh-server-7.4p1-16.el7.x86_64*
> Basic package information(rpm -qi)

    *# rpm -qi openssh-server*

    *Name        : openssh-server*

    *Version     : 7.4p1*

    *Release     : 16.el7*

    *Architecture: x86_64*

    *Install Date: Mon 28 Jan 2019 08:54:58 PM UTC*

    *Group       : System Environment/Daemons*

    *Size        : 993810*

    *License     : BSD*

    *Signature   : RSA/SHA256, Wed 25 Apr 2018 11:32:56 AM UTC, Key ID 24c6a8a7f4a80eb5*

    *Source RPM  : openssh-7.4p1-16.el7.src.rpm*

    *Build Date  : Wed 11 Apr 2018 04:21:33 AM UTC*

    *Build Host  : x86-01.bsys.centos.org*

    *Relocations : (not relocatable)*

    *Packager    : CentOS BuildSystem <http://bugs.centos.org>*

    *Vendor      : CentOS*

    *URL         : http://www.openssh.com/portable.html*

    *Summary     : An open source SSH server daemon*

    *Description :*

    *OpenSSH is a free version of SSH (Secure SHell), a program for logging*

    *into and executing commands on a remote machine. This package contains*

    *the secure shell daemon (sshd). The sshd daemon allows SSH clients to*

    *securely connect to your SSH server.*
> List all the files installed with this package(rpm -ql)

    *# rpm -ql openssh-server*

    */etc/pam.d/sshd*

    */etc/ssh/sshd_config*

    */etc/sysconfig/sshd*

    */usr/lib/systemd/system/sshd-keygen.service*

    */usr/lib/systemd/system/sshd.service*

    */usr/lib/systemd/system/sshd.socket*

    */usr/lib/systemd/system/sshd@.service*

    */usr/lib64/fipscheck/sshd.hmac*

    */usr/libexec/openssh/sftp-server*

    */usr/sbin/sshd*

    */usr/sbin/sshd-keygen*

    */usr/share/man/man5/moduli.5.gz*

    */usr/share/man/man5/sshd_config.5.gz*

    */usr/share/man/man8/sftp-server.8.gz*

    */usr/share/man/man8/sshd.8.gz*

    */var/empty/sshd*
> Verify/Display missing file(rpm -Va)

    *# rpm -Va*

    *S.5....T.  c /etc/ssh/sshd_config*

    *S.5....T.  c /etc/sysconfig/authconfig*

    *.M.......  g /boot/initramfs-3.10.0-957.1.3.el7.x86_64.img*

    *missing     /usr/lib/python2.7/site-packages/urllib3-1.10.2-py2.7.egg-info*

    *missing     /usr/lib/python2.7/site-packages/urllib3-1.10.2-py2.7.egg-info/PKG-INFO*

    *missing     /usr/lib/python2.7/site-packages/urllib3-1.10.2-py2.7.egg-info/SOURCES.txt*

    *missing     /usr/lib/python2.7/site-packages/urllib3-1.10.2-py2.7.egg-info/dependency_links.txt*

    *missing     /usr/lib/python2.7/site-packages/urllib3-1.10.2-py2.7.egg-info/top_level.txt*

    *S.5....T.    /usr/lib/python2.7/site-packages/urllib3/__init__.py*

    *S.5....T.    /usr/lib/python2.7/site-packages/urllib3/__init__.pyc*

    *missing     /usr/lib/python2.7/site-packages/urllib3/__init__.pyo*

    *S.5....T.    /usr/lib/python2.7/site-packages/urllib3/_collections.py*

    *S.5....T.    /usr/lib/python2.7/site-packages/urllib3/_collections.pyc*

    *missing     /usr/lib/python2.7/site-packages/urllib3/_collections.pyo*
> ***S** file **S**ize differs*
> ***M** **M**ode differs (includes permissions and file type)*
> ***5** digest (formerly MD**5** sum) differs*
> ***D** **D**evice major/minor number mismatch*
> ***L** read**L**ink(2) path mismatch*
> ***U** **U**ser ownership differs*
> ***G** **G**roup ownership differs*
> ***T** m**T**ime differs*
> ***P** ca**P**abilities differ*
> List config file(rpm -qc)

    *# rpm -qc openssh-server*

    */etc/pam.d/sshd*

    */etc/ssh/sshd_config*

    */etc/sysconfig/sshd*
> List documentation file(rpm -qd)

    *# rpm -qd openssh-server*

    */usr/share/man/man5/moduli.5.gz*

    */usr/share/man/man5/sshd_config.5.gz*

    */usr/share/man/man8/sftp-server.8.gz*

    */usr/share/man/man8/sshd.8.gz*
> List dependencies

    *# rpm -qR openssh-server*

    */bin/bash*

    */bin/sh*

    */bin/sh*

    */bin/sh*

    */bin/sh*

    */usr/sbin/useradd*

    *config(openssh-server) = 7.4p1-16.el7*

    *fipscheck-lib(x86-64) >= 1.3.0*

    *libaudit.so.1()(64bit)*

    *libc.so.6()(64bit)*

    *libc.so.6(GLIBC_2.14)(64bit)*

    *libc.so.6(GLIBC_2.16)(64bit)*

    *libc.so.6(GLIBC_2.17)(64bit)*

    *libc.so.6(GLIBC_2.2.5)(64bit)*

    *libc.so.6(GLIBC_2.3)(64bit)*

    *libc.so.6(GLIBC_2.3.4)(64bit)*

    *libc.so.6(GLIBC_2.4)(64bit)*

    *libc.so.6(GLIBC_2.8)(64bit)*

    *libcom_err.so.2()(64bit)*

    *libcrypt.so.1()(64bit)*

    *libcrypt.so.1(GLIBC_2.2.5)(64bit)*

    *libcrypto.so.10()(64bit)*

    *libcrypto.so.10(OPENSSL_1.0.1_EC)(64bit)*

    *libcrypto.so.10(OPENSSL_1.0.2)(64bit)*

    *libcrypto.so.10(libcrypto.so.10)(64bit)*

    *libdl.so.2()(64bit)*

    *libfipscheck.so.1()(64bit)*

    *libgssapi_krb5.so.2()(64bit)*

    *libgssapi_krb5.so.2(gssapi_krb5_2_MIT)(64bit)*

    *libk5crypto.so.3()(64bit)*

    *libkrb5.so.3()(64bit)*

    *libkrb5.so.3(krb5_3_MIT)(64bit)*

    *liblber-2.4.so.2()(64bit)*

    *libldap-2.4.so.2()(64bit)*

    *libpam.so.0()(64bit)*

    *libpam.so.0(LIBPAM_1.0)(64bit)*

    *libresolv.so.2()(64bit)*

    *libselinux.so.1()(64bit)*

    *libsystemd.so.0()(64bit)*

    *libsystemd.so.0(LIBSYSTEMD_209)(64bit)*

    *libutil.so.1()(64bit)*

    *libutil.so.1(GLIBC_2.2.5)(64bit)*

    *libwrap.so.0()(64bit)*

    *libz.so.1()(64bit)*

    *openssh = 7.4p1-16.el7*

    *pam >= 1.0.1-3*

    *rpmlib(CompressedFileNames) <= 3.0.4-1*

    *rpmlib(FileDigests) <= 4.6.0-1*

    *rpmlib(PayloadFilesHavePrefix) <= 4.0-1*

    *rtld(GNU_HASH)*

    *systemd-units*

    *systemd-units*

    *systemd-units*

    *rpmlib(PayloadIsXz) <= 5.2-1*
> List what provides a dependency

    *# rpm -q --whatprovides /bin/bash*

    *bash-4.2.46-31.el7.x86_64*

*NOTE: This is one of the drawbacks of rpm as we need to satisfy all of the dependencies*
> Display recently installed packages

    *# rpm -qa --last*

    *wget-1.14-18.el7.x86_64                       Fri 05 Apr 2019 04:37:41 PM UTC*

    *mailcap-2.1.41-2.el7.noarch                   Fri 05 Apr 2019 04:21:25 AM UTC*

    *httpd-2.4.6-88.el7.centos.x86_64              Fri 05 Apr 2019 04:21:25 AM UTC*

    *centos-logos-70.0.6-3.el7.centos.noarch       Fri 05 Apr 2019 04:21:25 AM UTC*

    *httpd-tools-2.4.6-88.el7.centos.x86_64        Fri 05 Apr 2019 04:21:24 AM UTC*

    *apr-util-1.5.2-6.el7.x86_64                   Fri 05 Apr 2019 04:21:24 AM UTC*

    *apr-1.4.8-3.el7_4.1.x86_64                    Fri 05 Apr 2019 04:21:24 AM UTC*

    *vim-enhanced-7.4.160-5.el7.x86_64             Fri 05 Apr 2019 04:03:34 AM UTC*
> Get information about package without installing it

    *# rpm -qpi vsftpd-3.0.2-25.el7.x86_64.rpm*

    *Name        : vsftpd*

    *Version     : 3.0.2*

    *Release     : 25.el7*

    *Architecture: x86_64*

    *Install Date: (not installed)*

    *Group       : System Environment/Daemons*

    *Size        : 361335*

    *License     : GPLv2 with exceptions*

    *Signature   : RSA/SHA256, Mon 12 Nov 2018 02:48:54 PM UTC, Key ID 24c6a8a7f4a80eb5*

    *Source RPM  : vsftpd-3.0.2-25.el7.src.rpm*

    *Build Date  : Tue 30 Oct 2018 07:45:10 PM UTC*

    *Build Host  : x86-01.bsys.centos.org*

    *Relocations : (not relocatable)*

    *Packager    : CentOS BuildSystem <http://bugs.centos.org>*

    *Vendor      : CentOS*

    *URL         : https://security.appspot.com/vsftpd.html*

    *Summary     : Very Secure Ftp Daemon*

    *Description :*

    *vsftpd is a Very Secure FTP daemon. It was written completely from*

    *scratch.*
> List packages by size

    *# rpm -qa --queryformat '%{name} %{size}\n' | sort -n -k 2*

    *basesystem 0*

    *filesystem 0*

    *gpg-pubkey 0*

    *gpg-pubkey 0*

    *grub2 0*

    *grub2-pc 0*

    *vim-filesystem 0*

    *dracut-config-generic 14*

    *rootfiles 599*

    *python-backports 638*

    *elfutils-default-yama-scope 1810*

    *crontabs 3700*

    *systemd-sysv 3979*

    *dracut-config-rescue 4067*

    *perl-macros 5134*

    *selinux-policy 6482*

    *perl-parent 8141*

    *fipscheck-lib 11466*

    *libverto-libevent 11552*

    *nss-sysinit 14061*
> List third-party packages

    *# rpm -qa --qf '%{name} %{vendor}\n' | grep -vi centos*

    *epel-release Fedora Project*

    *python2-pip Fedora Project*

    *gpg-pubkey (none)*

    *gpg-pubkey (none)*
> Reset permission of package

    *# rpm --setperms openssh-server*
> Before installing rpm check it’s signature

    *# rpm -K vsftpd-3.0.2-25.el7.x86_64.rpm*

    *vsftpd-3.0.2-25.el7.x86_64.rpm: rsa sha1 (md5) pgp md5 OK*
> To list all the signatures

    *# rpm -qa gpg-pubkey**

    *gpg-pubkey-f4a80eb5-53a7ff4b*

    *gpg-pubkey-352c64e5-52ae6884*
> For details of specific signature

    *# rpm -qi gpg-pubkey-f4a80eb5-53a7ff4b*

    *Name        : gpg-pubkey*

    *Version     : f4a80eb5*

    *Release     : 53a7ff4b*

    *Architecture: (none)*

    *Install Date: Thu 28 Mar 2019 03:47:12 AM UTC*

    *Group       : Public Keys*

    *Size        : 0*

    *License     : pubkey*

    *Signature   : (none)*

    *Source RPM  : (none)*

    *Build Date  : Mon 23 Jun 2014 10:19:55 AM UTC*

    *Build Host  : localhost*

    *Relocations : (not relocatable)*

    *Packager    : CentOS-7 Key (CentOS 7 Official Signing Key) <security@centos.org>*

    *Summary     : gpg(CentOS-7 Key (CentOS 7 Official Signing Key) <security@centos.org>)*

    *Description :*

    *-----BEGIN PGP PUBLIC KEY BLOCK-----*

    *Version: rpm-4.11.3 (NSS-3)*

    *mQINBFOn/0sBEADLDyZ+DQHkcTHDQSE0a0B2iYAEXwpPvs67cJ4tmhe/iMOyVMh9*

    *Yw/vBIF8scm6T/vPN5fopsKiW9UsAhGKg0epC6y5ed+NAUHTEa6pSOdo7CyFDwtn*

    *4HF61Esyb4gzPT6QiSr0zvdTtgYBRZjAEPFVu3Dio0oZ5UQZ7fzdZfeixMQ8VMTQ*

    *4y4x5vik9B+cqmGiq9AW71ixlDYVWasgR093fXiD9NLT4DTtK+KLGYNjJ8eMRqfZ*

    *Ws7g7C+9aEGHfsGZ/SxLOumx/GfiTloal0dnq8TC7XQ/JuNdB9qjoXzRF+faDUsj*

    *WuvNSQEqUXW1dzJjBvroEvgTdfCJfRpIgOrc256qvDMp1SxchMFltPlo5mbSMKu1*

    *x1p4UkAzx543meMlRXOgx2/hnBm6H6L0FsSyDS6P224yF+30eeODD4Ju4BCyQ0jO*

    *IpUxmUnApo/m0eRelI6TRl7jK6aGqSYUNhFBuFxSPKgKYBpFhVzRM63Jsvib82rY*

    *438q3sIOUdxZY6pvMOWRkdUVoz7WBExTdx5NtGX4kdW5QtcQHM+2kht6sBnJsvcB*

    *JYcYIwAUeA5vdRfwLKuZn6SgAUKdgeOtuf+cPR3/E68LZr784SlokiHLtQkfk98j*

    *NXm6fJjXwJvwiM2IiFyg8aUwEEDX5U+QOCA0wYrgUQ/h8iathvBJKSc9jQARAQAB*

    *tEJDZW50T1MtNyBLZXkgKENlbnRPUyA3IE9mZmljaWFsIFNpZ25pbmcgS2V5KSA8*

    *c2VjdXJpdHlAY2VudG9zLm9yZz6JAjUEEwECAB8FAlOn/0sCGwMGCwkIBwMCBBUC*

    *CAMDFgIBAh4BAheAAAoJECTGqKf0qA61TN0P/2730Th8cM+d1pEON7n0F1YiyxqG*

    *QzwpC2Fhr2UIsXpi/lWTXIG6AlRvrajjFhw9HktYjlF4oMG032SnI0XPdmrN29lL*

    *F+ee1ANdyvtkw4mMu2yQweVxU7Ku4oATPBvWRv+6pCQPTOMe5xPG0ZPjPGNiJ0xw*

    *4Ns+f5Q6Gqm927oHXpylUQEmuHKsCp3dK/kZaxJOXsmq6syY1gbrLj2Anq0iWWP4*

    *Tq8WMktUrTcc+zQ2pFR7ovEihK0Rvhmk6/N4+4JwAGijfhejxwNX8T6PCuYs5Jiv*

    *hQvsI9FdIIlTP4XhFZ4N9ndnEwA4AH7tNBsmB3HEbLqUSmu2Rr8hGiT2Plc4Y9AO*

    *aliW1kOMsZFYrX39krfRk2n2NXvieQJ/lw318gSGR67uckkz2ZekbCEpj/0mnHWD*

    *3R6V7m95R6UYqjcw++Q5CtZ2tzmxomZTf42IGIKBbSVmIS75WY+cBULUx3PcZYHD*

    *ZqAbB0Dl4MbdEH61kOI8EbN/TLl1i077r+9LXR1mOnlC3GLD03+XfY8eEBQf7137*

    *YSMiW5r/5xwQk7xEcKlbZdmUJp3ZDTQBXT06vavvp3jlkqqH9QOE8ViZZ6aKQLqv*

    *pL+4bs52jzuGwTMT7gOR5MzD+vT0fVS7Xm8MjOxvZgbHsAgzyFGlI1ggUQmU7lu3*

    *uPNL0eRx4S1G4Jn5*

    *=OGYX*

    *-----END PGP PUBLIC KEY BLOCK-----*
> In case of third party package

* *Import signature into rpm database and then verify it using rpm -K*

    *# rpm --import <location of key-file> *
> Extract files from rpm

    *# rpm2cpio vsftpd-3.0.2-25.el7.x86_64.rpm |cpio -idmv*

* *i, — extract*

* *d, — make-directories*

* *m, — preserve-modification-time*

* *v, — verbose*

* *The above command will extract all the files in the current directory*

    *drwxr-xr-x 3 root root     17 Apr  6 04:50 var*

    *drwxr-xr-x 5 root root     42 Apr  6 04:50 usr*

    *drwxr-xr-x 5 root root     52 Apr  6 04:50 etc*
> Remove rpm package

    *# rpm -e vsftpd*

* *Some handy options*

    *--allmatches --> Remove all versions
    --nodeps     --> Ignore dependencies: 
    --test -vv   --> Test, but don't uninstall*

* *Test before removing the package*

    *# rpm -e httpd --test*

    *You have new mail in /var/spool/mail/root*

    *[root@ip-172-31-31-68 tmp]# rpm -e httpd --test -vv*

    *D: loading keyring from pubkeys in /var/lib/rpm/pubkeys/*.key*

    *D: couldn't find any keys in /var/lib/rpm/pubkeys/*.key*

    *D: loading keyring from rpmdb*

    *D: opening  db environment /var/lib/rpm cdb:0x401*

    *D: opening  db index       /var/lib/rpm/Packages 0x400 mode=0x0*

    *D: locked   db index       /var/lib/rpm/Packages*

    *D: opening  db index       /var/lib/rpm/Name 0x400 mode=0x0*

    *D:  read h#     232 Header SHA1 digest: OK (489efff35e604042709daf46fb78611fe90a75aa)*

    *D: added key gpg-pubkey-f4a80eb5-53a7ff4b to keyring*

    *D:  read h#     234 Header SHA1 digest: OK (dd737a402556b7653c2bc971f343532046e26384)*

    *D: added key gpg-pubkey-352c64e5-52ae6884 to keyring*

    *D: Using legacy gpg-pubkey(s) from rpmdb*

    *D:  read h#     272 Header V3 RSA/SHA256 Signature, key ID f4a80eb5: OK*

    *D: opening  db index       /var/lib/rpm/Conflictname 0x400 mode=0x0*

    *D: ========== --- httpd-2.4.6-88.el7.centos x86_64/linux 0x0*

    *D: opening  db index       /var/lib/rpm/Requirename 0x400 mode=0x0*

    *D: ========== recording tsort relations*

    *D: ========== tsorting packages (order, #predecessors, #succesors, depth)*

    *D:     0    0    0    1   -httpd-2.4.6-88.el7.centos.x86_64*

    *D: erasing packages*

    *D: Selinux disabled.*

    *D: sanity checking 1 elements*

    *D: computing 496 file fingerprints*

    *D: opening  db index       /var/lib/rpm/Basenames 0x400 mode=0x0*

    *D: opening  db index       /var/lib/rpm/Group 0x400 mode=0x0*

    *D: opening  db index       /var/lib/rpm/Providename 0x400 mode=0x0*

    *D: opening  db index       /var/lib/rpm/Obsoletename 0x400 mode=0x0*

    *D: opening  db index       /var/lib/rpm/Triggername 0x400 mode=0x0*

    *D: opening  db index       /var/lib/rpm/Dirnames 0x400 mode=0x0*

    *D: opening  db index       /var/lib/rpm/Installtid 0x400 mode=0x0*

    *D: opening  db index       /var/lib/rpm/Sigmd5 0x400 mode=0x0*

    *D: opening  db index       /var/lib/rpm/Sha1header 0x400 mode=0x0*

    *Preparing packages...*

    *D: computing file dispositions*

    *D: 0x0000ca01     4096      1769925      4157797 /*

    *D: 0x00000013     4096       112280       126285 /run*

    *D: ========== +++ httpd-2.4.6-88.el7.centos x86_64-linux 0x0*

    *D: closed   db index       /var/lib/rpm/Sha1header*

    *D: closed   db index       /var/lib/rpm/Sigmd5*

    *D: closed   db index       /var/lib/rpm/Installtid*

    *D: closed   db index       /var/lib/rpm/Dirnames*

    *D: closed   db index       /var/lib/rpm/Triggername*

    *D: closed   db index       /var/lib/rpm/Obsoletename*

    *D: closed   db index       /var/lib/rpm/Conflictname*

    *D: closed   db index       /var/lib/rpm/Providename*

    *D: closed   db index       /var/lib/rpm/Requirename*

    *D: closed   db index       /var/lib/rpm/Group*

    *D: closed   db index       /var/lib/rpm/Basenames*

    *D: closed   db index       /var/lib/rpm/Name*

    *D: closed   db index       /var/lib/rpm/Packages*

    *D: closed   db environment /var/lib/rpm*
> Build your own rpm(Simple rpm creation)

***Step1: **Make sure these two packages must be installed*

    ***rpm**-build-4.11.3-35.el7.x86_64*

    ***rpm**devtools-8.3-5.el7.noarch*

***Step2:** Create Directory Structure*

    *$ rpmdev-setuptree*

* *Above command will create a directory structure like below*

    *$ tree*

    *.*

    *└── rpmbuild*

    *├── BUILD*

    *├── RPMS*

    *├── SOURCES*

    *├── SPECS*

    *└── SRPMS*

***Step 3**— Copy Files under SOURCES Directory*

* *Copy all your files and scripts folder inside **~/rpmbuild/SOURCES** directory, which we need to add in rpm file. Here I am using a simple index.html*

    *$ tree SOURCES/*

    *SOURCES/*

    *├── myhomepage-1*

    *│   └── tmp*

    *│       └── user*

    *│           └── index.html*

* *Create a tar ball of your code*

    ***tar** -cvf myhomepage-1.**tar** myhomepage-1*

***Step 4**— Create SPEC File*

* *Create a spec file **~/rpmbuild/SPECS/**myhomepage-1.spec*

<iframe src="https://medium.com/media/49d5aae0b11a85e0917cea7170c34537" frameborder=0></iframe>

* *SPEC file is self-explanatory, Change package name, script path, archive name, description etc, as per your requirement.*

***Step5 :** Build RPM*

<iframe src="https://medium.com/media/15346c5efd98b0647e6e5e58bfaea0ee" frameborder=0></iframe>

* *Install the rpm*

    *$ sudo rpm -ivh RPMS/x86_64/myhomepage-1-0.x86_64.rpm*

    *Preparing...                          ################################# [100%]*

    *Updating / installing...*

    *1:myhomepage-1-0                   ################################# [100%]*

* *Verify it*

    *$ ls -l /tmp/user/*

    *total 4*

    *-rw-r--r-- 1 root root 12 Apr  6 05:45 index.html*
> Location to Obtain RPM Packages

***Centos7***
[**Index of /centos/7/os/x86_64/Packages**
*Edit description*mirror.centos.org](http://mirror.centos.org/centos/7/os/x86_64/Packages/)

***Centos EPEL REPO***
[**Index of /pub/epel/7Server/x86_64/Packages**
*Edit description*dl.fedoraproject.org](https://dl.fedoraproject.org/pub/epel/7Server/x86_64/Packages/)
[**RPM Search**
*PBone RPM Search engine for Redhat CentOS Fedora Scientific SuSE Mandriva StartCom Linux packages.*rpm.pbone.net](http://rpm.pbone.net/)
[**Rpmfind mirror**
*This server is located in Lyon, within the Creatis laboratory. Like other rpmfind mirrors, this machine is using a…*rpmfind.net](https://rpmfind.net/)

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
