
# 100 Days of DevOps — Day 55-Introduction to YUM

Welcome to Day 55 of 100 Days of DevOps, Focus for today is Introduction to YUM
> What is YUM?

* *Yellowdog Updater Modified*

* *Resolve one of the major issues with rpm where we need to resolve all the package dependencies*

* *It acts as a frontend to rpm and handles all dependency issues*

* *No need to copy rpm locally*

* *You need to configure repositories(which is a collection of rpm files)to pull rpm(HTTP, FTP, HTTPS, local CD-ROM)*
> Installing Package with yum

    *# yum -y install bind*

* *-y to answer yes automatically*
> Download the package without installing it

    *# yum install --downloadonly --downloaddir=/tmp  cloud-init.x86_64*

    *Loaded plugins: fastestmirror*

    *Loading mirror speeds from cached hostfile*

    ** base: mirrors.cat.pdx.edu*

    ** epel: mirrors.cat.pdx.edu*

    ** extras: mirrors.cat.pdx.edu*

    ** updates: mirrors.cat.pdx.edu*

    *Resolving Dependencies*

    *--> Running transaction check*

    *---> Package cloud-init.x86_64 0:18.2-1.el7.centos.1 will be updated*

    *---> Package cloud-init.x86_64 0:18.2-1.el7.centos.2 will be an update*

    *--> Finished Dependency Resolution*

    *Dependencies Resolved*

    *============================================================================================================================================================================================================*

    *Package                                         Arch                                        Version                                                     Repository                                    Size*

    *============================================================================================================================================================================================================*

    *Updating:*

    *cloud-init                                      x86_64                                      18.2-1.el7.centos.2                                         updates                                      779 k*

    *Transaction Summary*

    *============================================================================================================================================================================================================*

    *Upgrade  1 Package*

    *Total download size: 779 k*

    *Background downloading packages, then exiting:*

    *Delta RPMs disabled because /usr/bin/applydeltarpm not installed.*

    *cloud-init-18.2-1.el7.centos.2.x86_64.rpm                                                                                                                                            | 779 kB  00:00:00*

    *exiting because "Download Only" specified*

***NOTE: **These are corner cases where you just want to download the package without installing it i.e maybe you want to build your own local YUM server or want to verify the package before installing it*
> Upgrade vs Update (This is a common confusion point)

    ** yum upgrade --> Delete Obselete Package
    * yum update  --> Preserve Obselete Package(I always prefer this) *
> Remove Package via yum

    *# yum remove bind*

    *Loaded plugins: fastestmirror*

    *Resolving Dependencies*

    *--> Running transaction check*

    *---> Package bind.x86_64 32:9.9.4-73.el7_6 will be erased*

    *--> Finished Dependency Resolution*

    *Dependencies Resolved*

    *============================================================================================================================================================================================================*

    *Package                                     Arch                                          Version                                                    Repository                                       Size*

    *============================================================================================================================================================================================================*

    *Removing:*

    *bind                                        x86_64                                        32:9.9.4-73.el7_6                                          @updates                                        4.5 M*

    *Transaction Summary*

    *============================================================================================================================================================================================================*

    *Remove  1 Package*

    *Installed size: 4.5 M*

    *Is this ok [y/N]: N*

    *Exiting on user command*

    *Your transaction was saved, rerun it with:*

    *yum load-transaction /tmp/yum_save_tx.2019-04-06.17-12.LD0_Pg.yumtx*

    *[root@ip-172-31-31-68 ~]# yum remove bind*

    *Loaded plugins: fastestmirror*

    *Resolving Dependencies*

    *--> Running transaction check*

    *---> Package bind.x86_64 32:9.9.4-73.el7_6 will be erased*

    *--> Finished Dependency Resolution*

    *Dependencies Resolved*

    *============================================================================================================================================================================================================*

    *Package                                     Arch                                          Version                                                    Repository                                       Size*

    *============================================================================================================================================================================================================*

    *Removing:*

    *bind                                        x86_64                                        32:9.9.4-73.el7_6                                          @updates                                        4.5 M*

    *Transaction Summary*

    *============================================================================================================================================================================================================*

    *Remove  1 Package*

    *Installed size: 4.5 M*

    *Is this ok [y/N]:*

***NOTE: **I didn’t put -y here while removing a package, I always do it intentionally so that I didn’t remove any package un-intentionally, however, this is a completely different story when I am building any automation script/tool.*
> Display Package Information

    *# yum list all
    GeoIP.x86_64                            1.5.0-13.el7                   installed*

    *PyYAML.x86_64                           3.10-11.el7                    installed*

    *acl.x86_64                              2.2.51-14.el7                  installed*

    *apr.x86_64                              1.4.8-3.el7_4.1                @base*

    *apr-util.x86_64                         1.5.2-6.el7                    @base*

    *audit.x86_64                            2.8.4-4.el7                    installed*

    *audit-libs.x86_64                       2.8.4-4.el7                    installed*

    *audit-libs-python.x86_64                2.8.4-4.el7                    installed*

    *authconfig.x86_64                       6.2.8-30.el7                   installed*

***OR***

    *# yum list all |wc -l*

    *23820*
> To get the list of installed package

    *# yum list available 
    GeoIP.x86_64                            1.5.0-13.el7                   installed*

    *PyYAML.x86_64                           3.10-11.el7                    installed*

    *acl.x86_64                              2.2.51-14.el7                  installed*

    *apr.x86_64                              1.4.8-3.el7_4.1                @base*

    *apr-util.x86_64                         1.5.2-6.el7                    @base*

    *audit.x86_64                            2.8.4-4.el7                    installed*

    *audit-libs.x86_64                       2.8.4-4.el7                    installed*

    *audit-libs-python.x86_64                2.8.4-4.el7                    installed*

    *authconfig.x86_64                       6.2.8-30.el7                   installed*
> To get the list of available package

    *# yum list available
    Available Packages*

    *0ad.x86_64                               0.0.22-1.el7                    epel*

    *0ad-data.noarch                          0.0.22-1.el7                    epel*

    *0install.x86_64                          2.11-1.el7                      epel*

    *2048-cli.x86_64                          0.9.1-1.el7                     epel*

    *2048-cli-nocurses.x86_64                 0.9.1-1.el7                     epel*

    *2ping.noarch                             3.2.1-2.el7                     epel*

    *389-admin.x86_64                         1.1.46-1.el7                    epel*

    *389-admin-console.noarch                 1.1.12-1.el7                    epel*

    *389-admin-console-doc.noarch             1.1.12-1.el7                    epel*

    *389-adminutil.x86_64                     1.1.21-2.el7                    epel*

    *389-adminutil-devel.x86_64               1.1.21-2.el7                    epel*

    *389-console.noarch                       1.1.18-1.el7                    epel*

    *389-ds.noarch                            1.2.2-6.el7                     epel*
> If you are looking for a specific package

    *# yum list httpd*

    *Installed Packages*

    *httpd.x86_64                                                                                    2.4.6-88.el7.centos                                                                                    @base*
> Basic package information

    *# yum info httpd*

    *Installed Packages*

    *Name        : httpd*

    *Arch        : x86_64*

    *Version     : 2.4.6*

    *Release     : 88.el7.centos*

    *Size        : 9.4 M*

    *Repo        : installed*

    *From repo   : base*

    *Summary     : Apache HTTP Server*

    *URL         : http://httpd.apache.org/*

    *License     : ASL 2.0*

    *Description : The Apache HTTP Server is a powerful, efficient, and extensible*

    *: web server.*

* *Similar to rpm -qi command*
> If you searching for a specific package

    *# yum search httpd*

    *============================================================================================ N/S matched: httpd ============================================================================================*

    *iipsrv-**httpd**-fcgi.noarch : Apache **HTTPD** files for iipsrv*

    *keycloak-**httpd**-client-install.noarch : Tools to configure Apache **HTTPD** as Keycloak client*

    *libmicro**httpd**-devel.i686 : Development files for libmicro**httpd***

    *libmicro**httpd**-devel.x86_64 : Development files for libmicro**httpd***

    *libmicro**httpd**-doc.noarch : Documentation for libmicro**httpd***

    *lig**httpd**-fastcgi.x86_64 : FastCGI module and spawning helper for lig**httpd** and PHP configuration*

    *lig**httpd**-mod_authn_gssapi.x86_64 : Authentication module for lig**httpd** that uses GSSAPI*

    *lig**httpd**-mod_authn_mysql.x86_64 : Authentication module for lig**httpd** that uses a MySQL database*

    *lig**httpd**-mod_authn_pam.x86_64 : Authentication module for lig**httpd** that uses PAM*

    *lig**httpd**-mod_geoip.x86_64 : GeoIP module for lig**httpd** to use for location lookups*

    *lig**httpd**-mod_mysql_vhost.x86_64 : Virtual host module for lig**httpd** that uses a MySQL database*

    *nextcloud-**httpd**.noarch : **Httpd** integration for NextCloud*

    *owncloud-**httpd**.noarch : **Httpd** integration for ownCloud*

    *python2-keycloak-**httpd**-client-install.noarch : Tools to configure Apache **HTTPD** as Keycloak client*

    *radicale-**httpd**.noarch : **httpd** config for Radicale*

    *dark**httpd**.x86_64 : A secure, lightweight, fast, single-threaded HTTP/1.1 server*

    ***httpd**.x86_64 : Apache HTTP Server*

    ***httpd**-devel.x86_64 : Development interfaces for the Apache HTTP server*

    ***httpd**-itk.x86_64 : MPM Itk for Apache HTTP Server*

    ***httpd**-manual.noarch : Documentation for the Apache HTTP server*

    ***httpd**-tools.x86_64 : Tools for use with the Apache HTTP Server*

    *libmicro**httpd**.i686 : Lightweight library for embedding a webserver in applications*

    *libmicro**httpd**.x86_64 : Lightweight library for embedding a webserver in applications*

    *lig**httpd**.x86_64 : Lightning fast webserver with light system requirements*

    *mirmon-**httpd**.noarch : Apache configuration for mirmon*

    *mod_auth_mellon.x86_64 : A SAML 2.0 authentication module for the Apache **Httpd** Server*

    *mod_dav_svn.x86_64 : Apache **httpd** module for Subversion server*

    *opensips-**httpd**.x86_64 : HTTP transport layer implementation*

    *perl-Test-Fake-**HTTPD**.noarch : Fake HTTP server module for testing*

    *python2-sphinxcontrib-**httpd**omain.noarch : Sphinx domain for documenting HTTP APIs*

    *sysusage-**httpd**.noarch : Apache configuration for sysusage*

    *t**httpd**.x86_64 : A tiny, turbo, throttleable lightweight HTTP server*

    *viewvc-**httpd**-fcgi.noarch : ViewVC configuration for Apache/mod_fcgid*

    *viewvc-**httpd**-wsgi.noarch : ViewVC configuration for Apache/mod_wsgi*

    *web-assets-**httpd**.noarch : Web Assets aliases for the Apache HTTP daemon*

* *Display Group Information*

    *# yum group list*

    *Available Environment Groups:*

    *Minimal Install*

    *Compute Node*

    *Infrastructure Server*

    *File and Print Server*

    *Cinnamon Desktop*

    *MATE Desktop*

    *Basic Web Server*

    *Virtualization Host*

    *Server with GUI*

    *GNOME Desktop*

    *KDE Plasma Workspaces*

    *Development and Creative Workstation*

    *Available Groups:*

    *Cinnamon*

    *Compatibility Libraries*

    *Console Internet Tools*

    *Development Tools*

    *Educational Software*

    *Electronic Lab*

    *Fedora Packager*

    *General Purpose Desktop*

    *Graphical Administration Tools*

    *Haskell*

    *Legacy UNIX Compatibility*

    *MATE*

    *Milkymist*

    *Scientific Support*

    *Security Tools*

    *Smart Card Support*

    *System Administration Tools*

    *System Management*

    *TurboGears application framework*

    *Xfce*

* *Similar to yum info you can get information about group info*

    *# yum group info "Basic Web Server"*

    *Environment Group: Basic Web Server*

    *Environment-Id: web-server-environment*

    *Description: Server for serving static and dynamic internet content.*

    *Mandatory Groups:*

    *+base*

    *+core*

    *+web-server*

    *Optional Groups:*

    *+backup-client*

    *+debugging*

    *+directory-client*

    *+guest-agents*

    *+hardware-monitoring*

    *+java-platform*

    *+large-systems*

    *+load-balancer*

    *+mariadb-client*

    *+network-file-system-client*

    *+performance*

    *+perl-web*

    *+php*

    *+postgresql-client*

    *+python-web*

    *+remote-system-management*

    *+web-servlet*
> To get yum history

    *# yum history list*

    *Loaded plugins: fastestmirror*

    *ID     | Command line             | Date and time    | Action(s)      | Altered*

    *-------------------------------------------------------------------------------*

    *10 | remove bind              | 2019-04-06 17:15 | Erase          |    1 EE*

    *9 | -y install bind          | 2019-04-06 17:01 | I, U           |    5 E<*

    *8 | -y install tree          | 2019-04-06 05:05 | Install        |    1 >*

    *7 | -y install rpmdevtools   | 2019-04-06 05:03 | Install        |    2*

    *6 | -y install rpm-build.x86 | 2019-04-06 05:02 | Install        |   11*

    *5 | -y install wget          | 2019-04-05 16:37 | Install        |    1*

    *4 | -y install httpd         | 2019-04-05 04:21 | Install        |    6*

    *3 | -y install vim           | 2019-04-05 04:03 | Install        |   31*

    *2 | -y install python2-pip.n | 2019-03-28 03:51 | Install        |    1*

    *1 | -y install epel-release  | 2019-03-28 03:47 | Install        |    1*

    *history list*
> To get history about specific package

    *# yum history list httpd*

    *Loaded plugins: fastestmirror*

    *ID     | Command line             | Date and time    | Action(s)      | Altered*

    *-------------------------------------------------------------------------------*

    *4 | -y install httpd         | 2019-04-05 04:21 | Install        |    6*

    *history list*
> To get the summary about specific package

    *# yum history summary httpd*

    *Loaded plugins: fastestmirror*

    *Login user                 | Time                | Action(s)        | Altered*

    *-------------------------------------------------------------------------------*

    *Cloud User <centos>        | Last week           | Install          |        6*
> You can undo changes based on historyI(ID can br obtained by the output of yum history list httpd*)*

    *# yum history undo 4*

    *Loaded plugins: fastestmirror*

    *Undoing transaction 4, from Fri Apr  5 04:21:24 2019*

    *Dep-Install apr-1.4.8-3.el7_4.1.x86_64              @base*

    *Dep-Install apr-util-1.5.2-6.el7.x86_64             @base*

    *Dep-Install centos-logos-70.0.6-3.el7.centos.noarch @base*

    *Install     httpd-2.4.6-88.el7.centos.x86_64        @base*

    *Dep-Install httpd-tools-2.4.6-88.el7.centos.x86_64  @base*

    *Dep-Install mailcap-2.1.41-2.el7.noarch             @base*

    *Resolving Dependencies*

    *--> Running transaction check*

    *---> Package apr.x86_64 0:1.4.8-3.el7_4.1 will be erased*

    *---> Package apr-util.x86_64 0:1.5.2-6.el7 will be erased*

    *---> Package centos-logos.noarch 0:70.0.6-3.el7.centos will be erased*

    *---> Package httpd.x86_64 0:2.4.6-88.el7.centos will be erased*

    *---> Package httpd-tools.x86_64 0:2.4.6-88.el7.centos will be erased*

    *---> Package mailcap.noarch 0:2.1.41-2.el7 will be erased*

    *--> Finished Dependency Resolution*

    *Dependencies Resolved*

    *============================================================================================================================================================================================================*

    *Package                                           Arch                                        Version                                                     Repository                                  Size*

    *============================================================================================================================================================================================================*

    *Removing:*

    *apr                                               x86_64                                      1.4.8-3.el7_4.1                                             @base                                      221 k*

    *apr-util                                          x86_64                                      1.5.2-6.el7                                                 @base                                      194 k*

    *centos-logos                                      noarch                                      70.0.6-3.el7.centos                                         @base                                       22 M*

    *httpd                                             x86_64                                      2.4.6-88.el7.centos                                         @base                                      9.4 M*

    *httpd-tools                                       x86_64                                      2.4.6-88.el7.centos                                         @base                                      169 k*

    *mailcap                                           noarch                                      2.1.41-2.el7                                                @base                                       62 k*

    *Transaction Summary*

    *============================================================================================================================================================================================================*

    *Remove  6 Packages*

    *Installed size: 31 M*

    *Is this ok [y/N]:*
> You can also use YUM to install a local package present on disk

    *# yum localinstall vsftpd-3.0.2-25.el7.x86_64.rpm*

    *Loaded plugins: fastestmirror*

    *Examining vsftpd-3.0.2-25.el7.x86_64.rpm: vsftpd-3.0.2-25.el7.x86_64*

    *Marking vsftpd-3.0.2-25.el7.x86_64.rpm to be installed*

    *Resolving Dependencies*

    *--> Running transaction check*

    *---> Package vsftpd.x86_64 0:3.0.2-25.el7 will be installed*

    *--> Finished Dependency Resolution*

    *Dependencies Resolved*

    *============================================================================================================================================================================================================*

    *Package                                   Arch                                      Version                                           Repository                                                      Size*

    *============================================================================================================================================================================================================*

    *Installing:*

    ***vsftpd**                                    x86_64                                    3.0.2-25.el7                                      /vsftpd-3.0.2-25.el7.x86_64                                    353 k*

    *Transaction Summary*

    *============================================================================================================================================================================================================*

    *Install  1 Package*

    *Total size: 353 k*

    *Installed size: 353 k*

    *Is this ok [y/d/N]: y*

    *Downloading packages:*

    *Running transaction check*

    *Running transaction test*

    *Transaction test succeeded*

    *Running transaction*

    *Installing : vsftpd-3.0.2-25.el7.x86_64                                                                                                                                                               1/1*

    *Verifying  : vsftpd-3.0.2-25.el7.x86_64                                                                                                                                                               1/1*

    *Installed:*

    *vsftpd.x86_64 0:3.0.2-25.el7*

    *Complete!*
> To clean yum cache

    *# yum clean all*

    *Loaded plugins: fastestmirror*

    *Cleaning repos: base epel extras updates*

    *Cleaning up list of fastest mirrors*

* *The above command will clean all the files inside*

    *# ls -l /var/cache/yum/x86_64/7/*

    *total 4*

    *drwxr-xr-x. 4 root root  33 Apr  6 17:25 base*

    *drwxr-xr-x  4 root root  33 Apr  6 17:25 epel*

    *drwxr-xr-x. 4 root root  33 Apr  6 17:25 extras*

    *-rw-r--r--  1 root root 126 Apr  6 17:04 timedhosts*

    *drwxr-xr-x. 4 root root  33 Apr  6 17:25 updates*
> Yum Plugins

* *To add more functionalities to yum*

* *Display current plugins*

    *# yum info yum*

    ***Loaded plugins: fastestmirror***

    *Installed Packages*

    *Name        : yum*

    *Arch        : noarch*

    *Version     : 3.4.3*

    *Release     : 161.el7.centos*

    *Size        : 5.6 M*

    *Repo        : installed*

    *Summary     : RPM package installer/updater/manager*

    *URL         : http://yum.baseurl.org/*

    *License     : GPLv2+*

    *Description : Yum is a utility that can check for and automatically download and*

    *: install updated RPM packages. Dependencies are obtained and downloaded*

    *: automatically, prompting the user for permission as necessary.*

* *To enable or disable a specific plugin*

    */etc/yum.conf*

    ***plugins=1 (to disable it set it value to zero)***

    *# To enable or disable specific plugin*

    *# ls -l /etc/yum/pluginconf.d/*

    *total 8*

    *-rw-r--r--. 1 root root 279 Oct 30 22:58 fastestmirror.conf*

    *-rw-r--r--. 1 root root 372 Jan 28 20:51 langpacks.conf*

* *Display available plugin*

    *# yum provides "/usr/lib/yum-plugins/*"*

    *PackageKit-yum-plugin-1.1.10-1.el7.centos.x86_64 : Tell PackageKit to check for updates when yum exits*

    *Repo        : base*

    *Matched from:*

    *Filename    : /usr/lib/yum-plugins/refresh-packagekit.py*

    *Filename    : /usr/lib/yum-plugins/refresh-packagekit.pyo*

    *Filename    : /usr/lib/yum-plugins/refresh-packagekit.pyc*

    *etckeeper-1.18.8-1.el7.noarch : Store /etc in a SCM system (git, mercurial, bzr or darcs)*

    *Repo        : epel*

    *Matched from:*

    *Filename    : /usr/lib/yum-plugins/etckeeper.py*

    *Filename    : /usr/lib/yum-plugins/etckeeper.pyc*

    *Filename    : /usr/lib/yum-plugins/etckeeper.pyo*

* *Some useful plugins*

    *- fastmirror   - finds the fastest mirror from mirror list 
    – changelog    - enables the --changelog option
    – snapshot     - automatically snapshot filesystem during updates 
    – versionlock  - enables a feature to "lock" a package*

* *Installing a specific plugin*

    *# yum -y install yum-plugin-changelog-1.1.31-50.el7.noarch*

* *Run yum info command*

    *# yum info yum*

    *Loaded plugins: **changelog**, fastestmirror*

* *Now you will see additional plugin changelog*

* *To list the most recent changelog*

    *# yum changelog 1 httpd*

    *Listing 1 changelog*

    *==================== Installed Packages ====================*

    *httpd-2.4.6-88.el7.centos.x86_64         installed*

    ** Tue Oct 30 12:00:00 2018 CentOS Sources <bugs@centos.org> - 2.4.6-88.el7.centos*

    *- Remove index.html, add centos-noindex.tar.gz*

    *- change vstring*

    *- change symlink for poweredby.png*

    *- update welcome.conf with proper aliases*

    *changelog stats. 1 pkg, 1 source pkg, 1 changelog*

* *Yesterday I forgot to mention this for rpm*

    *# **rpm -q httpd --changelog***

    ** Tue Oct 30 2018 CentOS Sources <bugs@centos.org> - 2.4.6-88.el7.centos*

    *- Remove index.html, add centos-noindex.tar.gz*

    *- change vstring*

    *- change symlink for poweredby.png*

    *- update welcome.conf with proper aliases*

    ** Thu Jun 21 2018 Luboš Uhliarik <luhliari@redhat.com> - 2.4.6-88*

    *- Resolves: #1527295 - httpd with worker/event mpm segfaults after multiple*

    *SIGUSR1*

    ** Thu Jun 21 2018 Luboš Uhliarik <luhliari@redhat.com> - 2.4.6-87*

    *- Resolves: #1458364 - RMM list corruption in ldap module results in server hang*

    ** Thu Jun 21 2018 Luboš Uhliarik <luhliari@redhat.com> - 2.4.6-86*

    *- Resolves: #1493181 - RFE: mod_ssl: allow sending multiple CA names which*

    *differ only in case*
> **Create yum repository**

* ***Step1:** Install createrepo package*

    *yum install createrepo*

* ***Step2:** Copy package to the repository*

    *# cp -vr /tmp/*.rpm /var/www/html/*

    *‘/tmp/cloud-init-18.2-1.el7.centos.2.x86_64.rpm’ -> ‘/var/www/html/cloud-init-18.2-1.el7.centos.2.x86_64.rpm’*

    *‘/tmp/vsftpd-3.0.2-25.el7.x86_64.rpm’ -> ‘/var/www/html/vsftpd-3.0.2-25.el7.x86_64.rpm’*

* ***Step3:** Run createrepo command*

    *# createrepo /var/www/html/*

    *Spawning worker 0 with 2 pkgs*

    *Workers Finished*

    *Saving Primary metadata*

    *Saving file lists metadata*

    *Saving other metadata*

    *Generating sqlite DBs*

    *Sqlite DBs complete*

* ***Step4:** On the client end, create this file and point it to Yum Server*

    *cat example.repo
    [example]*

    *name=Example Repository*

    *baseurl=http://172.31.31.68*

    *enabled=1*

* ***Step5:** Check if the repository is working*

    *# yum repolist*

    *example                                                                                                                                                                              | 2.9 kB  00:00:00*

    *example/primary_db                                                                                                                                                                   | 4.3 kB  00:00:00*

    *repo id                                                                             repo name                                                                                                         status*

***Reference***

<iframe src="https://medium.com/media/d54d861ce0913dd380eb5ccf018ddfbf" frameborder=0></iframe>
[**yum - Trac**
*Yum is an automatic updater and package installer/remover for rpm systems. It automatically computes dependencies and…*yum.baseurl.org](http://yum.baseurl.org/)

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
