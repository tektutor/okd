### Installing OpenShift Community Edition (OKD) of OpenShift in RHEL 8.5
I had setup RHEL 8.5 as Virtual Machine on my Dell Precision 7920 Tower.

Dell Precision 7920 Tower Configuration
- 128 GB RAM 
- Dual Socket with Intel Xeon Silver Processor with 12 Cores on each Processor 
- 1 TB SSD HDD.

However for the Virtual Machine I allocated
- 64 GB RAM
- 12 Cores on each Processor i.e 12 Cores x 2 Processor = 24 Virtual Cores
- 500 GB HDD

I enabled VT-X and IOMMU in VMWare Workstation Pro for this Virtual Machine.

##### Enabling Software Repositories on a fresh RHEL 8.5 OS
After installing RHEL 8.5, you need to register your system with RedHat and enable the below repositories to install softwares. This requires an active RedHat account.  For learning purpose, RedHat offers a Developer license which lets you use RHEL 8.5 Free for non-commercial purpose.

```
sudo subscription-manager register
sudo subscription-manager refresh
sudo subscription-manager attach --auto
sudo subscription-manager list --available --all

sudo subscription-manager repos --enable="rhel-8-for-x86_64-baseos-rpms"
sudo subscription-manager repos --enable="rhel-8-for-x86_64-appstream-rpms"
sudo subscription-manager repos --enable="ansible-2-for-rhel-8-x86_64-rpms"
sudo subscription-manager repos \
--enable=rhel-8-for-x86_64-baseos-rpms \
--enable=rhel-8-for-x86_64-appstream-rpms \
--enable=codeready-builder-for-rhel-8-x86_64-rpms
```
When prompted for username and password, type your redhat developer credentials to register your OS.

### Installing DNS Server in RHEL 8.5
```
sudo dnf install bind bind-utils
```
The expected output is
<pre>
[root@tektutor ~]# <b>dnf install bind bind-utils</b>
Updating Subscription Management repositories.
Red Hat Enterprise Linux 8 for x86_64 - AppStream (RPMs)                                                     15 MB/s |  38 MB     00:02    
Red Hat Enterprise Linux 8 for x86_64 - BaseOS (RPMs)                                                        16 MB/s |  43 MB     00:02    
Last metadata expiration check: 0:00:01 ago on Sat 12 Feb 2022 02:54:12 PM PST.
Package bind-utils-32:9.11.26-6.el8.x86_64 is already installed.
Dependencies resolved.
============================================================================================================================================
 Package               Architecture            Version                              Repository                                         Size
============================================================================================================================================
Installing:
 bind                  x86_64                  32:9.11.26-6.el8                     rhel-8-for-x86_64-appstream-rpms                  2.1 M

Transaction Summary
============================================================================================================================================
Install  1 Package

Total download size: 2.1 M
Installed size: 4.5 M
Is this ok [y/N]: y
Downloading Packages:
bind-9.11.26-6.el8.x86_64.rpm                                                                               3.6 MB/s | 2.1 MB     00:00    
--------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                       3.6 MB/s | 2.1 MB     00:00     
Red Hat Enterprise Linux 8 for x86_64 - AppStream (RPMs)                                                    4.9 MB/s | 5.0 kB     00:00    
Importing GPG key 0xFD431D51:
 Userid     : "Red Hat, Inc. (release key 2) <security@redhat.com>"
 Fingerprint: 567E 347A D004 4ADE 55BA 8A5F 199E 2F91 FD43 1D51
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
Is this ok [y/N]: y
Key imported successfully
Importing GPG key 0xD4082792:
 Userid     : "Red Hat, Inc. (auxiliary key) <security@redhat.com>"
 Fingerprint: 6A6A A7C9 7C88 90AE C6AE BFE2 F76F 66C3 D408 2792
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
Is this ok [y/N]: y
Key imported successfully
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                                    1/1 
  Running scriptlet: bind-32:9.11.26-6.el8.x86_64                                                                                       1/1 
  Installing       : bind-32:9.11.26-6.el8.x86_64                                                                                       1/1 
  Running scriptlet: bind-32:9.11.26-6.el8.x86_64                                                                                       1/1 
  Verifying        : bind-32:9.11.26-6.el8.x86_64                                                                                       1/1 
Installed products updated.

Installed:
  bind-32:9.11.26-6.el8.x86_64                                                                                                              

Complete!
[root@tektutor ~]# 
</pre>

##### Configuring DNS Server
In my case my RedHat 8.5 VM IP was 192.168.167.140, you may have to note down your IP address so that you can replace 192.168.167.140 with your IP address for the DNS Server configuration.

You need to edit /etc/named.conf as highlighted in bold below
<pre>
// named.conf
//
// Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
// server as a caching only nameserver (as a localhost DNS resolver only).
//
// See /usr/share/doc/bind*/sample/ for example named configuration files.
//

options {
<b>//      listen-on port 53 { 127.0.0.1; };</b>
<b>//      listen-on-v6 port 53 { ::1; };</b>
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        secroots-file   "/var/named/data/named.secroots";
        recursing-file  "/var/named/data/named.recursing";
        allow-query     { localhost; <b>192.168.167.0/24;</b> };

        /* 
         - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
         - If you are building a RECURSIVE (caching) DNS server, you need to enable 
           recursion. 
         - If your recursive DNS server has a public IP address, you MUST enable access 
           control to limit queries to your legitimate users. Failing to do so will
           cause your server to become part of large scale DNS amplification 
           attacks. Implementing BCP38 within your network would greatly
           reduce such attack surface 
        */
        recursion yes;
        
 
        dnssec-enable yes;
        dnssec-validation yes;

        managed-keys-directory "/var/named/dynamic";

        pid-file "/run/named/named.pid";
        session-keyfile "/run/named/session.key";

        /* https://fedoraproject.org/wiki/Changes/CryptoPolicy */
        include "/etc/crypto-policies/back-ends/bind.config";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};
<b>       
//forward zone
zone "okd.org" IN {
     type master;
     file "okd.org.db";
     allow-update { none; };
     allow-query { any; };
};

//backward zone
zone "167.168.192.in-addr.arpa" IN {
     type master;
     file "okd.org.rev";
     allow-update { none; };
     allow-query { any; };
};
</b>
</pre>


Starting DNS Service
```
sudo systemctl start named
sudo systemctl enable named
sudo systemctl status named
```

The expected output is
<pre>
Starting DNS Service
```
[root@tektutor ~]# <b>systemctl start named</b>
[root@tektutor ~]# <b>systemctl enable named</b>
Created symlink /etc/systemd/system/multi-user.target.wants/named.service → /usr/lib/systemd/system/named.service.
[root@tektutor ~]# <b>systemctl status named</b>
● named.service - Berkeley Internet Name Domain (DNS)
   Loaded: loaded (/usr/lib/systemd/system/named.service; enabled; vendor preset: disabled)
   Active: active (running) since Sat 2022-02-12 14:56:34 PST; 21s ago
 Main PID: 38264 (named)
    Tasks: 27 (limit: 410178)
   Memory: 81.0M
   CGroup: /system.slice/named.service
           └─38264 /usr/sbin/named -u named -c /etc/named.conf

Feb 12 14:56:34 tektutor named[38264]: network unreachable resolving './DNSKEY/IN': 2001:503:ba3e::2:30#53
Feb 12 14:56:34 tektutor named[38264]: network unreachable resolving './NS/IN': 2001:503:ba3e::2:30#53
Feb 12 14:56:34 tektutor named[38264]: network unreachable resolving './DNSKEY/IN': 2001:500:a8::e#53
Feb 12 14:56:34 tektutor named[38264]: network unreachable resolving './NS/IN': 2001:500:a8::e#53
Feb 12 14:56:34 tektutor named[38264]: network unreachable resolving './DNSKEY/IN': 2001:500:2f::f#53
Feb 12 14:56:34 tektutor named[38264]: network unreachable resolving './NS/IN': 2001:500:2f::f#53
Feb 12 14:56:34 tektutor named[38264]: network unreachable resolving './DNSKEY/IN': 2001:500:2::c#53
Feb 12 14:56:34 tektutor named[38264]: network unreachable resolving './NS/IN': 2001:500:2::c#53
Feb 12 14:56:34 tektutor named[38264]: managed-keys-zone: Key 20326 for zone . acceptance timer complete: key now trusted
Feb 12 14:56:34 tektutor named[38264]: resolver priming query complete
</pre>

### Installing ovirt hypervisor in RHEL 8.5
Make sure VT-x or AMD-v is enabled on the BIOS if RHEL 8.5 is setup as the base Operating System.

In case you have setup RHEL 8.5 as a Virtual Machine, make sure nested VM is enabled.  I enabled VT-X and IOMMU on my VMWare Workstation.

```
dnf -y update
```

The expected output is
<pre>
[root@tektutor ~]# dnf -y update
Updating Subscription Management repositories.
Last metadata expiration check: 0:07:11 ago on Fri 11 Feb 2022 07:23:19 PM PST.
Dependencies resolved.
==================================================================================
 Package          Arch   Version           Repository                        Size
==================================================================================
Installing:
 kernel           x86_64 4.18.0-348.12.2.el8_5
                                           rhel-8-for-x86_64-baseos-rpms    7.0 M
 kernel-devel     x86_64 4.18.0-348.12.2.el8_5
                                           rhel-8-for-x86_64-baseos-rpms     20 M
Upgrading:
 accountsservice  x86_64 0.6.55-2.el8_5.2  rhel-8-for-x86_64-appstream-rpms 139 k
 accountsservice-libs
                  x86_64 0.6.55-2.el8_5.2  rhel-8-for-x86_64-appstream-rpms  96 k
 binutils         x86_64 2.30-108.el8_5.1  rhel-8-for-x86_64-baseos-rpms    5.8 M
 bpftool          x86_64 4.18.0-348.12.2.el8_5
                                           rhel-8-for-x86_64-baseos-rpms    7.7 M
 clevis           x86_64 15-1.el8_5.1      rhel-8-for-x86_64-appstream-rpms  57 k
 clevis-luks      x86_64 15-1.el8_5.1      rhel-8-for-x86_64-appstream-rpms  37 k
 cockpit          x86_64 251.3-1.el8_5     rhel-8-for-x86_64-baseos-rpms     78 k
 cockpit-bridge   x86_64 251.3-1.el8_5     rhel-8-for-x86_64-baseos-rpms    539 k
 cockpit-system   noarch 251.3-1.el8_5     rhel-8-for-x86_64-baseos-rpms    3.2 M
 cockpit-ws       x86_64 251.3-1.el8_5     rhel-8-for-x86_64-baseos-rpms    1.3 M
 cpp              x86_64 8.5.0-4.el8_5     rhel-8-for-x86_64-appstream-rpms  10 M
 cryptsetup       x86_64 2.3.3-4.el8_5.1   rhel-8-for-x86_64-baseos-rpms    190 k
 cryptsetup-libs  x86_64 2.3.3-4.el8_5.1   rhel-8-for-x86_64-baseos-rpms    474 k
 dnf-plugins-core noarch 4.0.21-4.el8_5    rhel-8-for-x86_64-baseos-rpms     70 k
 firefox          x86_64 91.5.0-1.el8_5    rhel-8-for-x86_64-appstream-rpms 106 M
 flatpak          x86_64 1.8.5-5.el8_5     rhel-8-for-x86_64-appstream-rpms 1.6 M
 flatpak-libs     x86_64 1.8.5-5.el8_5     rhel-8-for-x86_64-appstream-rpms 440 k
 flatpak-selinux  noarch 1.8.5-5.el8_5     rhel-8-for-x86_64-appstream-rpms  27 k
 flatpak-session-helper
                  x86_64 1.8.5-5.el8_5     rhel-8-for-x86_64-appstream-rpms  75 k
 freerdp-libs     x86_64 2:2.2.0-7.el8_5   rhel-8-for-x86_64-appstream-rpms 892 k
 fwupd            x86_64 1.5.9-1.el8_4     rhel-8-for-x86_64-baseos-rpms    2.8 M
 gcc              x86_64 8.5.0-4.el8_5     rhel-8-for-x86_64-appstream-rpms  23 M
 gnome-classic-session
                  noarch 3.32.1-22.el8_5   rhel-8-for-x86_64-appstream-rpms  48 k
 gnome-control-center
                  x86_64 3.28.2-29.el8_5   rhel-8-for-x86_64-appstream-rpms 5.4 M
 gnome-control-center-filesystem
                  noarch 3.28.2-29.el8_5   rhel-8-for-x86_64-appstream-rpms  12 k
 gnome-shell-extension-apps-menu
                  noarch 3.32.1-22.el8_5   rhel-8-for-x86_64-appstream-rpms  32 k
 gnome-shell-extension-common
                  noarch 3.32.1-22.el8_5   rhel-8-for-x86_64-appstream-rpms 171 k
 gnome-shell-extension-desktop-icons
                  noarch 3.32.1-22.el8_5   rhel-8-for-x86_64-appstream-rpms  50 k
 gnome-shell-extension-horizontal-workspaces
                  noarch 3.32.1-22.el8_5   rhel-8-for-x86_64-appstream-rpms  26 k
 gnome-shell-extension-launch-new-instance
                  noarch 3.32.1-22.el8_5   rhel-8-for-x86_64-appstream-rpms  27 k
 gnome-shell-extension-places-menu
                  noarch 3.32.1-22.el8_5   rhel-8-for-x86_64-appstream-rpms  32 k
 gnome-shell-extension-window-list
                  noarch 3.32.1-22.el8_5   rhel-8-for-x86_64-appstream-rpms  40 k
 ibus             x86_64 1.5.19-14.el8_5   rhel-8-for-x86_64-appstream-rpms 9.1 M
 ibus-gtk2        x86_64 1.5.19-14.el8_5   rhel-8-for-x86_64-appstream-rpms  62 k
 ibus-gtk3        x86_64 1.5.19-14.el8_5   rhel-8-for-x86_64-appstream-rpms  63 k
 ibus-libs        x86_64 1.5.19-14.el8_5   rhel-8-for-x86_64-appstream-rpms 265 k
 ibus-setup       noarch 1.5.19-14.el8_5   rhel-8-for-x86_64-appstream-rpms  97 k
 insights-client  noarch 3.1.7-1.el8_5     rhel-8-for-x86_64-appstream-rpms 1.2 M
 java-1.8.0-openjdk
                  x86_64 1:1.8.0.322.b06-2.el8_5
                                           rhel-8-for-x86_64-appstream-rpms 342 k
 java-1.8.0-openjdk-headless
                  x86_64 1:1.8.0.322.b06-2.el8_5
                                           rhel-8-for-x86_64-appstream-rpms  34 M
 kernel-headers   x86_64 4.18.0-348.12.2.el8_5
                                           rhel-8-for-x86_64-baseos-rpms    8.3 M
 kernel-tools     x86_64 4.18.0-348.12.2.el8_5
                                           rhel-8-for-x86_64-baseos-rpms    7.2 M
 kernel-tools-libs
                  x86_64 4.18.0-348.12.2.el8_5
                                           rhel-8-for-x86_64-baseos-rpms    7.0 M
 kexec-tools      x86_64 2.0.20-57.el8_5.1 rhel-8-for-x86_64-baseos-rpms    514 k
 libgcc           x86_64 8.5.0-4.el8_5     rhel-8-for-x86_64-baseos-rpms     80 k
 libgomp          x86_64 8.5.0-4.el8_5     rhel-8-for-x86_64-baseos-rpms    206 k
 libipa_hbac      x86_64 2.5.2-2.el8_5.4   rhel-8-for-x86_64-baseos-rpms    116 k
 libsmbclient     x86_64 4.14.5-9.el8_5    rhel-8-for-x86_64-baseos-rpms    148 k
 libsss_autofs    x86_64 2.5.2-2.el8_5.4   rhel-8-for-x86_64-baseos-rpms    119 k
 libsss_certmap   x86_64 2.5.2-2.el8_5.4   rhel-8-for-x86_64-baseos-rpms    156 k
 libsss_idmap     x86_64 2.5.2-2.el8_5.4   rhel-8-for-x86_64-baseos-rpms    121 k
 libsss_nss_idmap x86_64 2.5.2-2.el8_5.4   rhel-8-for-x86_64-baseos-rpms    128 k
 libsss_sudo      x86_64 2.5.2-2.el8_5.4   rhel-8-for-x86_64-baseos-rpms    117 k
 libstdc++        x86_64 8.5.0-4.el8_5     rhel-8-for-x86_64-baseos-rpms    453 k
 libvirt-daemon   x86_64 6.0.0-37.1.module+el8.5.0+13858+39fdc467
                                           rhel-8-for-x86_64-appstream-rpms 351 k
 libvirt-daemon-config-network
                  x86_64 6.0.0-37.1.module+el8.5.0+13858+39fdc467
                                           rhel-8-for-x86_64-appstream-rpms  63 k
 libvirt-daemon-driver-interface
                  x86_64 6.0.0-37.1.module+el8.5.0+13858+39fdc467
                                           rhel-8-for-x86_64-appstream-rpms 209 k
 libvirt-daemon-driver-network
                  x86_64 6.0.0-37.1.module+el8.5.0+13858+39fdc467
                                           rhel-8-for-x86_64-appstream-rpms 236 k
 libvirt-daemon-driver-nodedev
                  x86_64 6.0.0-37.1.module+el8.5.0+13858+39fdc467
                                           rhel-8-for-x86_64-appstream-rpms 208 k
 libvirt-daemon-driver-nwfilter
                  x86_64 6.0.0-37.1.module+el8.5.0+13858+39fdc467
                                           rhel-8-for-x86_64-appstream-rpms 232 k
 libvirt-daemon-driver-qemu
                  x86_64 6.0.0-37.1.module+el8.5.0+13858+39fdc467
                                           rhel-8-for-x86_64-appstream-rpms 844 k
 libvirt-daemon-driver-secret
                  x86_64 6.0.0-37.1.module+el8.5.0+13858+39fdc467
                                           rhel-8-for-x86_64-appstream-rpms 198 k
 libvirt-daemon-driver-storage
                  x86_64 6.0.0-37.1.module+el8.5.0+13858+39fdc467
                                           rhel-8-for-x86_64-appstream-rpms  61 k
 libvirt-daemon-driver-storage-core
                  x86_64 6.0.0-37.1.module+el8.5.0+13858+39fdc467
                                           rhel-8-for-x86_64-appstream-rpms 259 k
 libvirt-daemon-driver-storage-disk
                  x86_64 6.0.0-37.1.module+el8.5.0+13858+39fdc467
                                           rhel-8-for-x86_64-appstream-rpms  82 k
 libvirt-daemon-driver-storage-gluster
                  x86_64 6.0.0-37.1.module+el8.5.0+13858+39fdc467
                                           rhel-8-for-x86_64-appstream-rpms  87 k
 libvirt-daemon-driver-storage-iscsi
                  x86_64 6.0.0-37.1.module+el8.5.0+13858+39fdc467
                                           rhel-8-for-x86_64-appstream-rpms  79 k
 libvirt-daemon-driver-storage-iscsi-direct
                  x86_64 6.0.0-37.1.module+el8.5.0+13858+39fdc467
                                           rhel-8-for-x86_64-appstream-rpms  81 k
 libvirt-daemon-driver-storage-logical
                  x86_64 6.0.0-37.1.module+el8.5.0+13858+39fdc467
                                           rhel-8-for-x86_64-appstream-rpms  83 k
 libvirt-daemon-driver-storage-mpath
                  x86_64 6.0.0-37.1.module+el8.5.0+13858+39fdc467
                                           rhel-8-for-x86_64-appstream-rpms  77 k
 libvirt-daemon-driver-storage-rbd
                  x86_64 6.0.0-37.1.module+el8.5.0+13858+39fdc467
                                           rhel-8-for-x86_64-appstream-rpms  87 k
 libvirt-daemon-driver-storage-scsi
                  x86_64 6.0.0-37.1.module+el8.5.0+13858+39fdc467
                                           rhel-8-for-x86_64-appstream-rpms  79 k
 libvirt-daemon-kvm
                  x86_64 6.0.0-37.1.module+el8.5.0+13858+39fdc467
                                           rhel-8-for-x86_64-appstream-rpms  61 k
 libvirt-libs     x86_64 6.0.0-37.1.module+el8.5.0+13858+39fdc467
                                           rhel-8-for-x86_64-appstream-rpms 4.3 M
 libwbclient      x86_64 4.14.5-9.el8_5    rhel-8-for-x86_64-baseos-rpms    122 k
 libwinpr         x86_64 2:2.2.0-7.el8_5   rhel-8-for-x86_64-appstream-rpms 357 k
 nss              x86_64 3.67.0-7.el8_5    rhel-8-for-x86_64-appstream-rpms 741 k
 nss-softokn      x86_64 3.67.0-7.el8_5    rhel-8-for-x86_64-appstream-rpms 487 k
 nss-softokn-freebl
                  x86_64 3.67.0-7.el8_5    rhel-8-for-x86_64-appstream-rpms 395 k
 nss-sysinit      x86_64 3.67.0-7.el8_5    rhel-8-for-x86_64-appstream-rpms  73 k
 nss-util         x86_64 3.67.0-7.el8_5    rhel-8-for-x86_64-appstream-rpms 137 k
 openssl          x86_64 1:1.1.1k-5.el8_5  rhel-8-for-x86_64-baseos-rpms    709 k
 openssl-libs     x86_64 1:1.1.1k-5.el8_5  rhel-8-for-x86_64-baseos-rpms    1.5 M
 ostree           x86_64 2021.3-2.el8_5    rhel-8-for-x86_64-appstream-rpms 247 k
 ostree-libs      x86_64 2021.3-2.el8_5    rhel-8-for-x86_64-appstream-rpms 432 k
 polkit           x86_64 0.115-13.el8_5.1  rhel-8-for-x86_64-baseos-rpms    154 k
 polkit-libs      x86_64 0.115-13.el8_5.1  rhel-8-for-x86_64-baseos-rpms     76 k
 poppler          x86_64 20.11.0-3.el8_5.1 rhel-8-for-x86_64-appstream-rpms 1.1 M
 poppler-glib     x86_64 20.11.0-3.el8_5.1 rhel-8-for-x86_64-appstream-rpms 174 k
 poppler-utils    x86_64 20.11.0-3.el8_5.1 rhel-8-for-x86_64-appstream-rpms 248 k
 ppp              x86_64 2.4.7-26.el8_1    rhel-8-for-x86_64-baseos-rpms    407 k
 python3-dnf-plugins-core
                  noarch 4.0.21-4.el8_5    rhel-8-for-x86_64-baseos-rpms    234 k
 python3-perf     x86_64 4.18.0-348.12.2.el8_5
                                           rhel-8-for-x86_64-baseos-rpms    7.1 M
 python3-rpm      x86_64 4.14.3-19.el8_5.2 rhel-8-for-x86_64-baseos-rpms    154 k
 python3-sssdconfig
                  noarch 2.5.2-2.el8_5.4   rhel-8-for-x86_64-baseos-rpms    143 k
 qemu-guest-agent x86_64 15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1
                                           rhel-8-for-x86_64-appstream-rpms 257 k
 qemu-img         x86_64 15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1
                                           rhel-8-for-x86_64-appstream-rpms 1.1 M
 qemu-kvm         x86_64 15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1
                                           rhel-8-for-x86_64-appstream-rpms 127 k
 qemu-kvm-block-curl
                  x86_64 15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1
                                           rhel-8-for-x86_64-appstream-rpms 138 k
 qemu-kvm-block-gluster
                  x86_64 15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1
                                           rhel-8-for-x86_64-appstream-rpms 139 k
 qemu-kvm-block-iscsi
                  x86_64 15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1
                                           rhel-8-for-x86_64-appstream-rpms 145 k
 qemu-kvm-block-rbd
                  x86_64 15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1
                                           rhel-8-for-x86_64-appstream-rpms 139 k
 qemu-kvm-block-ssh
                  x86_64 15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1
                                           rhel-8-for-x86_64-appstream-rpms 140 k
 qemu-kvm-common  x86_64 15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1
                                           rhel-8-for-x86_64-appstream-rpms 1.2 M
 qemu-kvm-core    x86_64 15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1
                                           rhel-8-for-x86_64-appstream-rpms 3.5 M
 rpm              x86_64 4.14.3-19.el8_5.2 rhel-8-for-x86_64-baseos-rpms    543 k
 rpm-build-libs   x86_64 4.14.3-19.el8_5.2 rhel-8-for-x86_64-baseos-rpms    157 k
 rpm-libs         x86_64 4.14.3-19.el8_5.2 rhel-8-for-x86_64-baseos-rpms    345 k
 rpm-plugin-selinux
                  x86_64 4.14.3-19.el8_5.2 rhel-8-for-x86_64-baseos-rpms     77 k
 rpm-plugin-systemd-inhibit
                  x86_64 4.14.3-19.el8_5.2 rhel-8-for-x86_64-baseos-rpms     78 k
 samba-client     x86_64 4.14.5-9.el8_5    rhel-8-for-x86_64-baseos-rpms    702 k
 samba-client-libs
                  x86_64 4.14.5-9.el8_5    rhel-8-for-x86_64-baseos-rpms    5.4 M
 samba-common     noarch 4.14.5-9.el8_5    rhel-8-for-x86_64-baseos-rpms    221 k
 samba-common-libs
                  x86_64 4.14.5-9.el8_5    rhel-8-for-x86_64-baseos-rpms    174 k
 selinux-policy   noarch 3.14.3-80.el8_5.2 rhel-8-for-x86_64-baseos-rpms    636 k
 selinux-policy-targeted
                  noarch 3.14.3-80.el8_5.2 rhel-8-for-x86_64-baseos-rpms     15 M
 sos              noarch 4.1-9.el8_5       rhel-8-for-x86_64-baseos-rpms    709 k
 sssd             x86_64 2.5.2-2.el8_5.4   rhel-8-for-x86_64-baseos-rpms    108 k
 sssd-ad          x86_64 2.5.2-2.el8_5.4   rhel-8-for-x86_64-baseos-rpms    271 k
 sssd-client      x86_64 2.5.2-2.el8_5.4   rhel-8-for-x86_64-baseos-rpms    206 k
 sssd-common      x86_64 2.5.2-2.el8_5.4   rhel-8-for-x86_64-baseos-rpms    1.6 M
 sssd-common-pac  x86_64 2.5.2-2.el8_5.4   rhel-8-for-x86_64-baseos-rpms    179 k
 sssd-ipa         x86_64 2.5.2-2.el8_5.4   rhel-8-for-x86_64-baseos-rpms    348 k
 sssd-kcm         x86_64 2.5.2-2.el8_5.4   rhel-8-for-x86_64-baseos-rpms    255 k
 sssd-krb5        x86_64 2.5.2-2.el8_5.4   rhel-8-for-x86_64-baseos-rpms    151 k
 sssd-krb5-common x86_64 2.5.2-2.el8_5.4   rhel-8-for-x86_64-baseos-rpms    186 k
 sssd-ldap        x86_64 2.5.2-2.el8_5.4   rhel-8-for-x86_64-baseos-rpms    209 k
 sssd-nfs-idmap   x86_64 2.5.2-2.el8_5.4   rhel-8-for-x86_64-baseos-rpms    116 k
 sssd-proxy       x86_64 2.5.2-2.el8_5.4   rhel-8-for-x86_64-baseos-rpms    148 k
 sudo             x86_64 1.8.29-7.el8_4.1  rhel-8-for-x86_64-baseos-rpms    925 k
 systemd          x86_64 239-51.el8_5.3    rhel-8-for-x86_64-baseos-rpms    3.6 M
 systemd-container
                  x86_64 239-51.el8_5.3    rhel-8-for-x86_64-baseos-rpms    752 k
 systemd-libs     x86_64 239-51.el8_5.3    rhel-8-for-x86_64-baseos-rpms    1.1 M
 systemd-pam      x86_64 239-51.el8_5.3    rhel-8-for-x86_64-baseos-rpms    477 k
 systemd-udev     x86_64 239-51.el8_5.3    rhel-8-for-x86_64-baseos-rpms    1.6 M
 tcpdump          x86_64 14:4.9.3-2.el8_5.1
                                           rhel-8-for-x86_64-appstream-rpms 453 k
 tigervnc-license noarch 1.11.0-10.el8_5   rhel-8-for-x86_64-appstream-rpms  39 k
 tigervnc-server-minimal
                  x86_64 1.11.0-10.el8_5   rhel-8-for-x86_64-appstream-rpms 1.1 M
 tzdata           noarch 2021e-1.el8       rhel-8-for-x86_64-baseos-rpms    474 k
 tzdata-java      noarch 2021e-1.el8       rhel-8-for-x86_64-appstream-rpms 191 k
 unzip            x86_64 6.0-45.el8_4      rhel-8-for-x86_64-baseos-rpms    195 k
 vim-common       x86_64 2:8.0.1763-16.el8_5.4
                                           rhel-8-for-x86_64-appstream-rpms 6.3 M
 vim-enhanced     x86_64 2:8.0.1763-16.el8_5.4
                                           rhel-8-for-x86_64-appstream-rpms 1.4 M
 vim-filesystem   noarch 2:8.0.1763-16.el8_5.4
                                           rhel-8-for-x86_64-appstream-rpms  49 k
 vim-minimal      x86_64 2:8.0.1763-16.el8_5.4
                                           rhel-8-for-x86_64-baseos-rpms    574 k
Installing dependencies:
 kernel-core      x86_64 4.18.0-348.12.2.el8_5
                                           rhel-8-for-x86_64-baseos-rpms     38 M
 kernel-modules   x86_64 4.18.0-348.12.2.el8_5
                                           rhel-8-for-x86_64-baseos-rpms     30 M

Transaction Summary
==================================================================================
Install    4 Packages
Upgrade  145 Packages

Total download size: 412 M
Downloading Packages:
(1/149): kernel-4.18.0-348.12.2.el8_5.x86_64.rpm  5.4 MB/s | 7.0 MB     00:01    
(2/149): kernel-devel-4.18.0-348.12.2.el8_5.x86_6 6.0 MB/s |  20 MB     00:03    
(3/149): kernel-modules-4.18.0-348.12.2.el8_5.x86 8.4 MB/s |  30 MB     00:03    
(4/149): tzdata-java-2021e-1.el8.noarch.rpm       500 kB/s | 191 kB     00:00    
(5/149): flatpak-session-helper-1.8.5-5.el8_5.x86 199 kB/s |  75 kB     00:00    
(6/149): flatpak-1.8.5-5.el8_5.x86_64.rpm         2.7 MB/s | 1.6 MB     00:00    
(7/149): ibus-gtk2-1.5.19-14.el8_5.x86_64.rpm     176 kB/s |  62 kB     00:00    
(8/149): ibus-setup-1.5.19-14.el8_5.noarch.rpm    281 kB/s |  97 kB     00:00    
(9/149): kernel-core-4.18.0-348.12.2.el8_5.x86_64  11 MB/s |  38 MB     00:03    
(10/149): flatpak-libs-1.8.5-5.el8_5.x86_64.rpm   1.2 MB/s | 440 kB     00:00    
(11/149): ibus-gtk3-1.5.19-14.el8_5.x86_64.rpm    177 kB/s |  63 kB     00:00    
(12/149): ibus-libs-1.5.19-14.el8_5.x86_64.rpm    799 kB/s | 265 kB     00:00    
(13/149): insights-client-3.1.7-1.el8_5.noarch.rp 2.7 MB/s | 1.2 MB     00:00    
(14/149): flatpak-selinux-1.8.5-5.el8_5.noarch.rp  78 kB/s |  27 kB     00:00    
(15/149): ibus-1.5.19-14.el8_5.x86_64.rpm          11 MB/s | 9.1 MB     00:00    
(16/149): libwinpr-2.2.0-7.el8_5.x86_64.rpm       898 kB/s | 357 kB     00:00    
(17/149): cpp-8.5.0-4.el8_5.x86_64.rpm             11 MB/s |  10 MB     00:00    
(18/149): freerdp-libs-2.2.0-7.el8_5.x86_64.rpm   2.0 MB/s | 892 kB     00:00    
(19/149): nss-softokn-3.67.0-7.el8_5.x86_64.rpm   1.1 MB/s | 487 kB     00:00    
(20/149): nss-sysinit-3.67.0-7.el8_5.x86_64.rpm   214 kB/s |  73 kB     00:00    
(21/149): nss-softokn-freebl-3.67.0-7.el8_5.x86_6 1.1 MB/s | 395 kB     00:00    
(22/149): gcc-8.5.0-4.el8_5.x86_64.rpm             13 MB/s |  23 MB     00:01    
(23/149): nss-util-3.67.0-7.el8_5.x86_64.rpm      352 kB/s | 137 kB     00:00    
(24/149): nss-3.67.0-7.el8_5.x86_64.rpm           1.5 MB/s | 741 kB     00:00    
(25/149): qemu-guest-agent-4.2.0-59.module+el8.5. 762 kB/s | 257 kB     00:00    
(26/149): qemu-kvm-4.2.0-59.module+el8.5.0+13495+ 362 kB/s | 127 kB     00:00    
(27/149): accountsservice-libs-0.6.55-2.el8_5.2.x 296 kB/s |  96 kB     00:00    
(28/149): gnome-classic-session-3.32.1-22.el8_5.n 149 kB/s |  48 kB     00:00    
(29/149): qemu-kvm-block-gluster-4.2.0-59.module+ 417 kB/s | 139 kB     00:00    
(30/149): qemu-kvm-block-ssh-4.2.0-59.module+el8. 368 kB/s | 140 kB     00:00    
(31/149): poppler-utils-20.11.0-3.el8_5.1.x86_64. 274 kB/s | 248 kB     00:00    
(32/149): gnome-shell-extension-desktop-icons-3.3 154 kB/s |  50 kB     00:00    
(33/149): gnome-shell-extension-horizontal-worksp  81 kB/s |  26 kB     00:00    
(34/149): poppler-glib-20.11.0-3.el8_5.1.x86_64.r 494 kB/s | 174 kB     00:00    
(35/149): gnome-shell-extension-apps-menu-3.32.1-  98 kB/s |  32 kB     00:00    
(36/149): qemu-kvm-block-curl-4.2.0-59.module+el8 423 kB/s | 138 kB     00:00    
(37/149): gnome-shell-extension-launch-new-instan  84 kB/s |  27 kB     00:00    
(38/149): poppler-20.11.0-3.el8_5.1.x86_64.rpm    2.5 MB/s | 1.1 MB     00:00    
(39/149): qemu-kvm-block-rbd-4.2.0-59.module+el8. 430 kB/s | 139 kB     00:00    
(40/149): gnome-shell-extension-window-list-3.32.  53 kB/s |  40 kB     00:00    
(41/149): qemu-kvm-core-4.2.0-59.module+el8.5.0+1 7.3 MB/s | 3.5 MB     00:00    
(42/149): gnome-shell-extension-places-menu-3.32.  76 kB/s |  32 kB     00:00    
(43/149): qemu-kvm-common-4.2.0-59.module+el8.5.0 3.3 MB/s | 1.2 MB     00:00    
(44/149): accountsservice-0.6.55-2.el8_5.2.x86_64 259 kB/s | 139 kB     00:00    
(45/149): qemu-kvm-block-iscsi-4.2.0-59.module+el 348 kB/s | 145 kB     00:00    
(46/149): qemu-img-4.2.0-59.module+el8.5.0+13495+ 3.0 MB/s | 1.1 MB     00:00    
(47/149): gnome-shell-extension-common-3.32.1-22. 495 kB/s | 171 kB     00:00    
(48/149): java-1.8.0-openjdk-1.8.0.322.b06-2.el8_ 712 kB/s | 342 kB     00:00    
(49/149): clevis-15-1.el8_5.1.x86_64.rpm          157 kB/s |  57 kB     00:00    
(50/149): libvirt-daemon-driver-nwfilter-6.0.0-37 579 kB/s | 232 kB     00:00    
(51/149): tigervnc-server-minimal-1.11.0-10.el8_5 2.5 MB/s | 1.1 MB     00:00    
(52/149): libvirt-daemon-driver-network-6.0.0-37. 669 kB/s | 236 kB     00:00    
(53/149): libvirt-daemon-config-network-6.0.0-37. 182 kB/s |  63 kB     00:00    
(54/149): clevis-luks-15-1.el8_5.1.x86_64.rpm     106 kB/s |  37 kB     00:00    
(55/149): libvirt-daemon-driver-qemu-6.0.0-37.1.m 2.0 MB/s | 844 kB     00:00    
(56/149): libvirt-daemon-driver-storage-gluster-6 257 kB/s |  87 kB     00:00    
(57/149): java-1.8.0-openjdk-headless-1.8.0.322.b 8.6 MB/s |  34 MB     00:03    
(58/149): libvirt-daemon-driver-nodedev-6.0.0-37. 444 kB/s | 208 kB     00:00    
(59/149): libvirt-daemon-driver-storage-iscsi-dir 220 kB/s |  81 kB     00:00    
(60/149): ostree-libs-2021.3-2.el8_5.x86_64.rpm   1.0 MB/s | 432 kB     00:00    
(61/149): libvirt-daemon-driver-storage-6.0.0-37. 178 kB/s |  61 kB     00:00    
(62/149): libvirt-daemon-driver-storage-iscsi-6.0 222 kB/s |  79 kB     00:00    
(63/149): libvirt-daemon-driver-interface-6.0.0-3 483 kB/s | 209 kB     00:00    
(64/149): vim-common-8.0.1763-16.el8_5.4.x86_64.r 7.2 MB/s | 6.3 MB     00:00    
(65/149): libvirt-libs-6.0.0-37.1.module+el8.5.0+ 6.9 MB/s | 4.3 MB     00:00    
(66/149): libvirt-daemon-driver-storage-scsi-6.0. 178 kB/s |  79 kB     00:00    
(67/149): libvirt-daemon-driver-storage-disk-6.0. 238 kB/s |  82 kB     00:00    
(68/149): libvirt-daemon-driver-storage-logical-6 232 kB/s |  83 kB     00:00    
(69/149): libvirt-daemon-6.0.0-37.1.module+el8.5. 691 kB/s | 351 kB     00:00    
(70/149): gnome-control-center-filesystem-3.28.2-  35 kB/s |  12 kB     00:00    
(71/149): firefox-91.5.0-1.el8_5.x86_64.rpm        15 MB/s | 106 MB     00:07    
(72/149): libvirt-daemon-driver-storage-mpath-6.0  99 kB/s |  77 kB     00:00    
(73/149): gnome-control-center-3.28.2-29.el8_5.x8 9.4 MB/s | 5.4 MB     00:00    
(74/149): libvirt-daemon-driver-storage-rbd-6.0.0 265 kB/s |  87 kB     00:00    
(75/149): libvirt-daemon-driver-storage-core-6.0. 763 kB/s | 259 kB     00:00    
(76/149): vim-enhanced-8.0.1763-16.el8_5.4.x86_64 3.0 MB/s | 1.4 MB     00:00    
(77/149): ostree-2021.3-2.el8_5.x86_64.rpm        715 kB/s | 247 kB     00:00    
(78/149): tigervnc-license-1.11.0-10.el8_5.noarch 114 kB/s |  39 kB     00:00    
(79/149): libvirt-daemon-kvm-6.0.0-37.1.module+el 194 kB/s |  61 kB     00:00    
(80/149): tcpdump-4.9.3-2.el8_5.1.x86_64.rpm      1.2 MB/s | 453 kB     00:00    
(81/149): libvirt-daemon-driver-secret-6.0.0-37.1 474 kB/s | 198 kB     00:00    
(82/149): vim-filesystem-8.0.1763-16.el8_5.4.noar 146 kB/s |  49 kB     00:00    
(83/149): ppp-2.4.7-26.el8_1.x86_64.rpm           1.0 MB/s | 407 kB     00:00    
(84/149): fwupd-1.5.9-1.el8_4.x86_64.rpm          6.1 MB/s | 2.8 MB     00:00    
(85/149): sudo-1.8.29-7.el8_4.1.x86_64.rpm        2.3 MB/s | 925 kB     00:00    
(86/149): tzdata-2021e-1.el8.noarch.rpm           1.0 MB/s | 474 kB     00:00    
(87/149): unzip-6.0-45.el8_4.x86_64.rpm           249 kB/s | 195 kB     00:00    
(88/149): libgcc-8.5.0-4.el8_5.x86_64.rpm         246 kB/s |  80 kB     00:00    
(89/149): binutils-2.30-108.el8_5.1.x86_64.rpm     10 MB/s | 5.8 MB     00:00    
(90/149): libgomp-8.5.0-4.el8_5.x86_64.rpm        572 kB/s | 206 kB     00:00    
(91/149): libstdc++-8.5.0-4.el8_5.x86_64.rpm      1.3 MB/s | 453 kB     00:00    
(92/149): selinux-policy-3.14.3-80.el8_5.2.noarch 1.8 MB/s | 636 kB     00:00    
(93/149): systemd-239-51.el8_5.3.x86_64.rpm       6.4 MB/s | 3.6 MB     00:00    
(94/149): systemd-container-239-51.el8_5.3.x86_64 1.8 MB/s | 752 kB     00:00    
(95/149): systemd-pam-239-51.el8_5.3.x86_64.rpm   892 kB/s | 477 kB     00:00    
(96/149): selinux-policy-targeted-3.14.3-80.el8_5  12 MB/s |  15 MB     00:01    
(97/149): systemd-udev-239-51.el8_5.3.x86_64.rpm  2.8 MB/s | 1.6 MB     00:00    
(98/149): openssl-1.1.1k-5.el8_5.x86_64.rpm       1.9 MB/s | 709 kB     00:00    
(99/149): kexec-tools-2.0.20-57.el8_5.1.x86_64.rp 1.4 MB/s | 514 kB     00:00    
(100/149): systemd-libs-239-51.el8_5.3.x86_64.rpm 1.7 MB/s | 1.1 MB     00:00    
(101/149): openssl-libs-1.1.1k-5.el8_5.x86_64.rpm 3.6 MB/s | 1.5 MB     00:00    
(102/149): kernel-tools-4.18.0-348.12.2.el8_5.x86 8.1 MB/s | 7.2 MB     00:00    
(103/149): kernel-tools-libs-4.18.0-348.12.2.el8_ 7.0 MB/s | 7.0 MB     00:01    
(104/149): bpftool-4.18.0-348.12.2.el8_5.x86_64.r 8.3 MB/s | 7.7 MB     00:00    
(105/149): kernel-headers-4.18.0-348.12.2.el8_5.x  11 MB/s | 8.3 MB     00:00    
(106/149): polkit-0.115-13.el8_5.1.x86_64.rpm     444 kB/s | 154 kB     00:00    
(107/149): python3-perf-4.18.0-348.12.2.el8_5.x86 9.1 MB/s | 7.1 MB     00:00    
(108/149): polkit-libs-0.115-13.el8_5.1.x86_64.rp 231 kB/s |  76 kB     00:00    
(109/149): samba-client-4.14.5-9.el8_5.x86_64.rpm 2.0 MB/s | 702 kB     00:00    
(110/149): samba-common-4.14.5-9.el8_5.noarch.rpm 674 kB/s | 221 kB     00:00    
(111/149): libsmbclient-4.14.5-9.el8_5.x86_64.rpm 444 kB/s | 148 kB     00:00    
(112/149): samba-common-libs-4.14.5-9.el8_5.x86_6 531 kB/s | 174 kB     00:00    
(113/149): sssd-krb5-2.5.2-2.el8_5.4.x86_64.rpm   448 kB/s | 151 kB     00:00    
(114/149): samba-client-libs-4.14.5-9.el8_5.x86_6 9.5 MB/s | 5.4 MB     00:00    
(115/149): libwbclient-4.14.5-9.el8_5.x86_64.rpm  189 kB/s | 122 kB     00:00    
(116/149): cryptsetup-2.3.3-4.el8_5.1.x86_64.rpm  579 kB/s | 190 kB     00:00    
(117/149): sssd-client-2.5.2-2.el8_5.4.x86_64.rpm 599 kB/s | 206 kB     00:00    
(118/149): sos-4.1-9.el8_5.noarch.rpm             1.9 MB/s | 709 kB     00:00    
(119/149): libsss_sudo-2.5.2-2.el8_5.4.x86_64.rpm 358 kB/s | 117 kB     00:00    
(120/149): libsss_idmap-2.5.2-2.el8_5.4.x86_64.rp 369 kB/s | 121 kB     00:00    
(121/149): sssd-ad-2.5.2-2.el8_5.4.x86_64.rpm     788 kB/s | 271 kB     00:00    
(122/149): sssd-kcm-2.5.2-2.el8_5.4.x86_64.rpm    755 kB/s | 255 kB     00:00    
(123/149): rpm-plugin-systemd-inhibit-4.14.3-19.e 233 kB/s |  78 kB     00:00    
(124/149): cockpit-system-251.3-1.el8_5.noarch.rp 6.9 MB/s | 3.2 MB     00:00    
(125/149): cockpit-ws-251.3-1.el8_5.x86_64.rpm    3.2 MB/s | 1.3 MB     00:00    
(126/149): sssd-krb5-common-2.5.2-2.el8_5.4.x86_6 558 kB/s | 186 kB     00:00    
(127/149): sssd-nfs-idmap-2.5.2-2.el8_5.4.x86_64. 359 kB/s | 116 kB     00:00    
(128/149): python3-sssdconfig-2.5.2-2.el8_5.4.noa 219 kB/s | 143 kB     00:00    
(129/149): sssd-2.5.2-2.el8_5.4.x86_64.rpm        145 kB/s | 108 kB     00:00    
(130/149): vim-minimal-8.0.1763-16.el8_5.4.x86_64 1.6 MB/s | 574 kB     00:00    
(131/149): rpm-build-libs-4.14.3-19.el8_5.2.x86_6 431 kB/s | 157 kB     00:00    
(132/149): cockpit-bridge-251.3-1.el8_5.x86_64.rp 1.6 MB/s | 539 kB     00:00    
(133/149): sssd-ldap-2.5.2-2.el8_5.4.x86_64.rpm   612 kB/s | 209 kB     00:00    
(134/149): sssd-common-2.5.2-2.el8_5.4.x86_64.rpm 3.7 MB/s | 1.6 MB     00:00    
(135/149): sssd-common-pac-2.5.2-2.el8_5.4.x86_64 326 kB/s | 179 kB     00:00    
(136/149): cockpit-251.3-1.el8_5.x86_64.rpm       214 kB/s |  78 kB     00:00    
(137/149): sssd-ipa-2.5.2-2.el8_5.4.x86_64.rpm    1.0 MB/s | 348 kB     00:00    
(138/149): libsss_nss_idmap-2.5.2-2.el8_5.4.x86_6 369 kB/s | 128 kB     00:00    
(139/149): libsss_autofs-2.5.2-2.el8_5.4.x86_64.r 317 kB/s | 119 kB     00:00    
(140/149): rpm-libs-4.14.3-19.el8_5.2.x86_64.rpm  928 kB/s | 345 kB     00:00    
(141/149): python3-rpm-4.14.3-19.el8_5.2.x86_64.r 226 kB/s | 154 kB     00:00    
(142/149): python3-dnf-plugins-core-4.0.21-4.el8_ 354 kB/s | 234 kB     00:00    
(143/149): cryptsetup-libs-2.3.3-4.el8_5.1.x86_64 1.3 MB/s | 474 kB     00:00    
(144/149): rpm-plugin-selinux-4.14.3-19.el8_5.2.x 180 kB/s |  77 kB     00:00    
(145/149): sssd-proxy-2.5.2-2.el8_5.4.x86_64.rpm  427 kB/s | 148 kB     00:00    
(146/149): dnf-plugins-core-4.0.21-4.el8_5.noarch 110 kB/s |  70 kB     00:00    
(147/149): libsss_certmap-2.5.2-2.el8_5.4.x86_64. 487 kB/s | 156 kB     00:00    
(148/149): rpm-4.14.3-19.el8_5.2.x86_64.rpm       1.5 MB/s | 543 kB     00:00    
(149/149): libipa_hbac-2.5.2-2.el8_5.4.x86_64.rpm 165 kB/s | 116 kB     00:00    
----------------------------------------------------------------------------------
Total                                              14 MB/s | 412 MB     00:29     
Red Hat Enterprise Linux 8 for x86_64 - BaseOS (R 4.9 MB/s | 5.0 kB     00:00    
Importing GPG key 0xFD431D51:
 Userid     : "Red Hat, Inc. (release key 2) <security@redhat.com>"
 Fingerprint: 567E 347A D004 4ADE 55BA 8A5F 199E 2F91 FD43 1D51
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
Key imported successfully
Importing GPG key 0xD4082792:
 Userid     : "Red Hat, Inc. (auxiliary key) <security@redhat.com>"
 Fingerprint: 6A6A A7C9 7C88 90AE C6AE BFE2 F76F 66C3 D408 2792
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
Key imported successfully
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Running scriptlet: java-1.8.0-openjdk-headless-1:1.8.0.322.b06-2.el8_5.x86_64                                                         1/1 
  Preparing        :                                                                                                                    1/1 
  Running scriptlet: libgcc-8.5.0-4.el8_5.x86_64                                                                                        1/1 
  Upgrading        : libgcc-8.5.0-4.el8_5.x86_64                                                                                      1/294 
  Running scriptlet: libgcc-8.5.0-4.el8_5.x86_64                                                                                      1/294 
  Upgrading        : systemd-libs-239-51.el8_5.3.x86_64                                                                               2/294 
  Running scriptlet: systemd-libs-239-51.el8_5.3.x86_64                                                                               2/294 
  Upgrading        : openssl-libs-1:1.1.1k-5.el8_5.x86_64                                                                             3/294 
  Running scriptlet: openssl-libs-1:1.1.1k-5.el8_5.x86_64                                                                             3/294 
  Upgrading        : libstdc++-8.5.0-4.el8_5.x86_64                                                                                   4/294 
  Running scriptlet: libstdc++-8.5.0-4.el8_5.x86_64                                                                                   4/294 
  Upgrading        : libvirt-libs-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                                     5/294 
  Running scriptlet: libvirt-libs-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                                     5/294 
  Upgrading        : polkit-libs-0.115-13.el8_5.1.x86_64                                                                              6/294 
  Running scriptlet: polkit-libs-0.115-13.el8_5.1.x86_64                                                                              6/294 
  Upgrading        : libsss_idmap-2.5.2-2.el8_5.4.x86_64                                                                              7/294 
  Running scriptlet: libsss_idmap-2.5.2-2.el8_5.4.x86_64                                                                              7/294 
  Upgrading        : gnome-shell-extension-common-3.32.1-22.el8_5.noarch                                                              8/294 
  Upgrading        : nss-util-3.67.0-7.el8_5.x86_64                                                                                   9/294 
  Upgrading        : rpm-libs-4.14.3-19.el8_5.2.x86_64                                                                               10/294 
  Running scriptlet: rpm-libs-4.14.3-19.el8_5.2.x86_64                                                                               10/294 
  Upgrading        : rpm-4.14.3-19.el8_5.2.x86_64                                                                                    11/294 
  Upgrading        : libsss_certmap-2.5.2-2.el8_5.4.x86_64                                                                           12/294 
  Running scriptlet: libsss_certmap-2.5.2-2.el8_5.4.x86_64                                                                           12/294 
  Upgrading        : ibus-libs-1.5.19-14.el8_5.x86_64                                                                                13/294 
  Upgrading        : qemu-img-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                                                     14/294 
  Upgrading        : ostree-libs-2021.3-2.el8_5.x86_64                                                                               15/294 
  Upgrading        : cryptsetup-libs-2.3.3-4.el8_5.1.x86_64                                                                          16/294 
  Running scriptlet: cryptsetup-libs-2.3.3-4.el8_5.1.x86_64                                                                          16/294 
  Upgrading        : systemd-pam-239-51.el8_5.3.x86_64                                                                               17/294 
  Running scriptlet: systemd-239-51.el8_5.3.x86_64                                                                                   18/294 
  Upgrading        : systemd-239-51.el8_5.3.x86_64                                                                                   18/294 
  Running scriptlet: systemd-239-51.el8_5.3.x86_64                                                                                   18/294 
  Upgrading        : qemu-kvm-common-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                                              19/294 
  Running scriptlet: qemu-kvm-common-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                                              19/294 
  Running scriptlet: samba-common-4.14.5-9.el8_5.noarch                                                                              20/294 
  Upgrading        : samba-common-4.14.5-9.el8_5.noarch                                                                              20/294 
  Running scriptlet: samba-common-4.14.5-9.el8_5.noarch                                                                              20/294 
  Upgrading        : samba-common-libs-4.14.5-9.el8_5.x86_64                                                                         21/294 
  Upgrading        : libwbclient-4.14.5-9.el8_5.x86_64                                                                               22/294 
  Upgrading        : samba-client-libs-4.14.5-9.el8_5.x86_64                                                                         23/294 
  Upgrading        : libsmbclient-4.14.5-9.el8_5.x86_64                                                                              24/294 
  Running scriptlet: polkit-0.115-13.el8_5.1.x86_64                                                                                  25/294 
  Upgrading        : polkit-0.115-13.el8_5.1.x86_64                                                                                  25/294 
  Running scriptlet: polkit-0.115-13.el8_5.1.x86_64                                                                                  25/294 
  Running scriptlet: libvirt-daemon-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                                  26/294 
  Upgrading        : libvirt-daemon-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                                  26/294 
  Running scriptlet: libvirt-daemon-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                                  26/294 
  Upgrading        : libvirt-daemon-driver-storage-core-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                              27/294 
  Upgrading        : libvirt-daemon-driver-network-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                   28/294 
  Running scriptlet: libvirt-daemon-driver-network-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                   28/294 
  Upgrading        : accountsservice-0.6.55-2.el8_5.2.x86_64                                                                         29/294 
  Running scriptlet: accountsservice-0.6.55-2.el8_5.2.x86_64                                                                         29/294 
  Upgrading        : cockpit-bridge-251.3-1.el8_5.x86_64                                                                             30/294 
  Upgrading        : accountsservice-libs-0.6.55-2.el8_5.2.x86_64                                                                    31/294 
  Upgrading        : libvirt-daemon-driver-storage-gluster-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                           32/294 
  Upgrading        : libvirt-daemon-driver-storage-iscsi-direct-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                      33/294 
  Upgrading        : libvirt-daemon-driver-storage-iscsi-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                             34/294 
  Upgrading        : libvirt-daemon-driver-storage-scsi-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                              35/294 
  Upgrading        : libvirt-daemon-driver-storage-disk-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                              36/294 
  Upgrading        : libvirt-daemon-driver-storage-logical-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                           37/294 
  Upgrading        : libvirt-daemon-driver-storage-mpath-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                             38/294 
  Upgrading        : libvirt-daemon-driver-storage-rbd-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                               39/294 
  Upgrading        : libvirt-daemon-driver-storage-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                   40/294 
  Upgrading        : libvirt-daemon-driver-nwfilter-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                  41/294 
  Upgrading        : libvirt-daemon-driver-nodedev-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                   42/294 
  Upgrading        : libvirt-daemon-driver-interface-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                 43/294 
  Upgrading        : libvirt-daemon-driver-secret-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                    44/294 
  Upgrading        : qemu-kvm-block-gluster-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                                       45/294 
  Upgrading        : qemu-kvm-block-ssh-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                                           46/294 
  Upgrading        : qemu-kvm-block-curl-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                                          47/294 
  Upgrading        : qemu-kvm-core-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                                                48/294 
  Running scriptlet: qemu-kvm-core-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                                                48/294 
  Upgrading        : qemu-kvm-block-rbd-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                                           49/294 
  Upgrading        : qemu-kvm-block-iscsi-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                                         50/294 
  Upgrading        : qemu-kvm-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                                                     51/294 
  Upgrading        : flatpak-session-helper-1.8.5-5.el8_5.x86_64                                                                     52/294 
  Upgrading        : ostree-2021.3-2.el8_5.x86_64                                                                                    53/294 
  Running scriptlet: ostree-2021.3-2.el8_5.x86_64                                                                                    53/294 
  Upgrading        : systemd-container-239-51.el8_5.3.x86_64                                                                         54/294 
  Running scriptlet: libvirt-daemon-driver-qemu-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                      55/294 
  Upgrading        : libvirt-daemon-driver-qemu-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                      55/294 
  Upgrading        : systemd-udev-239-51.el8_5.3.x86_64                                                                              56/294 
  Running scriptlet: systemd-udev-239-51.el8_5.3.x86_64                                                                              56/294 
  Installing       : kernel-core-4.18.0-348.12.2.el8_5.x86_64                                                                        57/294 
  Running scriptlet: kernel-core-4.18.0-348.12.2.el8_5.x86_64                                                                        57/294 
  Installing       : kernel-modules-4.18.0-348.12.2.el8_5.x86_64                                                                     58/294 
  Running scriptlet: kernel-modules-4.18.0-348.12.2.el8_5.x86_64                                                                     58/294 
  Upgrading        : kexec-tools-2.0.20-57.el8_5.1.x86_64                                                                            59/294 
  Running scriptlet: kexec-tools-2.0.20-57.el8_5.1.x86_64                                                                            59/294 
  Upgrading        : cryptsetup-2.3.3-4.el8_5.1.x86_64                                                                               60/294 
  Upgrading        : ibus-gtk2-1.5.19-14.el8_5.x86_64                                                                                61/294 
  Upgrading        : ibus-gtk3-1.5.19-14.el8_5.x86_64                                                                                62/294 
  Upgrading        : ibus-setup-1.5.19-14.el8_5.noarch                                                                               63/294 
  Upgrading        : ibus-1.5.19-14.el8_5.x86_64                                                                                     64/294 
  Running scriptlet: ibus-1.5.19-14.el8_5.x86_64                                                                                     64/294 
  Upgrading        : rpm-build-libs-4.14.3-19.el8_5.2.x86_64                                                                         65/294 
  Running scriptlet: rpm-build-libs-4.14.3-19.el8_5.2.x86_64                                                                         65/294 
  Upgrading        : rpm-plugin-selinux-4.14.3-19.el8_5.2.x86_64                                                                     66/294 
  Upgrading        : selinux-policy-3.14.3-80.el8_5.2.noarch                                                                         67/294 
  Running scriptlet: selinux-policy-3.14.3-80.el8_5.2.noarch                                                                         67/294 
  Running scriptlet: selinux-policy-targeted-3.14.3-80.el8_5.2.noarch                                                                68/294 
  Upgrading        : selinux-policy-targeted-3.14.3-80.el8_5.2.noarch                                                                68/294 
  Running scriptlet: selinux-policy-targeted-3.14.3-80.el8_5.2.noarch                                                                68/294 
  Upgrading        : flatpak-selinux-1.8.5-5.el8_5.noarch                                                                            69/294 
  Running scriptlet: flatpak-selinux-1.8.5-5.el8_5.noarch                                                                            69/294 
  Upgrading        : nss-softokn-freebl-3.67.0-7.el8_5.x86_64                                                                        70/294 
  Upgrading        : nss-softokn-3.67.0-7.el8_5.x86_64                                                                               71/294 
  Upgrading        : nss-sysinit-3.67.0-7.el8_5.x86_64                                                                               72/294 
  Upgrading        : nss-3.67.0-7.el8_5.x86_64                                                                                       73/294 
  Upgrading        : poppler-20.11.0-3.el8_5.1.x86_64                                                                                74/294 
  Running scriptlet: poppler-20.11.0-3.el8_5.1.x86_64                                                                                74/294 
  Upgrading        : gnome-shell-extension-desktop-icons-3.32.1-22.el8_5.noarch                                                      75/294 
  Upgrading        : gnome-shell-extension-horizontal-workspaces-3.32.1-22.el8_5.noarch                                              76/294 
  Upgrading        : gnome-shell-extension-apps-menu-3.32.1-22.el8_5.noarch                                                          77/294 
  Upgrading        : gnome-shell-extension-window-list-3.32.1-22.el8_5.noarch                                                        78/294 
  Upgrading        : gnome-shell-extension-launch-new-instance-3.32.1-22.el8_5.noarch                                                79/294 
  Upgrading        : gnome-shell-extension-places-menu-3.32.1-22.el8_5.noarch                                                        80/294 
  Upgrading        : binutils-2.30-108.el8_5.1.x86_64                                                                                81/294 
  Running scriptlet: binutils-2.30-108.el8_5.1.x86_64                                                                                81/294 
  Upgrading        : libwinpr-2:2.2.0-7.el8_5.x86_64                                                                                 82/294 
  Running scriptlet: clevis-15-1.el8_5.1.x86_64                                                                                      83/294 
  Upgrading        : clevis-15-1.el8_5.1.x86_64                                                                                      83/294 
  Upgrading        : openssl-1:1.1.1k-5.el8_5.x86_64                                                                                 84/294 
  Running scriptlet: cockpit-ws-251.3-1.el8_5.x86_64                                                                                 85/294 
  Upgrading        : cockpit-ws-251.3-1.el8_5.x86_64                                                                                 85/294 
  Running scriptlet: cockpit-ws-251.3-1.el8_5.x86_64                                                                                 85/294 
  Upgrading        : libipa_hbac-2.5.2-2.el8_5.4.x86_64                                                                              86/294 
  Running scriptlet: libipa_hbac-2.5.2-2.el8_5.4.x86_64                                                                              86/294 
  Upgrading        : python3-dnf-plugins-core-4.0.21-4.el8_5.noarch                                                                  87/294 
  Upgrading        : libsss_nss_idmap-2.5.2-2.el8_5.4.x86_64                                                                         88/294 
  Running scriptlet: libsss_nss_idmap-2.5.2-2.el8_5.4.x86_64                                                                         88/294 
  Upgrading        : sssd-client-2.5.2-2.el8_5.4.x86_64                                                                              89/294 
  Running scriptlet: sssd-client-2.5.2-2.el8_5.4.x86_64                                                                              89/294 
  Upgrading        : libsss_autofs-2.5.2-2.el8_5.4.x86_64                                                                            90/294 
  Upgrading        : vim-minimal-2:8.0.1763-16.el8_5.4.x86_64                                                                        91/294 
  Upgrading        : sudo-1.8.29-7.el8_4.1.x86_64                                                                                    92/294 
  Running scriptlet: sudo-1.8.29-7.el8_4.1.x86_64                                                                                    92/294 
  Upgrading        : sssd-nfs-idmap-2.5.2-2.el8_5.4.x86_64                                                                           93/294 
  Upgrading        : python3-sssdconfig-2.5.2-2.el8_5.4.noarch                                                                       94/294 
  Upgrading        : libsss_sudo-2.5.2-2.el8_5.4.x86_64                                                                              95/294 
  Running scriptlet: libsss_sudo-2.5.2-2.el8_5.4.x86_64                                                                              95/294 
  Running scriptlet: sssd-common-2.5.2-2.el8_5.4.x86_64                                                                              96/294 
  Upgrading        : sssd-common-2.5.2-2.el8_5.4.x86_64                                                                              96/294 
  Running scriptlet: sssd-common-2.5.2-2.el8_5.4.x86_64                                                                              96/294 
  Running scriptlet: sssd-krb5-common-2.5.2-2.el8_5.4.x86_64                                                                         97/294 
  Upgrading        : sssd-krb5-common-2.5.2-2.el8_5.4.x86_64                                                                         97/294 
  Upgrading        : sssd-common-pac-2.5.2-2.el8_5.4.x86_64                                                                          98/294 
  Upgrading        : sssd-ad-2.5.2-2.el8_5.4.x86_64                                                                                  99/294 
  Running scriptlet: sssd-ipa-2.5.2-2.el8_5.4.x86_64                                                                                100/294 
  Upgrading        : sssd-ipa-2.5.2-2.el8_5.4.x86_64                                                                                100/294 
  Upgrading        : sssd-krb5-2.5.2-2.el8_5.4.x86_64                                                                               101/294 
  Upgrading        : sssd-ldap-2.5.2-2.el8_5.4.x86_64                                                                               102/294 
  Running scriptlet: sssd-proxy-2.5.2-2.el8_5.4.x86_64                                                                              103/294 
  Upgrading        : sssd-proxy-2.5.2-2.el8_5.4.x86_64                                                                              103/294 
  Upgrading        : sos-4.1-9.el8_5.noarch                                                                                         104/294 
  Upgrading        : cockpit-system-251.3-1.el8_5.noarch                                                                            105/294 
  Upgrading        : kernel-tools-libs-4.18.0-348.12.2.el8_5.x86_64                                                                 106/294 
  Running scriptlet: kernel-tools-libs-4.18.0-348.12.2.el8_5.x86_64                                                                 106/294 
  Upgrading        : libgomp-8.5.0-4.el8_5.x86_64                                                                                   107/294 
  Running scriptlet: libgomp-8.5.0-4.el8_5.x86_64                                                                                   107/294 
  Upgrading        : vim-filesystem-2:8.0.1763-16.el8_5.4.noarch                                                                    108/294 
  Upgrading        : vim-common-2:8.0.1763-16.el8_5.4.x86_64                                                                        109/294 
  Upgrading        : tigervnc-license-1.11.0-10.el8_5.noarch                                                                        110/294 
  Upgrading        : gnome-control-center-filesystem-3.28.2-29.el8_5.noarch                                                         111/294 
  Upgrading        : cpp-8.5.0-4.el8_5.x86_64                                                                                       112/294 
  Running scriptlet: cpp-8.5.0-4.el8_5.x86_64                                                                                       112/294 
  Upgrading        : tzdata-java-2021e-1.el8.noarch                                                                                 113/294 
  Upgrading        : java-1.8.0-openjdk-headless-1:1.8.0.322.b06-2.el8_5.x86_64                                                     114/294 
  Running scriptlet: java-1.8.0-openjdk-headless-1:1.8.0.322.b06-2.el8_5.x86_64                                                     114/294 
  Upgrading        : java-1.8.0-openjdk-1:1.8.0.322.b06-2.el8_5.x86_64                                                              115/294 
  Running scriptlet: java-1.8.0-openjdk-1:1.8.0.322.b06-2.el8_5.x86_64                                                              115/294 
  Upgrading        : gcc-8.5.0-4.el8_5.x86_64                                                                                       116/294 
  Running scriptlet: gcc-8.5.0-4.el8_5.x86_64                                                                                       116/294 
  Upgrading        : gnome-control-center-3.28.2-29.el8_5.x86_64                                                                    117/294 
  Upgrading        : tigervnc-server-minimal-1.11.0-10.el8_5.x86_64                                                                 118/294 
  Upgrading        : vim-enhanced-2:8.0.1763-16.el8_5.4.x86_64                                                                      119/294 
  Upgrading        : kernel-tools-4.18.0-348.12.2.el8_5.x86_64                                                                      120/294 
  Upgrading        : cockpit-251.3-1.el8_5.x86_64                                                                                   121/294 
  Upgrading        : sssd-2.5.2-2.el8_5.4.x86_64                                                                                    122/294 
  Upgrading        : sssd-kcm-2.5.2-2.el8_5.4.x86_64                                                                                123/294 
  Running scriptlet: sssd-kcm-2.5.2-2.el8_5.4.x86_64                                                                                123/294 
  Upgrading        : dnf-plugins-core-4.0.21-4.el8_5.noarch                                                                         124/294 
  Upgrading        : clevis-luks-15-1.el8_5.1.x86_64                                                                                125/294 
  Upgrading        : freerdp-libs-2:2.2.0-7.el8_5.x86_64                                                                            126/294 
  Upgrading        : gnome-classic-session-3.32.1-22.el8_5.noarch                                                                   127/294 
  Upgrading        : poppler-utils-20.11.0-3.el8_5.1.x86_64                                                                         128/294 
  Upgrading        : poppler-glib-20.11.0-3.el8_5.1.x86_64                                                                          129/294 
  Running scriptlet: poppler-glib-20.11.0-3.el8_5.1.x86_64                                                                          129/294 
  Upgrading        : firefox-91.5.0-1.el8_5.x86_64                                                                                  130/294 
  Running scriptlet: firefox-91.5.0-1.el8_5.x86_64                                                                                  130/294 
  Running scriptlet: flatpak-1.8.5-5.el8_5.x86_64                                                                                   131/294 
  Upgrading        : flatpak-1.8.5-5.el8_5.x86_64                                                                                   131/294 
  Running scriptlet: flatpak-1.8.5-5.el8_5.x86_64                                                                                   131/294 
  Upgrading        : python3-rpm-4.14.3-19.el8_5.2.x86_64                                                                           132/294 
  Installing       : kernel-4.18.0-348.12.2.el8_5.x86_64                                                                            133/294 
  Upgrading        : libvirt-daemon-kvm-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                             134/294 
  Upgrading        : flatpak-libs-1.8.5-5.el8_5.x86_64                                                                              135/294 
  Upgrading        : libvirt-daemon-config-network-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                  136/294 
  Running scriptlet: libvirt-daemon-config-network-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                  136/294 
  Upgrading        : samba-client-4.14.5-9.el8_5.x86_64                                                                             137/294 
  Running scriptlet: samba-client-4.14.5-9.el8_5.x86_64                                                                             137/294 
  Upgrading        : insights-client-3.1.7-1.el8_5.noarch                                                                           138/294 
  Running scriptlet: insights-client-3.1.7-1.el8_5.noarch                                                                           138/294 
  Upgrading        : qemu-guest-agent-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                                            139/294 
  Running scriptlet: qemu-guest-agent-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                                            139/294 
  Running scriptlet: ppp-2.4.7-26.el8_1.x86_64                                                                                      140/294 
  Upgrading        : ppp-2.4.7-26.el8_1.x86_64                                                                                      140/294 
  Running scriptlet: ppp-2.4.7-26.el8_1.x86_64                                                                                      140/294 
  Upgrading        : fwupd-1.5.9-1.el8_4.x86_64                                                                                     141/294 
  Running scriptlet: fwupd-1.5.9-1.el8_4.x86_64                                                                                     141/294 
  Upgrading        : rpm-plugin-systemd-inhibit-4.14.3-19.el8_5.2.x86_64                                                            142/294 
  Running scriptlet: tcpdump-14:4.9.3-2.el8_5.1.x86_64                                                                              143/294 
  Upgrading        : tcpdump-14:4.9.3-2.el8_5.1.x86_64                                                                              143/294 
  Upgrading        : python3-perf-4.18.0-348.12.2.el8_5.x86_64                                                                      144/294 
  Upgrading        : kernel-headers-4.18.0-348.12.2.el8_5.x86_64                                                                    145/294 
  Upgrading        : bpftool-4.18.0-348.12.2.el8_5.x86_64                                                                           146/294 
  Upgrading        : tzdata-2021e-1.el8.noarch                                                                                      147/294 
  Upgrading        : unzip-6.0-45.el8_4.x86_64                                                                                      148/294 
  Installing       : kernel-devel-4.18.0-348.12.2.el8_5.x86_64                                                                      149/294 
  Running scriptlet: kernel-devel-4.18.0-348.12.2.el8_5.x86_64                                                                      149/294 
  Running scriptlet: firefox-91.2.0-4.el8_4.x86_64                                                                                  150/294 
  Cleanup          : firefox-91.2.0-4.el8_4.x86_64                                                                                  150/294 
  Running scriptlet: firefox-91.2.0-4.el8_4.x86_64                                                                                  150/294 
  Running scriptlet: samba-client-4.14.5-2.el8.x86_64                                                                               151/294 
  Cleanup          : samba-client-4.14.5-2.el8.x86_64                                                                               151/294 
  Running scriptlet: samba-client-4.14.5-2.el8.x86_64                                                                               151/294 
  Cleanup          : flatpak-1.8.5-4.el8.x86_64                                                                                     152/294 
  Cleanup          : libvirt-daemon-kvm-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                                               153/294 
  Cleanup          : libvirt-daemon-driver-storage-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                                    154/294 
  Cleanup          : sssd-2.5.2-2.el8.x86_64                                                                                        155/294 
  Cleanup          : qemu-kvm-15:4.2.0-59.module+el8.5.0+12817+cb650d43.x86_64                                                      156/294 
  Cleanup          : gnome-classic-session-3.32.1-20.el8.noarch                                                                     157/294 
  Cleanup          : cockpit-251.1-1.el8.x86_64                                                                                     158/294 
  Cleanup          : cockpit-system-251.1-1.el8.noarch                                                                              159/294 
  Running scriptlet: insights-client-3.1.5-1.el8.noarch                                                                             160/294 
  Cleanup          : insights-client-3.1.5-1.el8.noarch                                                                             160/294 
  Running scriptlet: insights-client-3.1.5-1.el8.noarch                                                                             160/294 
  Cleanup          : flatpak-selinux-1.8.5-4.el8.noarch                                                                             161/294 
  Running scriptlet: flatpak-selinux-1.8.5-4.el8.noarch                                                                             161/294 
  Cleanup          : clevis-luks-15-1.el8.x86_64                                                                                    162/294 
  Cleanup          : libvirt-daemon-config-network-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                                    163/294 
  Cleanup          : rpm-plugin-selinux-4.14.3-19.el8.x86_64                                                                        164/294 
  Cleanup          : selinux-policy-targeted-3.14.3-80.el8.noarch                                                                   165/294 
  Running scriptlet: selinux-policy-targeted-3.14.3-80.el8.noarch                                                                   165/294 
  Cleanup          : selinux-policy-3.14.3-80.el8.noarch                                                                            166/294 
  Running scriptlet: selinux-policy-3.14.3-80.el8.noarch                                                                            166/294 
  Cleanup          : gnome-shell-extension-apps-menu-3.32.1-20.el8.noarch                                                           167/294 
  Cleanup          : gnome-shell-extension-desktop-icons-3.32.1-20.el8.noarch                                                       168/294 
  Cleanup          : gnome-shell-extension-horizontal-workspaces-3.32.1-20.el8.noarch                                               169/294 
  Cleanup          : gnome-shell-extension-launch-new-instance-3.32.1-20.el8.noarch                                                 170/294 
  Cleanup          : gnome-shell-extension-places-menu-3.32.1-20.el8.noarch                                                         171/294 
  Cleanup          : gnome-shell-extension-window-list-3.32.1-20.el8.noarch                                                         172/294 
  Cleanup          : dnf-plugins-core-4.0.21-3.el8.noarch                                                                           173/294 
  Cleanup          : ibus-setup-1.5.19-13.el8.noarch                                                                                174/294 
  Cleanup          : libvirt-daemon-driver-nodedev-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                                    175/294 
  Cleanup          : libvirt-daemon-driver-qemu-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                                       176/294 
  Cleanup          : libvirt-daemon-driver-interface-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                                  177/294 
  Cleanup          : libvirt-daemon-driver-network-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                                    178/294 
  Running scriptlet: libvirt-daemon-driver-network-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                                    178/294 
  Cleanup          : libvirt-daemon-driver-nwfilter-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                                   179/294 
  Cleanup          : libvirt-daemon-driver-secret-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                                     180/294 
  Cleanup          : sssd-ipa-2.5.2-2.el8.x86_64                                                                                    181/294 
  Cleanup          : sssd-ad-2.5.2-2.el8.x86_64                                                                                     182/294 
  Cleanup          : flatpak-libs-1.8.5-4.el8.x86_64                                                                                183/294 
  Running scriptlet: ostree-2021.3-1.el8.x86_64                                                                                     184/294 
  Cleanup          : ostree-2021.3-1.el8.x86_64                                                                                     184/294 
  Cleanup          : sssd-common-pac-2.5.2-2.el8.x86_64                                                                             185/294 
  Cleanup          : sssd-ldap-2.5.2-2.el8.x86_64                                                                                   186/294 
  Running scriptlet: sssd-kcm-2.5.2-2.el8.x86_64                                                                                    187/294 
  Cleanup          : sssd-kcm-2.5.2-2.el8.x86_64                                                                                    187/294 
  Running scriptlet: sssd-kcm-2.5.2-2.el8.x86_64                                                                                    187/294 
  Cleanup          : poppler-utils-20.11.0-3.el8.x86_64                                                                             188/294 
  Cleanup          : poppler-glib-20.11.0-3.el8.x86_64                                                                              189/294 
  Running scriptlet: poppler-glib-20.11.0-3.el8.x86_64                                                                              189/294 
  Cleanup          : poppler-20.11.0-3.el8.x86_64                                                                                   190/294 
  Running scriptlet: poppler-20.11.0-3.el8.x86_64                                                                                   190/294 
  Cleanup          : systemd-udev-239-51.el8.x86_64                                                                                 191/294 
  Running scriptlet: systemd-udev-239-51.el8.x86_64                                                                                 191/294 
  Cleanup          : sssd-krb5-2.5.2-2.el8.x86_64                                                                                   192/294 
  Cleanup          : sssd-proxy-2.5.2-2.el8.x86_64                                                                                  193/294 
  Cleanup          : gnome-control-center-3.28.2-28.el8.x86_64                                                                      194/294 
  Cleanup          : libsmbclient-4.14.5-2.el8.x86_64                                                                               195/294 
  Running scriptlet: qemu-guest-agent-15:4.2.0-59.module+el8.5.0+12817+cb650d43.x86_64                                              196/294 
  Cleanup          : qemu-guest-agent-15:4.2.0-59.module+el8.5.0+12817+cb650d43.x86_64                                              196/294 
  Running scriptlet: qemu-guest-agent-15:4.2.0-59.module+el8.5.0+12817+cb650d43.x86_64                                              196/294 
  Cleanup          : systemd-container-239-51.el8.x86_64                                                                            197/294 
  Running scriptlet: cockpit-ws-251.1-1.el8.x86_64                                                                                  198/294 
  Cleanup          : cockpit-ws-251.1-1.el8.x86_64                                                                                  198/294 
  Running scriptlet: cockpit-ws-251.1-1.el8.x86_64                                                                                  198/294 
  Running scriptlet: fwupd-1.5.9-1.el8.x86_64                                                                                       199/294 
  Cleanup          : fwupd-1.5.9-1.el8.x86_64                                                                                       199/294 
  Running scriptlet: fwupd-1.5.9-1.el8.x86_64                                                                                       199/294 
  Cleanup          : tigervnc-server-minimal-1.11.0-9.el8.x86_64                                                                    200/294 
  Cleanup          : qemu-kvm-core-15:4.2.0-59.module+el8.5.0+12817+cb650d43.x86_64                                                 201/294 
  Cleanup          : libvirt-daemon-driver-storage-scsi-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                               202/294 
  Cleanup          : openssl-1:1.1.1k-4.el8.x86_64                                                                                  203/294 
  Cleanup          : ostree-libs-2021.3-1.el8.x86_64                                                                                204/294 
  Cleanup          : cockpit-bridge-251.1-1.el8.x86_64                                                                              205/294 
  Cleanup          : libvirt-daemon-driver-storage-disk-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                               206/294 
  Cleanup          : libvirt-daemon-driver-storage-gluster-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                            207/294 
  Cleanup          : libvirt-daemon-driver-storage-iscsi-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                              208/294 
  Cleanup          : libvirt-daemon-driver-storage-iscsi-direct-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                       209/294 
  Cleanup          : libvirt-daemon-driver-storage-logical-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                            210/294 
  Cleanup          : libvirt-daemon-driver-storage-mpath-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                              211/294 
  Cleanup          : libvirt-daemon-driver-storage-rbd-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                                212/294 
  Cleanup          : libvirt-daemon-driver-storage-core-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                               213/294 
  Running scriptlet: libvirt-daemon-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                                                   214/294 
  Cleanup          : libvirt-daemon-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                                                   214/294 
  Running scriptlet: libvirt-daemon-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                                                   214/294 
  Cleanup          : ppp-2.4.7-26.el8.x86_64                                                                                        215/294 
  Cleanup          : accountsservice-libs-0.6.55-2.el8.x86_64                                                                       216/294 
  Running scriptlet: accountsservice-0.6.55-2.el8.x86_64                                                                            217/294 
  Cleanup          : accountsservice-0.6.55-2.el8.x86_64                                                                            217/294 
  Running scriptlet: accountsservice-0.6.55-2.el8.x86_64                                                                            217/294 
  Running scriptlet: polkit-0.115-12.el8.x86_64                                                                                     218/294 
  Cleanup          : polkit-0.115-12.el8.x86_64                                                                                     218/294 
  Running scriptlet: polkit-0.115-12.el8.x86_64                                                                                     218/294 
  Running scriptlet: kexec-tools-2.0.20-57.el8.x86_64                                                                               219/294 
  Cleanup          : kexec-tools-2.0.20-57.el8.x86_64                                                                               219/294 
  Running scriptlet: kexec-tools-2.0.20-57.el8.x86_64                                                                               219/294 
  Cleanup          : python3-rpm-4.14.3-19.el8.x86_64                                                                               220/294 
  Cleanup          : java-1.8.0-openjdk-1:1.8.0.302.b08-3.el8.x86_64                                                                221/294 
  Running scriptlet: java-1.8.0-openjdk-1:1.8.0.302.b08-3.el8.x86_64                                                                221/294 
  Cleanup          : java-1.8.0-openjdk-headless-1:1.8.0.302.b08-3.el8.x86_64                                                       222/294 
  Running scriptlet: java-1.8.0-openjdk-headless-1:1.8.0.302.b08-3.el8.x86_64                                                       222/294 
  Cleanup          : freerdp-libs-2:2.2.0-2.el8.x86_64                                                                              223/294 
  Cleanup          : libwinpr-2:2.2.0-2.el8.x86_64                                                                                  224/294 
  Cleanup          : qemu-img-15:4.2.0-59.module+el8.5.0+12817+cb650d43.x86_64                                                      225/294 
  Running scriptlet: libwbclient-4.14.5-2.el8.x86_64                                                                                226/294 
  Cleanup          : libwbclient-4.14.5-2.el8.x86_64                                                                                226/294 
  Cleanup          : samba-client-libs-4.14.5-2.el8.x86_64                                                                          227/294 
  Cleanup          : samba-common-libs-4.14.5-2.el8.x86_64                                                                          228/294 
  Cleanup          : clevis-15-1.el8.x86_64                                                                                         229/294 
  Running scriptlet: gcc-8.5.0-3.el8.x86_64                                                                                         230/294 
  Cleanup          : gcc-8.5.0-3.el8.x86_64                                                                                         230/294 
  Running scriptlet: binutils-2.30-108.el8.x86_64                                                                                   231/294 
  Cleanup          : binutils-2.30-108.el8.x86_64                                                                                   231/294 
  Running scriptlet: binutils-2.30-108.el8.x86_64                                                                                   231/294 
  Cleanup          : rpm-build-libs-4.14.3-19.el8.x86_64                                                                            232/294 
  Running scriptlet: rpm-build-libs-4.14.3-19.el8.x86_64                                                                            232/294 
  Cleanup          : sssd-krb5-common-2.5.2-2.el8.x86_64                                                                            233/294 
  Running scriptlet: sssd-common-2.5.2-2.el8.x86_64                                                                                 234/294 
  Cleanup          : sssd-common-2.5.2-2.el8.x86_64                                                                                 234/294 
  Running scriptlet: sssd-common-2.5.2-2.el8.x86_64                                                                                 234/294 
  Running scriptlet: sssd-client-2.5.2-2.el8.x86_64                                                                                 235/294 
  Cleanup          : sssd-client-2.5.2-2.el8.x86_64                                                                                 235/294 
  Running scriptlet: sssd-client-2.5.2-2.el8.x86_64                                                                                 235/294 
  Cleanup          : ibus-1.5.19-13.el8.x86_64                                                                                      236/294 
  Running scriptlet: ibus-1.5.19-13.el8.x86_64                                                                                      236/294 
  Cleanup          : flatpak-session-helper-1.8.5-4.el8.x86_64                                                                      237/294 
  Cleanup          : rpm-plugin-systemd-inhibit-4.14.3-19.el8.x86_64                                                                238/294 
  Cleanup          : rpm-libs-4.14.3-19.el8.x86_64                                                                                  239/294 
  Running scriptlet: rpm-libs-4.14.3-19.el8.x86_64                                                                                  239/294 
  Cleanup          : nss-sysinit-3.67.0-6.el8_4.x86_64                                                                              240/294 
  Cleanup          : nss-3.67.0-6.el8_4.x86_64                                                                                      241/294 
  Cleanup          : nss-softokn-3.67.0-6.el8_4.x86_64                                                                              242/294 
  Cleanup          : polkit-libs-0.115-12.el8.x86_64                                                                                243/294 
  Running scriptlet: polkit-libs-0.115-12.el8.x86_64                                                                                243/294 
  Cleanup          : libvirt-libs-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                                                     244/294 
  Running scriptlet: libvirt-libs-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                                                     244/294 
  Cleanup          : cryptsetup-2.3.3-4.el8.x86_64                                                                                  245/294 
  Cleanup          : qemu-kvm-block-curl-15:4.2.0-59.module+el8.5.0+12817+cb650d43.x86_64                                           246/294 
  Cleanup          : qemu-kvm-block-gluster-15:4.2.0-59.module+el8.5.0+12817+cb650d43.x86_64                                        247/294 
  Cleanup          : qemu-kvm-block-iscsi-15:4.2.0-59.module+el8.5.0+12817+cb650d43.x86_64                                          248/294 
  Cleanup          : qemu-kvm-block-rbd-15:4.2.0-59.module+el8.5.0+12817+cb650d43.x86_64                                            249/294 
  Cleanup          : qemu-kvm-block-ssh-15:4.2.0-59.module+el8.5.0+12817+cb650d43.x86_64                                            250/294 
  Running scriptlet: qemu-kvm-common-15:4.2.0-59.module+el8.5.0+12817+cb650d43.x86_64                                               251/294 
  Cleanup          : qemu-kvm-common-15:4.2.0-59.module+el8.5.0+12817+cb650d43.x86_64                                               251/294 
  Running scriptlet: qemu-kvm-common-15:4.2.0-59.module+el8.5.0+12817+cb650d43.x86_64                                               251/294 
  Cleanup          : libstdc++-8.5.0-3.el8.x86_64                                                                                   252/294 
  Running scriptlet: libstdc++-8.5.0-3.el8.x86_64                                                                                   252/294 
  Cleanup          : ibus-gtk2-1.5.19-13.el8.x86_64                                                                                 253/294 
  Cleanup          : ibus-gtk3-1.5.19-13.el8.x86_64                                                                                 254/294 
  Cleanup          : libsss_certmap-2.5.2-2.el8.x86_64                                                                              255/294 
  Running scriptlet: libsss_certmap-2.5.2-2.el8.x86_64                                                                              255/294 
  Cleanup          : kernel-tools-4.18.0-348.el8.x86_64                                                                             256/294 
  Cleanup          : tcpdump-14:4.9.3-2.el8.x86_64                                                                                  257/294 
  Cleanup          : samba-common-4.14.5-2.el8.noarch                                                                               258/294 
  Running scriptlet: systemd-239-51.el8.x86_64                                                                                      259/294 
  Cleanup          : systemd-239-51.el8.x86_64                                                                                      259/294 
  Cleanup          : cryptsetup-libs-2.3.3-4.el8.x86_64                                                                             260/294 
  Running scriptlet: cryptsetup-libs-2.3.3-4.el8.x86_64                                                                             260/294 
  Cleanup          : systemd-libs-239-51.el8.x86_64                                                                                 261/294 
  Cleanup          : systemd-pam-239-51.el8.x86_64                                                                                  262/294 
  Cleanup          : nss-softokn-freebl-3.67.0-6.el8_4.x86_64                                                                       263/294 
  Cleanup          : rpm-4.14.3-19.el8.x86_64                                                                                       264/294 
  Cleanup          : sudo-1.8.29-7.el8.x86_64                                                                                       265/294 
  Cleanup          : vim-enhanced-2:8.0.1763-16.el8.x86_64                                                                          266/294 
  Cleanup          : tzdata-java-2021c-1.el8.noarch                                                                                 267/294 
  Cleanup          : tigervnc-license-1.11.0-9.el8.noarch                                                                           268/294 
  Cleanup          : gnome-control-center-filesystem-3.28.2-28.el8.noarch                                                           269/294 
  Cleanup          : python3-dnf-plugins-core-4.0.21-3.el8.noarch                                                                   270/294 
  Cleanup          : gnome-shell-extension-common-3.32.1-20.el8.noarch                                                              271/294 
  Cleanup          : sos-4.1-5.el8.noarch                                                                                           272/294 
  Cleanup          : python3-sssdconfig-2.5.2-2.el8.noarch                                                                          273/294 
  Cleanup          : kernel-headers-4.18.0-348.el8.x86_64                                                                           274/294 
  Cleanup          : tzdata-2021c-1.el8.noarch                                                                                      275/294 
  Cleanup          : vim-common-2:8.0.1763-16.el8.x86_64                                                                            276/294 
  Cleanup          : vim-filesystem-2:8.0.1763-16.el8.noarch                                                                        277/294 
  Cleanup          : vim-minimal-2:8.0.1763-16.el8.x86_64                                                                           278/294 
  Cleanup          : openssl-libs-1:1.1.1k-4.el8.x86_64                                                                             279/294 
  Running scriptlet: openssl-libs-1:1.1.1k-4.el8.x86_64                                                                             279/294 
  Cleanup          : nss-util-3.67.0-6.el8_4.x86_64                                                                                 280/294 
  Cleanup          : libgcc-8.5.0-3.el8.x86_64                                                                                      281/294 
  Running scriptlet: libgcc-8.5.0-3.el8.x86_64                                                                                      281/294 
  Cleanup          : kernel-tools-libs-4.18.0-348.el8.x86_64                                                                        282/294 
  Running scriptlet: kernel-tools-libs-4.18.0-348.el8.x86_64                                                                        282/294 
  Cleanup          : ibus-libs-1.5.19-13.el8.x86_64                                                                                 283/294 
  Cleanup          : libsss_idmap-2.5.2-2.el8.x86_64                                                                                284/294 
  Running scriptlet: libsss_idmap-2.5.2-2.el8.x86_64                                                                                284/294 
  Cleanup          : libsss_nss_idmap-2.5.2-2.el8.x86_64                                                                            285/294 
  Running scriptlet: libsss_nss_idmap-2.5.2-2.el8.x86_64                                                                            285/294 
  Cleanup          : libsss_autofs-2.5.2-2.el8.x86_64                                                                               286/294 
  Cleanup          : libsss_sudo-2.5.2-2.el8.x86_64                                                                                 287/294 
  Running scriptlet: libsss_sudo-2.5.2-2.el8.x86_64                                                                                 287/294 
  Cleanup          : sssd-nfs-idmap-2.5.2-2.el8.x86_64                                                                              288/294 
  Running scriptlet: cpp-8.5.0-3.el8.x86_64                                                                                         289/294 
  Cleanup          : cpp-8.5.0-3.el8.x86_64                                                                                         289/294 
  Running scriptlet: libgomp-8.5.0-3.el8.x86_64                                                                                     290/294 
  Cleanup          : libgomp-8.5.0-3.el8.x86_64                                                                                     290/294 
  Running scriptlet: libgomp-8.5.0-3.el8.x86_64                                                                                     290/294 
  Cleanup          : libipa_hbac-2.5.2-2.el8.x86_64                                                                                 291/294 
  Running scriptlet: libipa_hbac-2.5.2-2.el8.x86_64                                                                                 291/294 
  Cleanup          : python3-perf-4.18.0-348.el8.x86_64                                                                             292/294 
  Cleanup          : bpftool-4.18.0-348.el8.x86_64                                                                                  293/294 
  Cleanup          : unzip-6.0-45.el8.x86_64                                                                                        294/294 
  Running scriptlet: libwbclient-4.14.5-9.el8_5.x86_64                                                                              294/294 
  Running scriptlet: libvirt-daemon-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                                 294/294 
  Running scriptlet: kernel-core-4.18.0-348.12.2.el8_5.x86_64                                                                       294/294 
  Running scriptlet: ibus-1.5.19-14.el8_5.x86_64                                                                                    294/294 
  Running scriptlet: nss-3.67.0-7.el8_5.x86_64                                                                                      294/294 
  Running scriptlet: clevis-15-1.el8_5.1.x86_64                                                                                     294/294 
  Running scriptlet: sssd-common-2.5.2-2.el8_5.4.x86_64                                                                             294/294 
  Running scriptlet: java-1.8.0-openjdk-1:1.8.0.322.b06-2.el8_5.x86_64                                                              294/294 
  Running scriptlet: firefox-91.5.0-1.el8_5.x86_64                                                                                  294/294 
  Running scriptlet: libvirt-daemon-config-network-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                  294/294 
  Running scriptlet: unzip-6.0-45.el8.x86_64                                                                                        294/294 
  Running scriptlet: systemd-239-51.el8_5.3.x86_64                                                                                  294/294 
  Running scriptlet: systemd-udev-239-51.el8_5.3.x86_64                                                                             294/294 
  Running scriptlet: vim-common-2:8.0.1763-16.el8_5.4.x86_64                                                                        294/294 
  Verifying        : kernel-4.18.0-348.12.2.el8_5.x86_64                                                                              1/294 
  Verifying        : kernel-devel-4.18.0-348.12.2.el8_5.x86_64                                                                        2/294 
  Verifying        : kernel-modules-4.18.0-348.12.2.el8_5.x86_64                                                                      3/294 
  Verifying        : kernel-core-4.18.0-348.12.2.el8_5.x86_64                                                                         4/294 
  Verifying        : tzdata-java-2021e-1.el8.noarch                                                                                   5/294 
  Verifying        : tzdata-java-2021c-1.el8.noarch                                                                                   6/294 
  Verifying        : flatpak-1.8.5-5.el8_5.x86_64                                                                                     7/294 
  Verifying        : flatpak-1.8.5-4.el8.x86_64                                                                                       8/294 
  Verifying        : flatpak-session-helper-1.8.5-5.el8_5.x86_64                                                                      9/294 
  Verifying        : flatpak-session-helper-1.8.5-4.el8.x86_64                                                                       10/294 
  Verifying        : ibus-gtk2-1.5.19-14.el8_5.x86_64                                                                                11/294 
  Verifying        : ibus-gtk2-1.5.19-13.el8.x86_64                                                                                  12/294 
  Verifying        : ibus-setup-1.5.19-14.el8_5.noarch                                                                               13/294 
  Verifying        : ibus-setup-1.5.19-13.el8.noarch                                                                                 14/294 
  Verifying        : flatpak-libs-1.8.5-5.el8_5.x86_64                                                                               15/294 
  Verifying        : flatpak-libs-1.8.5-4.el8.x86_64                                                                                 16/294 
  Verifying        : ibus-gtk3-1.5.19-14.el8_5.x86_64                                                                                17/294 
  Verifying        : ibus-gtk3-1.5.19-13.el8.x86_64                                                                                  18/294 
  Verifying        : ibus-libs-1.5.19-14.el8_5.x86_64                                                                                19/294 
  Verifying        : ibus-libs-1.5.19-13.el8.x86_64                                                                                  20/294 
  Verifying        : insights-client-3.1.7-1.el8_5.noarch                                                                            21/294 
  Verifying        : insights-client-3.1.5-1.el8.noarch                                                                              22/294 
  Verifying        : ibus-1.5.19-14.el8_5.x86_64                                                                                     23/294 
  Verifying        : ibus-1.5.19-13.el8.x86_64                                                                                       24/294 
  Verifying        : flatpak-selinux-1.8.5-5.el8_5.noarch                                                                            25/294 
  Verifying        : flatpak-selinux-1.8.5-4.el8.noarch                                                                              26/294 
  Verifying        : cpp-8.5.0-4.el8_5.x86_64                                                                                        27/294 
  Verifying        : cpp-8.5.0-3.el8.x86_64                                                                                          28/294 
  Verifying        : gcc-8.5.0-4.el8_5.x86_64                                                                                        29/294 
  Verifying        : gcc-8.5.0-3.el8.x86_64                                                                                          30/294 
  Verifying        : libwinpr-2:2.2.0-7.el8_5.x86_64                                                                                 31/294 
  Verifying        : libwinpr-2:2.2.0-2.el8.x86_64                                                                                   32/294 
  Verifying        : freerdp-libs-2:2.2.0-7.el8_5.x86_64                                                                             33/294 
  Verifying        : freerdp-libs-2:2.2.0-2.el8.x86_64                                                                               34/294 
  Verifying        : nss-softokn-3.67.0-7.el8_5.x86_64                                                                               35/294 
  Verifying        : nss-softokn-3.67.0-6.el8_4.x86_64                                                                               36/294 
  Verifying        : nss-sysinit-3.67.0-7.el8_5.x86_64                                                                               37/294 
  Verifying        : nss-sysinit-3.67.0-6.el8_4.x86_64                                                                               38/294 
  Verifying        : nss-softokn-freebl-3.67.0-7.el8_5.x86_64                                                                        39/294 
  Verifying        : nss-softokn-freebl-3.67.0-6.el8_4.x86_64                                                                        40/294 
  Verifying        : nss-util-3.67.0-7.el8_5.x86_64                                                                                  41/294 
  Verifying        : nss-util-3.67.0-6.el8_4.x86_64                                                                                  42/294 
  Verifying        : nss-3.67.0-7.el8_5.x86_64                                                                                       43/294 
  Verifying        : nss-3.67.0-6.el8_4.x86_64                                                                                       44/294 
  Verifying        : qemu-guest-agent-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                                             45/294 
  Verifying        : qemu-guest-agent-15:4.2.0-59.module+el8.5.0+12817+cb650d43.x86_64                                               46/294 
  Verifying        : qemu-kvm-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                                                     47/294 
  Verifying        : qemu-kvm-15:4.2.0-59.module+el8.5.0+12817+cb650d43.x86_64                                                       48/294 
  Verifying        : accountsservice-libs-0.6.55-2.el8_5.2.x86_64                                                                    49/294 
  Verifying        : accountsservice-libs-0.6.55-2.el8.x86_64                                                                        50/294 
  Verifying        : gnome-classic-session-3.32.1-22.el8_5.noarch                                                                    51/294 
  Verifying        : gnome-classic-session-3.32.1-20.el8.noarch                                                                      52/294 
  Verifying        : poppler-utils-20.11.0-3.el8_5.1.x86_64                                                                          53/294 
  Verifying        : poppler-utils-20.11.0-3.el8.x86_64                                                                              54/294 
  Verifying        : qemu-kvm-block-gluster-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                                       55/294 
  Verifying        : qemu-kvm-block-gluster-15:4.2.0-59.module+el8.5.0+12817+cb650d43.x86_64                                         56/294 
  Verifying        : qemu-kvm-block-ssh-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                                           57/294 
  Verifying        : qemu-kvm-block-ssh-15:4.2.0-59.module+el8.5.0+12817+cb650d43.x86_64                                             58/294 
  Verifying        : gnome-shell-extension-desktop-icons-3.32.1-22.el8_5.noarch                                                      59/294 
  Verifying        : gnome-shell-extension-desktop-icons-3.32.1-20.el8.noarch                                                        60/294 
  Verifying        : gnome-shell-extension-horizontal-workspaces-3.32.1-22.el8_5.noarch                                              61/294 
  Verifying        : gnome-shell-extension-horizontal-workspaces-3.32.1-20.el8.noarch                                                62/294 
  Verifying        : poppler-glib-20.11.0-3.el8_5.1.x86_64                                                                           63/294 
  Verifying        : poppler-glib-20.11.0-3.el8.x86_64                                                                               64/294 
  Verifying        : gnome-shell-extension-apps-menu-3.32.1-22.el8_5.noarch                                                          65/294 
  Verifying        : gnome-shell-extension-apps-menu-3.32.1-20.el8.noarch                                                            66/294 
  Verifying        : qemu-kvm-block-curl-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                                          67/294 
  Verifying        : qemu-kvm-block-curl-15:4.2.0-59.module+el8.5.0+12817+cb650d43.x86_64                                            68/294 
  Verifying        : poppler-20.11.0-3.el8_5.1.x86_64                                                                                69/294 
  Verifying        : poppler-20.11.0-3.el8.x86_64                                                                                    70/294 
  Verifying        : gnome-shell-extension-window-list-3.32.1-22.el8_5.noarch                                                        71/294 
  Verifying        : gnome-shell-extension-window-list-3.32.1-20.el8.noarch                                                          72/294 
  Verifying        : gnome-shell-extension-launch-new-instance-3.32.1-22.el8_5.noarch                                                73/294 
  Verifying        : gnome-shell-extension-launch-new-instance-3.32.1-20.el8.noarch                                                  74/294 
  Verifying        : qemu-kvm-core-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                                                75/294 
  Verifying        : qemu-kvm-core-15:4.2.0-59.module+el8.5.0+12817+cb650d43.x86_64                                                  76/294 
  Verifying        : qemu-kvm-block-rbd-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                                           77/294 
  Verifying        : qemu-kvm-block-rbd-15:4.2.0-59.module+el8.5.0+12817+cb650d43.x86_64                                             78/294 
  Verifying        : gnome-shell-extension-places-menu-3.32.1-22.el8_5.noarch                                                        79/294 
  Verifying        : gnome-shell-extension-places-menu-3.32.1-20.el8.noarch                                                          80/294 
  Verifying        : accountsservice-0.6.55-2.el8_5.2.x86_64                                                                         81/294 
  Verifying        : accountsservice-0.6.55-2.el8.x86_64                                                                             82/294 
  Verifying        : qemu-kvm-common-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                                              83/294 
  Verifying        : qemu-kvm-common-15:4.2.0-59.module+el8.5.0+12817+cb650d43.x86_64                                                84/294 
  Verifying        : qemu-kvm-block-iscsi-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                                         85/294 
  Verifying        : qemu-kvm-block-iscsi-15:4.2.0-59.module+el8.5.0+12817+cb650d43.x86_64                                           86/294 
  Verifying        : qemu-img-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                                                     87/294 
  Verifying        : qemu-img-15:4.2.0-59.module+el8.5.0+12817+cb650d43.x86_64                                                       88/294 
  Verifying        : gnome-shell-extension-common-3.32.1-22.el8_5.noarch                                                             89/294 
  Verifying        : gnome-shell-extension-common-3.32.1-20.el8.noarch                                                               90/294 
  Verifying        : firefox-91.5.0-1.el8_5.x86_64                                                                                   91/294 
  Verifying        : firefox-91.2.0-4.el8_4.x86_64                                                                                   92/294 
  Verifying        : java-1.8.0-openjdk-1:1.8.0.322.b06-2.el8_5.x86_64                                                               93/294 
  Verifying        : java-1.8.0-openjdk-1:1.8.0.302.b08-3.el8.x86_64                                                                 94/294 
  Verifying        : java-1.8.0-openjdk-headless-1:1.8.0.322.b06-2.el8_5.x86_64                                                      95/294 
  Verifying        : java-1.8.0-openjdk-headless-1:1.8.0.302.b08-3.el8.x86_64                                                        96/294 
  Verifying        : clevis-15-1.el8_5.1.x86_64                                                                                      97/294 
  Verifying        : clevis-15-1.el8.x86_64                                                                                          98/294 
  Verifying        : libvirt-daemon-driver-nwfilter-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                  99/294 
  Verifying        : libvirt-daemon-driver-nwfilter-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                                   100/294 
  Verifying        : tigervnc-server-minimal-1.11.0-10.el8_5.x86_64                                                                 101/294 
  Verifying        : tigervnc-server-minimal-1.11.0-9.el8.x86_64                                                                    102/294 
  Verifying        : libvirt-daemon-driver-network-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                  103/294 
  Verifying        : libvirt-daemon-driver-network-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                                    104/294 
  Verifying        : libvirt-daemon-config-network-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                  105/294 
  Verifying        : libvirt-daemon-config-network-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                                    106/294 
  Verifying        : clevis-luks-15-1.el8_5.1.x86_64                                                                                107/294 
  Verifying        : clevis-luks-15-1.el8.x86_64                                                                                    108/294 
  Verifying        : libvirt-daemon-driver-qemu-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                     109/294 
  Verifying        : libvirt-daemon-driver-qemu-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                                       110/294 
  Verifying        : libvirt-daemon-driver-storage-gluster-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                          111/294 
  Verifying        : libvirt-daemon-driver-storage-gluster-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                            112/294 
  Verifying        : libvirt-daemon-driver-nodedev-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                  113/294 
  Verifying        : libvirt-daemon-driver-nodedev-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                                    114/294 
  Verifying        : libvirt-daemon-driver-storage-iscsi-direct-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                     115/294 
  Verifying        : libvirt-daemon-driver-storage-iscsi-direct-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                       116/294 
  Verifying        : ostree-libs-2021.3-2.el8_5.x86_64                                                                              117/294 
  Verifying        : ostree-libs-2021.3-1.el8.x86_64                                                                                118/294 
  Verifying        : libvirt-daemon-driver-storage-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                  119/294 
  Verifying        : libvirt-daemon-driver-storage-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                                    120/294 
  Verifying        : libvirt-daemon-driver-storage-iscsi-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                            121/294 
  Verifying        : libvirt-daemon-driver-storage-iscsi-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                              122/294 
  Verifying        : vim-common-2:8.0.1763-16.el8_5.4.x86_64                                                                        123/294 
  Verifying        : vim-common-2:8.0.1763-16.el8.x86_64                                                                            124/294 
  Verifying        : libvirt-daemon-driver-interface-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                125/294 
  Verifying        : libvirt-daemon-driver-interface-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                                  126/294 
  Verifying        : libvirt-libs-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                                   127/294 
  Verifying        : libvirt-libs-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                                                     128/294 
  Verifying        : libvirt-daemon-driver-storage-scsi-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                             129/294 
  Verifying        : libvirt-daemon-driver-storage-scsi-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                               130/294 
  Verifying        : libvirt-daemon-driver-storage-disk-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                             131/294 
  Verifying        : libvirt-daemon-driver-storage-disk-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                               132/294 
  Verifying        : libvirt-daemon-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                                 133/294 
  Verifying        : libvirt-daemon-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                                                   134/294 
  Verifying        : libvirt-daemon-driver-storage-logical-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                          135/294 
  Verifying        : libvirt-daemon-driver-storage-logical-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                            136/294 
  Verifying        : libvirt-daemon-driver-storage-mpath-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                            137/294 
  Verifying        : libvirt-daemon-driver-storage-mpath-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                              138/294 
  Verifying        : gnome-control-center-filesystem-3.28.2-29.el8_5.noarch                                                         139/294 
  Verifying        : gnome-control-center-filesystem-3.28.2-28.el8.noarch                                                           140/294 
  Verifying        : gnome-control-center-3.28.2-29.el8_5.x86_64                                                                    141/294 
  Verifying        : gnome-control-center-3.28.2-28.el8.x86_64                                                                      142/294 
  Verifying        : libvirt-daemon-driver-storage-rbd-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                              143/294 
  Verifying        : libvirt-daemon-driver-storage-rbd-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                                144/294 
  Verifying        : libvirt-daemon-driver-storage-core-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                             145/294 
  Verifying        : libvirt-daemon-driver-storage-core-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                               146/294 
  Verifying        : vim-enhanced-2:8.0.1763-16.el8_5.4.x86_64                                                                      147/294 
  Verifying        : vim-enhanced-2:8.0.1763-16.el8.x86_64                                                                          148/294 
  Verifying        : ostree-2021.3-2.el8_5.x86_64                                                                                   149/294 
  Verifying        : ostree-2021.3-1.el8.x86_64                                                                                     150/294 
  Verifying        : tigervnc-license-1.11.0-10.el8_5.noarch                                                                        151/294 
  Verifying        : tigervnc-license-1.11.0-9.el8.noarch                                                                           152/294 
  Verifying        : libvirt-daemon-driver-secret-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                   153/294 
  Verifying        : libvirt-daemon-driver-secret-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                                     154/294 
  Verifying        : tcpdump-14:4.9.3-2.el8_5.1.x86_64                                                                              155/294 
  Verifying        : tcpdump-14:4.9.3-2.el8.x86_64                                                                                  156/294 
  Verifying        : libvirt-daemon-kvm-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                             157/294 
  Verifying        : libvirt-daemon-kvm-6.0.0-37.module+el8.5.0+12162+40884dd2.x86_64                                               158/294 
  Verifying        : vim-filesystem-2:8.0.1763-16.el8_5.4.noarch                                                                    159/294 
  Verifying        : vim-filesystem-2:8.0.1763-16.el8.noarch                                                                        160/294 
  Verifying        : ppp-2.4.7-26.el8_1.x86_64                                                                                      161/294 
  Verifying        : ppp-2.4.7-26.el8.x86_64                                                                                        162/294 
  Verifying        : fwupd-1.5.9-1.el8_4.x86_64                                                                                     163/294 
  Verifying        : fwupd-1.5.9-1.el8.x86_64                                                                                       164/294 
  Verifying        : unzip-6.0-45.el8_4.x86_64                                                                                      165/294 
  Verifying        : unzip-6.0-45.el8.x86_64                                                                                        166/294 
  Verifying        : tzdata-2021e-1.el8.noarch                                                                                      167/294 
  Verifying        : tzdata-2021c-1.el8.noarch                                                                                      168/294 
  Verifying        : sudo-1.8.29-7.el8_4.1.x86_64                                                                                   169/294 
  Verifying        : sudo-1.8.29-7.el8.x86_64                                                                                       170/294 
  Verifying        : libgcc-8.5.0-4.el8_5.x86_64                                                                                    171/294 
  Verifying        : libgcc-8.5.0-3.el8.x86_64                                                                                      172/294 
  Verifying        : binutils-2.30-108.el8_5.1.x86_64                                                                               173/294 
  Verifying        : binutils-2.30-108.el8.x86_64                                                                                   174/294 
  Verifying        : libgomp-8.5.0-4.el8_5.x86_64                                                                                   175/294 
  Verifying        : libgomp-8.5.0-3.el8.x86_64                                                                                     176/294 
  Verifying        : libstdc++-8.5.0-4.el8_5.x86_64                                                                                 177/294 
  Verifying        : libstdc++-8.5.0-3.el8.x86_64                                                                                   178/294 
  Verifying        : selinux-policy-3.14.3-80.el8_5.2.noarch                                                                        179/294 
  Verifying        : selinux-policy-3.14.3-80.el8.noarch                                                                            180/294 
  Verifying        : selinux-policy-targeted-3.14.3-80.el8_5.2.noarch                                                               181/294 
  Verifying        : selinux-policy-targeted-3.14.3-80.el8.noarch                                                                   182/294 
  Verifying        : systemd-239-51.el8_5.3.x86_64                                                                                  183/294 
  Verifying        : systemd-239-51.el8.x86_64                                                                                      184/294 
  Verifying        : systemd-container-239-51.el8_5.3.x86_64                                                                        185/294 
  Verifying        : systemd-container-239-51.el8.x86_64                                                                            186/294 
  Verifying        : systemd-pam-239-51.el8_5.3.x86_64                                                                              187/294 
  Verifying        : systemd-pam-239-51.el8.x86_64                                                                                  188/294 
  Verifying        : systemd-udev-239-51.el8_5.3.x86_64                                                                             189/294 
  Verifying        : systemd-udev-239-51.el8.x86_64                                                                                 190/294 
  Verifying        : systemd-libs-239-51.el8_5.3.x86_64                                                                             191/294 
  Verifying        : systemd-libs-239-51.el8.x86_64                                                                                 192/294 
  Verifying        : openssl-1:1.1.1k-5.el8_5.x86_64                                                                                193/294 
  Verifying        : openssl-1:1.1.1k-4.el8.x86_64                                                                                  194/294 
  Verifying        : kexec-tools-2.0.20-57.el8_5.1.x86_64                                                                           195/294 
  Verifying        : kexec-tools-2.0.20-57.el8.x86_64                                                                               196/294 
  Verifying        : openssl-libs-1:1.1.1k-5.el8_5.x86_64                                                                           197/294 
  Verifying        : openssl-libs-1:1.1.1k-4.el8.x86_64                                                                             198/294 
  Verifying        : kernel-tools-4.18.0-348.12.2.el8_5.x86_64                                                                      199/294 
  Verifying        : kernel-tools-4.18.0-348.el8.x86_64                                                                             200/294 
  Verifying        : kernel-tools-libs-4.18.0-348.12.2.el8_5.x86_64                                                                 201/294 
  Verifying        : kernel-tools-libs-4.18.0-348.el8.x86_64                                                                        202/294 
  Verifying        : bpftool-4.18.0-348.12.2.el8_5.x86_64                                                                           203/294 
  Verifying        : bpftool-4.18.0-348.el8.x86_64                                                                                  204/294 
  Verifying        : kernel-headers-4.18.0-348.12.2.el8_5.x86_64                                                                    205/294 
  Verifying        : kernel-headers-4.18.0-348.el8.x86_64                                                                           206/294 
  Verifying        : python3-perf-4.18.0-348.12.2.el8_5.x86_64                                                                      207/294 
  Verifying        : python3-perf-4.18.0-348.el8.x86_64                                                                             208/294 
  Verifying        : polkit-0.115-13.el8_5.1.x86_64                                                                                 209/294 
  Verifying        : polkit-0.115-12.el8.x86_64                                                                                     210/294 
  Verifying        : polkit-libs-0.115-13.el8_5.1.x86_64                                                                            211/294 
  Verifying        : polkit-libs-0.115-12.el8.x86_64                                                                                212/294 
  Verifying        : samba-client-4.14.5-9.el8_5.x86_64                                                                             213/294 
  Verifying        : samba-client-4.14.5-2.el8.x86_64                                                                               214/294 
  Verifying        : samba-common-4.14.5-9.el8_5.noarch                                                                             215/294 
  Verifying        : samba-common-4.14.5-2.el8.noarch                                                                               216/294 
  Verifying        : libsmbclient-4.14.5-9.el8_5.x86_64                                                                             217/294 
  Verifying        : libsmbclient-4.14.5-2.el8.x86_64                                                                               218/294 
  Verifying        : samba-common-libs-4.14.5-9.el8_5.x86_64                                                                        219/294 
  Verifying        : samba-common-libs-4.14.5-2.el8.x86_64                                                                          220/294 
  Verifying        : libwbclient-4.14.5-9.el8_5.x86_64                                                                              221/294 
  Verifying        : libwbclient-4.14.5-2.el8.x86_64                                                                                222/294 
  Verifying        : samba-client-libs-4.14.5-9.el8_5.x86_64                                                                        223/294 
  Verifying        : samba-client-libs-4.14.5-2.el8.x86_64                                                                          224/294 
  Verifying        : sssd-krb5-2.5.2-2.el8_5.4.x86_64                                                                               225/294 
  Verifying        : sssd-krb5-2.5.2-2.el8.x86_64                                                                                   226/294 
  Verifying        : cryptsetup-2.3.3-4.el8_5.1.x86_64                                                                              227/294 
  Verifying        : cryptsetup-2.3.3-4.el8.x86_64                                                                                  228/294 
  Verifying        : sos-4.1-9.el8_5.noarch                                                                                         229/294 
  Verifying        : sos-4.1-5.el8.noarch                                                                                           230/294 
  Verifying        : sssd-client-2.5.2-2.el8_5.4.x86_64                                                                             231/294 
  Verifying        : sssd-client-2.5.2-2.el8.x86_64                                                                                 232/294 
  Verifying        : libsss_sudo-2.5.2-2.el8_5.4.x86_64                                                                             233/294 
  Verifying        : libsss_sudo-2.5.2-2.el8.x86_64                                                                                 234/294 
  Verifying        : sssd-ad-2.5.2-2.el8_5.4.x86_64                                                                                 235/294 
  Verifying        : sssd-ad-2.5.2-2.el8.x86_64                                                                                     236/294 
  Verifying        : libsss_idmap-2.5.2-2.el8_5.4.x86_64                                                                            237/294 
  Verifying        : libsss_idmap-2.5.2-2.el8.x86_64                                                                                238/294 
  Verifying        : sssd-kcm-2.5.2-2.el8_5.4.x86_64                                                                                239/294 
  Verifying        : sssd-kcm-2.5.2-2.el8.x86_64                                                                                    240/294 
  Verifying        : rpm-plugin-systemd-inhibit-4.14.3-19.el8_5.2.x86_64                                                            241/294 
  Verifying        : rpm-plugin-systemd-inhibit-4.14.3-19.el8.x86_64                                                                242/294 
  Verifying        : cockpit-system-251.3-1.el8_5.noarch                                                                            243/294 
  Verifying        : cockpit-system-251.1-1.el8.noarch                                                                              244/294 
  Verifying        : cockpit-ws-251.3-1.el8_5.x86_64                                                                                245/294 
  Verifying        : cockpit-ws-251.1-1.el8.x86_64                                                                                  246/294 
  Verifying        : sssd-krb5-common-2.5.2-2.el8_5.4.x86_64                                                                        247/294 
  Verifying        : sssd-krb5-common-2.5.2-2.el8.x86_64                                                                            248/294 
  Verifying        : python3-sssdconfig-2.5.2-2.el8_5.4.noarch                                                                      249/294 
  Verifying        : python3-sssdconfig-2.5.2-2.el8.noarch                                                                          250/294 
  Verifying        : sssd-2.5.2-2.el8_5.4.x86_64                                                                                    251/294 
  Verifying        : sssd-2.5.2-2.el8.x86_64                                                                                        252/294 
  Verifying        : sssd-nfs-idmap-2.5.2-2.el8_5.4.x86_64                                                                          253/294 
  Verifying        : sssd-nfs-idmap-2.5.2-2.el8.x86_64                                                                              254/294 
  Verifying        : vim-minimal-2:8.0.1763-16.el8_5.4.x86_64                                                                       255/294 
  Verifying        : vim-minimal-2:8.0.1763-16.el8.x86_64                                                                           256/294 
  Verifying        : rpm-build-libs-4.14.3-19.el8_5.2.x86_64                                                                        257/294 
  Verifying        : rpm-build-libs-4.14.3-19.el8.x86_64                                                                            258/294 
  Verifying        : cockpit-bridge-251.3-1.el8_5.x86_64                                                                            259/294 
  Verifying        : cockpit-bridge-251.1-1.el8.x86_64                                                                              260/294 
  Verifying        : sssd-ldap-2.5.2-2.el8_5.4.x86_64                                                                               261/294 
  Verifying        : sssd-ldap-2.5.2-2.el8.x86_64                                                                                   262/294 
  Verifying        : sssd-common-pac-2.5.2-2.el8_5.4.x86_64                                                                         263/294 
  Verifying        : sssd-common-pac-2.5.2-2.el8.x86_64                                                                             264/294 
  Verifying        : sssd-common-2.5.2-2.el8_5.4.x86_64                                                                             265/294 
  Verifying        : sssd-common-2.5.2-2.el8.x86_64                                                                                 266/294 
  Verifying        : cockpit-251.3-1.el8_5.x86_64                                                                                   267/294 
  Verifying        : cockpit-251.1-1.el8.x86_64                                                                                     268/294 
  Verifying        : sssd-ipa-2.5.2-2.el8_5.4.x86_64                                                                                269/294 
  Verifying        : sssd-ipa-2.5.2-2.el8.x86_64                                                                                    270/294 
  Verifying        : libsss_autofs-2.5.2-2.el8_5.4.x86_64                                                                           271/294 
  Verifying        : libsss_autofs-2.5.2-2.el8.x86_64                                                                               272/294 
  Verifying        : libsss_nss_idmap-2.5.2-2.el8_5.4.x86_64                                                                        273/294 
  Verifying        : libsss_nss_idmap-2.5.2-2.el8.x86_64                                                                            274/294 
  Verifying        : python3-rpm-4.14.3-19.el8_5.2.x86_64                                                                           275/294 
  Verifying        : python3-rpm-4.14.3-19.el8.x86_64                                                                               276/294 
  Verifying        : python3-dnf-plugins-core-4.0.21-4.el8_5.noarch                                                                 277/294 
  Verifying        : python3-dnf-plugins-core-4.0.21-3.el8.noarch                                                                   278/294 
  Verifying        : rpm-libs-4.14.3-19.el8_5.2.x86_64                                                                              279/294 
  Verifying        : rpm-libs-4.14.3-19.el8.x86_64                                                                                  280/294 
  Verifying        : cryptsetup-libs-2.3.3-4.el8_5.1.x86_64                                                                         281/294 
  Verifying        : cryptsetup-libs-2.3.3-4.el8.x86_64                                                                             282/294 
  Verifying        : rpm-plugin-selinux-4.14.3-19.el8_5.2.x86_64                                                                    283/294 
  Verifying        : rpm-plugin-selinux-4.14.3-19.el8.x86_64                                                                        284/294 
  Verifying        : dnf-plugins-core-4.0.21-4.el8_5.noarch                                                                         285/294 
  Verifying        : dnf-plugins-core-4.0.21-3.el8.noarch                                                                           286/294 
  Verifying        : sssd-proxy-2.5.2-2.el8_5.4.x86_64                                                                              287/294 
  Verifying        : sssd-proxy-2.5.2-2.el8.x86_64                                                                                  288/294 
  Verifying        : libsss_certmap-2.5.2-2.el8_5.4.x86_64                                                                          289/294 
  Verifying        : libsss_certmap-2.5.2-2.el8.x86_64                                                                              290/294 
  Verifying        : rpm-4.14.3-19.el8_5.2.x86_64                                                                                   291/294 
  Verifying        : rpm-4.14.3-19.el8.x86_64                                                                                       292/294 
  Verifying        : libipa_hbac-2.5.2-2.el8_5.4.x86_64                                                                             293/294 
  Verifying        : libipa_hbac-2.5.2-2.el8.x86_64                                                                                 294/294 
Installed products updated.

Upgraded:
  accountsservice-0.6.55-2.el8_5.2.x86_64                                                                                                   
  accountsservice-libs-0.6.55-2.el8_5.2.x86_64                                                                                              
  binutils-2.30-108.el8_5.1.x86_64                                                                                                          
  bpftool-4.18.0-348.12.2.el8_5.x86_64                                                                                                      
  clevis-15-1.el8_5.1.x86_64                                                                                                                
  clevis-luks-15-1.el8_5.1.x86_64                                                                                                           
  cockpit-251.3-1.el8_5.x86_64                                                                                                              
  cockpit-bridge-251.3-1.el8_5.x86_64                                                                                                       
  cockpit-system-251.3-1.el8_5.noarch                                                                                                       
  cockpit-ws-251.3-1.el8_5.x86_64                                                                                                           
  cpp-8.5.0-4.el8_5.x86_64                                                                                                                  
  cryptsetup-2.3.3-4.el8_5.1.x86_64                                                                                                         
  cryptsetup-libs-2.3.3-4.el8_5.1.x86_64                                                                                                    
  dnf-plugins-core-4.0.21-4.el8_5.noarch                                                                                                    
  firefox-91.5.0-1.el8_5.x86_64                                                                                                             
  flatpak-1.8.5-5.el8_5.x86_64                                                                                                              
  flatpak-libs-1.8.5-5.el8_5.x86_64                                                                                                         
  flatpak-selinux-1.8.5-5.el8_5.noarch                                                                                                      
  flatpak-session-helper-1.8.5-5.el8_5.x86_64                                                                                               
  freerdp-libs-2:2.2.0-7.el8_5.x86_64                                                                                                       
  fwupd-1.5.9-1.el8_4.x86_64                                                                                                                
  gcc-8.5.0-4.el8_5.x86_64                                                                                                                  
  gnome-classic-session-3.32.1-22.el8_5.noarch                                                                                              
  gnome-control-center-3.28.2-29.el8_5.x86_64                                                                                               
  gnome-control-center-filesystem-3.28.2-29.el8_5.noarch                                                                                    
  gnome-shell-extension-apps-menu-3.32.1-22.el8_5.noarch                                                                                    
  gnome-shell-extension-common-3.32.1-22.el8_5.noarch                                                                                       
  gnome-shell-extension-desktop-icons-3.32.1-22.el8_5.noarch                                                                                
  gnome-shell-extension-horizontal-workspaces-3.32.1-22.el8_5.noarch                                                                        
  gnome-shell-extension-launch-new-instance-3.32.1-22.el8_5.noarch                                                                          
  gnome-shell-extension-places-menu-3.32.1-22.el8_5.noarch                                                                                  
  gnome-shell-extension-window-list-3.32.1-22.el8_5.noarch                                                                                  
  ibus-1.5.19-14.el8_5.x86_64                                                                                                               
  ibus-gtk2-1.5.19-14.el8_5.x86_64                                                                                                          
  ibus-gtk3-1.5.19-14.el8_5.x86_64                                                                                                          
  ibus-libs-1.5.19-14.el8_5.x86_64                                                                                                          
  ibus-setup-1.5.19-14.el8_5.noarch                                                                                                         
  insights-client-3.1.7-1.el8_5.noarch                                                                                                      
  java-1.8.0-openjdk-1:1.8.0.322.b06-2.el8_5.x86_64                                                                                         
  java-1.8.0-openjdk-headless-1:1.8.0.322.b06-2.el8_5.x86_64                                                                                
  kernel-headers-4.18.0-348.12.2.el8_5.x86_64                                                                                               
  kernel-tools-4.18.0-348.12.2.el8_5.x86_64                                                                                                 
  kernel-tools-libs-4.18.0-348.12.2.el8_5.x86_64                                                                                            
  kexec-tools-2.0.20-57.el8_5.1.x86_64                                                                                                      
  libgcc-8.5.0-4.el8_5.x86_64                                                                                                               
  libgomp-8.5.0-4.el8_5.x86_64                                                                                                              
  libipa_hbac-2.5.2-2.el8_5.4.x86_64                                                                                                        
  libsmbclient-4.14.5-9.el8_5.x86_64                                                                                                        
  libsss_autofs-2.5.2-2.el8_5.4.x86_64                                                                                                      
  libsss_certmap-2.5.2-2.el8_5.4.x86_64                                                                                                     
  libsss_idmap-2.5.2-2.el8_5.4.x86_64                                                                                                       
  libsss_nss_idmap-2.5.2-2.el8_5.4.x86_64                                                                                                   
  libsss_sudo-2.5.2-2.el8_5.4.x86_64                                                                                                        
  libstdc++-8.5.0-4.el8_5.x86_64                                                                                                            
  libvirt-daemon-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                                                            
  libvirt-daemon-config-network-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                                             
  libvirt-daemon-driver-interface-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                                           
  libvirt-daemon-driver-network-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                                             
  libvirt-daemon-driver-nodedev-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                                             
  libvirt-daemon-driver-nwfilter-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                                            
  libvirt-daemon-driver-qemu-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                                                
  libvirt-daemon-driver-secret-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                                              
  libvirt-daemon-driver-storage-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                                             
  libvirt-daemon-driver-storage-core-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                                        
  libvirt-daemon-driver-storage-disk-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                                        
  libvirt-daemon-driver-storage-gluster-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                                     
  libvirt-daemon-driver-storage-iscsi-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                                       
  libvirt-daemon-driver-storage-iscsi-direct-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                                
  libvirt-daemon-driver-storage-logical-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                                     
  libvirt-daemon-driver-storage-mpath-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                                       
  libvirt-daemon-driver-storage-rbd-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                                         
  libvirt-daemon-driver-storage-scsi-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                                        
  libvirt-daemon-kvm-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                                                        
  libvirt-libs-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                                                              
  libwbclient-4.14.5-9.el8_5.x86_64                                                                                                         
  libwinpr-2:2.2.0-7.el8_5.x86_64                                                                                                           
  nss-3.67.0-7.el8_5.x86_64                                                                                                                 
  nss-softokn-3.67.0-7.el8_5.x86_64                                                                                                         
  nss-softokn-freebl-3.67.0-7.el8_5.x86_64                                                                                                  
  nss-sysinit-3.67.0-7.el8_5.x86_64                                                                                                         
  nss-util-3.67.0-7.el8_5.x86_64                                                                                                            
  openssl-1:1.1.1k-5.el8_5.x86_64                                                                                                           
  openssl-libs-1:1.1.1k-5.el8_5.x86_64                                                                                                      
  ostree-2021.3-2.el8_5.x86_64                                                                                                              
  ostree-libs-2021.3-2.el8_5.x86_64                                                                                                         
  polkit-0.115-13.el8_5.1.x86_64                                                                                                            
  polkit-libs-0.115-13.el8_5.1.x86_64                                                                                                       
  poppler-20.11.0-3.el8_5.1.x86_64                                                                                                          
  poppler-glib-20.11.0-3.el8_5.1.x86_64                                                                                                     
  poppler-utils-20.11.0-3.el8_5.1.x86_64                                                                                                    
  ppp-2.4.7-26.el8_1.x86_64                                                                                                                 
  python3-dnf-plugins-core-4.0.21-4.el8_5.noarch                                                                                            
  python3-perf-4.18.0-348.12.2.el8_5.x86_64                                                                                                 
  python3-rpm-4.14.3-19.el8_5.2.x86_64                                                                                                      
  python3-sssdconfig-2.5.2-2.el8_5.4.noarch                                                                                                 
  qemu-guest-agent-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                                                                       
  qemu-img-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                                                                               
  qemu-kvm-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                                                                               
  qemu-kvm-block-curl-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                                                                    
  qemu-kvm-block-gluster-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                                                                 
  qemu-kvm-block-iscsi-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                                                                   
  qemu-kvm-block-rbd-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                                                                     
  qemu-kvm-block-ssh-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                                                                     
  qemu-kvm-common-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                                                                        
  qemu-kvm-core-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                                                                          
  rpm-4.14.3-19.el8_5.2.x86_64                                                                                                              
  rpm-build-libs-4.14.3-19.el8_5.2.x86_64                                                                                                   
  rpm-libs-4.14.3-19.el8_5.2.x86_64                                                                                                         
  rpm-plugin-selinux-4.14.3-19.el8_5.2.x86_64                                                                                               
  rpm-plugin-systemd-inhibit-4.14.3-19.el8_5.2.x86_64                                                                                       
  samba-client-4.14.5-9.el8_5.x86_64                                                                                                        
  samba-client-libs-4.14.5-9.el8_5.x86_64                                                                                                   
  samba-common-4.14.5-9.el8_5.noarch                                                                                                        
  samba-common-libs-4.14.5-9.el8_5.x86_64                                                                                                   
  selinux-policy-3.14.3-80.el8_5.2.noarch                                                                                                   
  selinux-policy-targeted-3.14.3-80.el8_5.2.noarch                                                                                          
  sos-4.1-9.el8_5.noarch                                                                                                                    
  sssd-2.5.2-2.el8_5.4.x86_64                                                                                                               
  sssd-ad-2.5.2-2.el8_5.4.x86_64                                                                                                            
  sssd-client-2.5.2-2.el8_5.4.x86_64                                                                                                        
  sssd-common-2.5.2-2.el8_5.4.x86_64                                                                                                        
  sssd-common-pac-2.5.2-2.el8_5.4.x86_64                                                                                                    
  sssd-ipa-2.5.2-2.el8_5.4.x86_64                                                                                                           
  sssd-kcm-2.5.2-2.el8_5.4.x86_64                                                                                                           
  sssd-krb5-2.5.2-2.el8_5.4.x86_64                                                                                                          
  sssd-krb5-common-2.5.2-2.el8_5.4.x86_64                                                                                                   
  sssd-ldap-2.5.2-2.el8_5.4.x86_64                                                                                                          
  sssd-nfs-idmap-2.5.2-2.el8_5.4.x86_64                                                                                                     
  sssd-proxy-2.5.2-2.el8_5.4.x86_64                                                                                                         
  sudo-1.8.29-7.el8_4.1.x86_64                                                                                                              
  systemd-239-51.el8_5.3.x86_64                                                                                                             
  systemd-container-239-51.el8_5.3.x86_64                                                                                                   
  systemd-libs-239-51.el8_5.3.x86_64                                                                                                        
  systemd-pam-239-51.el8_5.3.x86_64                                                                                                         
  systemd-udev-239-51.el8_5.3.x86_64                                                                                                        
  tcpdump-14:4.9.3-2.el8_5.1.x86_64                                                                                                         
  tigervnc-license-1.11.0-10.el8_5.noarch                                                                                                   
  tigervnc-server-minimal-1.11.0-10.el8_5.x86_64                                                                                            
  tzdata-2021e-1.el8.noarch                                                                                                                 
  tzdata-java-2021e-1.el8.noarch                                                                                                            
  unzip-6.0-45.el8_4.x86_64                                                                                                                 
  vim-common-2:8.0.1763-16.el8_5.4.x86_64                                                                                                   
  vim-enhanced-2:8.0.1763-16.el8_5.4.x86_64                                                                                                 
  vim-filesystem-2:8.0.1763-16.el8_5.4.noarch                                                                                               
  vim-minimal-2:8.0.1763-16.el8_5.4.x86_64                                                                                                  
Installed:
  kernel-4.18.0-348.12.2.el8_5.x86_64             kernel-core-4.18.0-348.12.2.el8_5.x86_64     kernel-devel-4.18.0-348.12.2.el8_5.x86_64    
  kernel-modules-4.18.0-348.12.2.el8_5.x86_64    

Complete!
[root@tektutor ~]# 
</pre>

Now since all the updates are done, let's proceed with ovirt installation

```
sudo yum -y install https://resources.ovirt.org/pub/yum-repo/ovirt-release44.rpm
```
The expected output is
<pre>
[root@tektutor ~]# yum -y install https://resources.ovirt.org/pub/yum-repo/ovirt-release44.rpm
Updating Subscription Management repositories.
Last metadata expiration check: 0:10:42 ago on Fri 11 Feb 2022 07:31:38 PM PST.
ovirt-release44.rpm                                                                       10 kB/s |  21 kB     00:02    
Dependencies resolved.
=========================================================================================================================
 Package                        Architecture          Version                          Repository                   Size
=========================================================================================================================
Installing:
 ovirt-release44                noarch                4.4.10.1-1.el8                   @commandline                 21 k

Transaction Summary
=========================================================================================================================
Install  1 Package

Total size: 21 k
Installed size: 31 k
Downloading Packages:
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                 1/1 
  Installing       : ovirt-release44-4.4.10.1-1.el8.noarch                                                           1/1 
  Running scriptlet: ovirt-release44-4.4.10.1-1.el8.noarch                                                           1/1 
  Verifying        : ovirt-release44-4.4.10.1-1.el8.noarch                                                           1/1 
Installed products updated.

Installed:
  ovirt-release44-4.4.10.1-1.el8.noarch                                                                                  

Complete!
[root@tektutor ~]# 
</pre>

#### Install Java packages
```
sudo yum module -y enable javapackages-tools
```
The expected output is
<pre>
[root@tektutor ~]# yum module -y enable javapackages-tools
Updating Subscription Management repositories.
CentOS-$stream - Ceph Pacific                                                            461 kB/s | 375 kB     00:00    
determining the fastest mirror (12 hosts).. done.             ===                      ] ---  B/s |   0  B     --:-- ETA
Latest oVirt 4.4 Release                                                                 216 kB/s | 2.7 MB     00:12    
Extra Packages for Enterprise Linux 8 - x86_64                                           7.4 MB/s |  11 MB     00:01    
CentOS-8 - Gluster 8                                                                      53 kB/s |  42 kB     00:00    
virtio-win builds roughly matching what will be shipped in upcoming RHEL                  43 kB/s | 143 kB     00:03    
Copr repo for EL8_collection owned by sbonazzo                                            47 kB/s | 246 kB     00:05    
Copr repo for gluster-ansible owned by sac                                               2.4 kB/s | 7.3 kB     00:02    
CentOS-8 - Advanced Virtualization                                                       255 kB/s | 207 kB     00:00    
CentOS-8 - oVirt 4.4                                                                      18 kB/s | 1.2 MB     01:05    
CentOS-8 - OpsTools - collectd                                                            47 kB/s |  41 kB     00:00    
CentOS-8 - OpsTools - collectd - vault                                                    34 kB/s | 149 kB     00:04    
CentOS-8 - NFV OpenvSwitch                                                               143 kB/s | 107 kB     00:00    
CentOS-8 - OpenStack victoria                                                            3.1 MB/s | 2.7 MB     00:00    
Error: Problems in request:
missing groups or modules: javapackages-tools
[root@tektutor ~]# 
</pre>

##### Enabled pki dependencies
```
sudo yum module -y enable pki-deps
```
The expected output is
<pre>
[root@tektutor ~]# yum module -y enable pki-deps
Updating Subscription Management repositories.
Last metadata expiration check: 0:02:08 ago on Fri 11 Feb 2022 07:46:37 PM PST.
Dependencies resolved.
=========================================================================================================================
 Package                     Architecture               Version                        Repository                   Size
=========================================================================================================================
Enabling module streams:
 pki-deps                                               10.6                                                            

Transaction Summary
=========================================================================================================================

Complete!
</pre>

#### Install postgresql v12
```
sudo yum module -y enable postgresql:12
```
The expected output is
<pre>
[root@tektutor ~]# yum module -y enable postgresql:12
Updating Subscription Management repositories.
Last metadata expiration check: 0:14:29 ago on Fri 11 Feb 2022 07:46:37 PM PST.
Dependencies resolved.
=========================================================================================================================
 Package                     Architecture               Version                        Repository                   Size
=========================================================================================================================
Enabling module streams:
 postgresql                                             12                                                              

Transaction Summary
=========================================================================================================================

Complete!
</pre>

### Let's update the software packages in RHEL 8.5
```
sudo yum update
```
The expected output is
<pre>
root@tektutor ~]# yum -y update
Updating Subscription Management repositories.
Last metadata expiration check: 0:15:05 ago on Fri 11 Feb 2022 07:46:37 PM PST.
Dependencies resolved.
=========================================================================================================================
 Package                                    Arch   Version             Repository                                   Size
=========================================================================================================================
Installing:
 libgfapi0                                  x86_64 8.6-2.el8s          ovirt-4.4-centos-gluster8                   125 k
     replacing  glusterfs-api.x86_64 6.0-56.4.el8
 libglusterfs0                              x86_64 8.6-2.el8s          ovirt-4.4-centos-gluster8                   350 k
     replacing  glusterfs-libs.x86_64 6.0-56.4.el8
Upgrading:
 glusterfs                                  x86_64 8.6-2.el8s          ovirt-4.4-centos-gluster8                   689 k
 glusterfs-cli                              x86_64 8.6-2.el8s          ovirt-4.4-centos-gluster8                   214 k
 glusterfs-client-xlators                   x86_64 8.6-2.el8s          ovirt-4.4-centos-gluster8                   899 k
 glusterfs-fuse                             x86_64 8.6-2.el8s          ovirt-4.4-centos-gluster8                   171 k
 jansson                                    x86_64 2.12-5.el8          ovirt-4.4-centos-opstools-vault              45 k
 librados2                                  x86_64 2:16.2.7-1.el8s     ovirt-4.4-centos-ceph-pacific               3.8 M
 librbd1                                    x86_64 2:16.2.7-1.el8s     ovirt-4.4-centos-ceph-pacific               4.2 M
 libvirt-daemon                             x86_64 7.6.0-6.el8s        ovirt-4.4-centos-advanced-virtualization    420 k
 libvirt-daemon-config-network              x86_64 7.6.0-6.el8s        ovirt-4.4-centos-advanced-virtualization     76 k
 libvirt-daemon-driver-interface            x86_64 7.6.0-6.el8s        ovirt-4.4-centos-advanced-virtualization    215 k
 libvirt-daemon-driver-network              x86_64 7.6.0-6.el8s        ovirt-4.4-centos-advanced-virtualization    245 k
 libvirt-daemon-driver-nodedev              x86_64 7.6.0-6.el8s        ovirt-4.4-centos-advanced-virtualization    225 k
 libvirt-daemon-driver-nwfilter             x86_64 7.6.0-6.el8s        ovirt-4.4-centos-advanced-virtualization    241 k
 libvirt-daemon-driver-qemu                 x86_64 7.6.0-6.el8s        ovirt-4.4-centos-advanced-virtualization    920 k
 libvirt-daemon-driver-secret               x86_64 7.6.0-6.el8s        ovirt-4.4-centos-advanced-virtualization    205 k
 libvirt-daemon-driver-storage              x86_64 7.6.0-6.el8s        ovirt-4.4-centos-advanced-virtualization     74 k
 libvirt-daemon-driver-storage-core         x86_64 7.6.0-6.el8s        ovirt-4.4-centos-advanced-virtualization    261 k
 libvirt-daemon-driver-storage-disk         x86_64 7.6.0-6.el8s        ovirt-4.4-centos-advanced-virtualization     85 k
 libvirt-daemon-driver-storage-gluster      x86_64 7.6.0-6.el8s        ovirt-4.4-centos-advanced-virtualization     87 k
 libvirt-daemon-driver-storage-iscsi        x86_64 7.6.0-6.el8s        ovirt-4.4-centos-advanced-virtualization     82 k
 libvirt-daemon-driver-storage-iscsi-direct x86_64 7.6.0-6.el8s        ovirt-4.4-centos-advanced-virtualization     84 k
 libvirt-daemon-driver-storage-logical      x86_64 7.6.0-6.el8s        ovirt-4.4-centos-advanced-virtualization     86 k
 libvirt-daemon-driver-storage-mpath        x86_64 7.6.0-6.el8s        ovirt-4.4-centos-advanced-virtualization     80 k
 libvirt-daemon-driver-storage-rbd          x86_64 7.6.0-6.el8s        ovirt-4.4-centos-advanced-virtualization     90 k
 libvirt-daemon-driver-storage-scsi         x86_64 7.6.0-6.el8s        ovirt-4.4-centos-advanced-virtualization     82 k
 libvirt-daemon-kvm                         x86_64 7.6.0-6.el8s        ovirt-4.4-centos-advanced-virtualization     74 k
 libvirt-libs                               x86_64 7.6.0-6.el8s        ovirt-4.4-centos-advanced-virtualization    4.5 M
 libzstd                                    x86_64 1.4.5-6.el8         ovirt-4.4-centos-openstack-victoria         335 k
 python3-cairo                              x86_64 1.18.1-2.el8        ovirt-4.4-copr:copr.fedorainfracloud.org:sbonazzo:EL8_collection
                                                                                                                    93 k
 python3-pexpect                            noarch 4.7.0-4.el8         ovirt-4.4-centos-ovirt44                    144 k
 python3-psutil                             x86_64 5.7.2-1.el8         ovirt-4.4-centos-openstack-victoria         419 k
 python3-pyparsing                          noarch 2.4.6-1.el8         ovirt-4.4-centos-openstack-victoria         161 k
 python3-pyyaml                             x86_64 5.1.2-3.el8         ovirt-4.4-copr:copr.fedorainfracloud.org:sbonazzo:EL8_collection
                                                                                                                   197 k
 python3-requests                           noarch 2.22.0-7.el8        ovirt-4.4-centos-openstack-victoria         123 k
 python3-six                                noarch 1.15.0-2.el8        ovirt-4.4-centos-openstack-victoria          39 k
 qemu-guest-agent                           x86_64 15:6.0.0-33.el8s    ovirt-4.4-centos-advanced-virtualization    195 k
 qemu-img                                   x86_64 15:6.0.0-33.el8s    ovirt-4.4-centos-advanced-virtualization    1.9 M
 qemu-kvm                                   x86_64 15:6.0.0-33.el8s    ovirt-4.4-centos-advanced-virtualization     21 k
 qemu-kvm-block-curl                        x86_64 15:6.0.0-33.el8s    ovirt-4.4-centos-advanced-virtualization     32 k
 qemu-kvm-block-gluster                     x86_64 15:6.0.0-33.el8s    ovirt-4.4-centos-advanced-virtualization     32 k
 qemu-kvm-block-iscsi                       x86_64 15:6.0.0-33.el8s    ovirt-4.4-centos-advanced-virtualization     39 k
 qemu-kvm-block-rbd                         x86_64 15:6.0.0-33.el8s    ovirt-4.4-centos-advanced-virtualization     32 k
 qemu-kvm-block-ssh                         x86_64 15:6.0.0-33.el8s    ovirt-4.4-centos-advanced-virtualization     33 k
 qemu-kvm-common                            x86_64 15:6.0.0-33.el8s    ovirt-4.4-centos-advanced-virtualization    880 k
 qemu-kvm-core                              x86_64 15:6.0.0-33.el8s    ovirt-4.4-centos-advanced-virtualization    3.2 M
 seabios-bin                                noarch 1.14.0-1.el8s       ovirt-4.4-centos-advanced-virtualization    131 k
 seavgabios-bin                             noarch 1.14.0-1.el8s       ovirt-4.4-centos-advanced-virtualization     41 k
Installing dependencies:
 autogen-libopts                            x86_64 5.18.12-8.el8       rhel-8-for-x86_64-appstream-rpms             75 k
 gnutls-dane                                x86_64 3.6.16-4.el8        rhel-8-for-x86_64-appstream-rpms             52 k
 gnutls-utils                               x86_64 3.6.16-4.el8        rhel-8-for-x86_64-appstream-rpms            348 k
 libgfchangelog0                            x86_64 8.6-2.el8s          ovirt-4.4-centos-gluster8                    67 k
     replacing  glusterfs-libs.x86_64 6.0-56.4.el8
 libgfrpc0                                  x86_64 8.6-2.el8s          ovirt-4.4-centos-gluster8                    89 k
     replacing  glusterfs-libs.x86_64 6.0-56.4.el8
 libgfxdr0                                  x86_64 8.6-2.el8s          ovirt-4.4-centos-gluster8                    61 k
     replacing  glusterfs-libs.x86_64 6.0-56.4.el8
 libglusterd0                               x86_64 8.6-2.el8s          ovirt-4.4-centos-gluster8                    45 k
     replacing  glusterfs-libs.x86_64 6.0-56.4.el8
 libtpms                                    x86_64 0.8.6-0.20210910git7a4d46a119.el8.0
                                                                       epel                                        369 k
 lttng-ust                                  x86_64 2.8.1-11.el8        rhel-8-for-x86_64-appstream-rpms            259 k
 mdevctl                                    noarch 0.81-1.el8          rhel-8-for-x86_64-appstream-rpms             33 k
 qemu-kvm-docs                              x86_64 15:6.0.0-33.el8s    ovirt-4.4-centos-advanced-virtualization    885 k
 qemu-kvm-hw-usbredir                       x86_64 15:6.0.0-33.el8s    ovirt-4.4-centos-advanced-virtualization     43 k
 qemu-kvm-ui-opengl                         x86_64 15:6.0.0-33.el8s    ovirt-4.4-centos-advanced-virtualization     34 k
 qemu-kvm-ui-spice                          x86_64 15:6.0.0-33.el8s    ovirt-4.4-centos-advanced-virtualization     82 k
 swtpm                                      x86_64 0.6.0-2.20210607gitea627b3.el8s
                                                                       ovirt-4.4-centos-advanced-virtualization     39 k
 swtpm-libs                                 x86_64 0.6.0-2.20210607gitea627b3.el8s
                                                                       ovirt-4.4-centos-advanced-virtualization     42 k
 swtpm-tools                                x86_64 0.6.0-2.20210607gitea627b3.el8s
                                                                       ovirt-4.4-centos-advanced-virtualization    109 k

Transaction Summary
=========================================================================================================================
Install  19 Packages
Upgrade  47 Packages

Total download size: 29 M
Downloading Packages:
(1/66): libgfchangelog0-8.6-2.el8s.x86_64.rpm                                            396 kB/s |  67 kB     00:00    
(2/66): libgfapi0-8.6-2.el8s.x86_64.rpm                                                  693 kB/s | 125 kB     00:00    
(3/66): libgfxdr0-8.6-2.el8s.x86_64.rpm                                                  1.5 MB/s |  61 kB     00:00    
(4/66): libgfrpc0-8.6-2.el8s.x86_64.rpm                                                  1.4 MB/s |  89 kB     00:00    
(5/66): libglusterd0-8.6-2.el8s.x86_64.rpm                                               1.1 MB/s |  45 kB     00:00    
(6/66): libglusterfs0-8.6-2.el8s.x86_64.rpm                                              4.3 MB/s | 350 kB     00:00    
(7/66): libtpms-0.8.6-0.20210910git7a4d46a119.el8.0.x86_64.rpm                           1.0 MB/s | 369 kB     00:00    
(8/66): qemu-kvm-hw-usbredir-6.0.0-33.el8s.x86_64.rpm                                    389 kB/s |  43 kB     00:00    
(9/66): qemu-kvm-ui-opengl-6.0.0-33.el8s.x86_64.rpm                                      505 kB/s |  34 kB     00:00    
(10/66): swtpm-0.6.0-2.20210607gitea627b3.el8s.x86_64.rpm                                644 kB/s |  39 kB     00:00    
(11/66): qemu-kvm-ui-spice-6.0.0-33.el8s.x86_64.rpm                                      1.2 MB/s |  82 kB     00:00    
(12/66): qemu-kvm-docs-6.0.0-33.el8s.x86_64.rpm                                          3.5 MB/s | 885 kB     00:00    
(13/66): swtpm-libs-0.6.0-2.20210607gitea627b3.el8s.x86_64.rpm                           1.9 MB/s |  42 kB     00:00    
(14/66): swtpm-tools-0.6.0-2.20210607gitea627b3.el8s.x86_64.rpm                          1.6 MB/s | 109 kB     00:00    
(15/66): mdevctl-0.81-1.el8.noarch.rpm                                                    67 kB/s |  33 kB     00:00    
(16/66): autogen-libopts-5.18.12-8.el8.x86_64.rpm                                        133 kB/s |  75 kB     00:00    
(17/66): lttng-ust-2.8.1-11.el8.x86_64.rpm                                               406 kB/s | 259 kB     00:00    
(18/66): gnutls-dane-3.6.16-4.el8.x86_64.rpm                                             143 kB/s |  52 kB     00:00    
(19/66): gnutls-utils-3.6.16-4.el8.x86_64.rpm                                            787 kB/s | 348 kB     00:00    
(20/66): glusterfs-8.6-2.el8s.x86_64.rpm                                                 5.9 MB/s | 689 kB     00:00    
(21/66): glusterfs-cli-8.6-2.el8s.x86_64.rpm                                             5.3 MB/s | 214 kB     00:00    
(22/66): glusterfs-client-xlators-8.6-2.el8s.x86_64.rpm                                   14 MB/s | 899 kB     00:00    
(23/66): glusterfs-fuse-8.6-2.el8s.x86_64.rpm                                            4.8 MB/s | 171 kB     00:00    
(24/66): librados2-16.2.7-1.el8s.x86_64.rpm                                              5.7 MB/s | 3.8 MB     00:00    
(25/66): librbd1-16.2.7-1.el8s.x86_64.rpm                                                1.4 MB/s | 4.2 MB     00:02    
(26/66): libvirt-daemon-7.6.0-6.el8s.x86_64.rpm                                          2.7 MB/s | 420 kB     00:00    
(27/66): libvirt-daemon-config-network-7.6.0-6.el8s.x86_64.rpm                           2.4 MB/s |  76 kB     00:00    
(28/66): libvirt-daemon-driver-interface-7.6.0-6.el8s.x86_64.rpm                         6.0 MB/s | 215 kB     00:00    
(29/66): libvirt-daemon-driver-network-7.6.0-6.el8s.x86_64.rpm                           8.0 MB/s | 245 kB     00:00    
(30/66): libvirt-daemon-driver-nodedev-7.6.0-6.el8s.x86_64.rpm                           5.2 MB/s | 225 kB     00:00    
(31/66): libvirt-daemon-driver-nwfilter-7.6.0-6.el8s.x86_64.rpm                          5.3 MB/s | 241 kB     00:00    
(32/66): libvirt-daemon-driver-qemu-7.6.0-6.el8s.x86_64.rpm                              9.0 MB/s | 920 kB     00:00    
(33/66): libvirt-daemon-driver-secret-7.6.0-6.el8s.x86_64.rpm                            7.4 MB/s | 205 kB     00:00    
(34/66): libvirt-daemon-driver-storage-7.6.0-6.el8s.x86_64.rpm                           3.2 MB/s |  74 kB     00:00    
(35/66): libvirt-daemon-driver-storage-core-7.6.0-6.el8s.x86_64.rpm                      8.6 MB/s | 261 kB     00:00    
(36/66): libvirt-daemon-driver-storage-disk-7.6.0-6.el8s.x86_64.rpm                      3.3 MB/s |  85 kB     00:00    
(37/66): libvirt-daemon-driver-storage-gluster-7.6.0-6.el8s.x86_64.rpm                   3.7 MB/s |  87 kB     00:00    
(38/66): libvirt-daemon-driver-storage-iscsi-7.6.0-6.el8s.x86_64.rpm                     3.4 MB/s |  82 kB     00:00    
(39/66): libvirt-daemon-driver-storage-iscsi-direct-7.6.0-6.el8s.x86_64.rpm              3.3 MB/s |  84 kB     00:00    
(40/66): libvirt-daemon-driver-storage-logical-7.6.0-6.el8s.x86_64.rpm                   3.5 MB/s |  86 kB     00:00    
(41/66): libvirt-daemon-driver-storage-mpath-7.6.0-6.el8s.x86_64.rpm                     3.2 MB/s |  80 kB     00:00    
(42/66): libvirt-daemon-driver-storage-rbd-7.6.0-6.el8s.x86_64.rpm                       3.8 MB/s |  90 kB     00:00    
(43/66): libvirt-daemon-driver-storage-scsi-7.6.0-6.el8s.x86_64.rpm                      3.4 MB/s |  82 kB     00:00    
(44/66): libvirt-daemon-kvm-7.6.0-6.el8s.x86_64.rpm                                      2.9 MB/s |  74 kB     00:00    
(45/66): libvirt-libs-7.6.0-6.el8s.x86_64.rpm                                             17 MB/s | 4.5 MB     00:00    
(46/66): qemu-guest-agent-6.0.0-33.el8s.x86_64.rpm                                       3.7 MB/s | 195 kB     00:00    
(47/66): qemu-img-6.0.0-33.el8s.x86_64.rpm                                                14 MB/s | 1.9 MB     00:00    
(48/66): qemu-kvm-6.0.0-33.el8s.x86_64.rpm                                               583 kB/s |  21 kB     00:00    
(49/66): qemu-kvm-block-curl-6.0.0-33.el8s.x86_64.rpm                                    1.2 MB/s |  32 kB     00:00    
(50/66): qemu-kvm-block-gluster-6.0.0-33.el8s.x86_64.rpm                                 1.5 MB/s |  32 kB     00:00    
(51/66): qemu-kvm-block-iscsi-6.0.0-33.el8s.x86_64.rpm                                   1.7 MB/s |  39 kB     00:00    
(52/66): qemu-kvm-block-rbd-6.0.0-33.el8s.x86_64.rpm                                     946 kB/s |  32 kB     00:00    
(53/66): qemu-kvm-block-ssh-6.0.0-33.el8s.x86_64.rpm                                     1.5 MB/s |  33 kB     00:00    
(54/66): qemu-kvm-common-6.0.0-33.el8s.x86_64.rpm                                        9.3 MB/s | 880 kB     00:00    
(55/66): qemu-kvm-core-6.0.0-33.el8s.x86_64.rpm                                           16 MB/s | 3.2 MB     00:00    
(56/66): python3-cairo-1.18.1-2.el8.x86_64.rpm                                            22 kB/s |  93 kB     00:04    
(57/66): seabios-bin-1.14.0-1.el8s.noarch.rpm                                            1.8 MB/s | 131 kB     00:00    
(58/66): seavgabios-bin-1.14.0-1.el8s.noarch.rpm                                         422 kB/s |  41 kB     00:00    
(59/66): python3-pexpect-4.7.0-4.el8.noarch.rpm                                          239 kB/s | 144 kB     00:00    
(60/66): jansson-2.12-5.el8.x86_64.rpm                                                    23 kB/s |  45 kB     00:01    
(61/66): libzstd-1.4.5-6.el8.x86_64.rpm                                                  208 kB/s | 335 kB     00:01    
(62/66): python3-pyparsing-2.4.6-1.el8.noarch.rpm                                        930 kB/s | 161 kB     00:00    
(63/66): python3-requests-2.22.0-7.el8.noarch.rpm                                        1.6 MB/s | 123 kB     00:00    
(64/66): python3-psutil-5.7.2-1.el8.x86_64.rpm                                           1.0 MB/s | 419 kB     00:00    
(65/66): python3-six-1.15.0-2.el8.noarch.rpm                                             747 kB/s |  39 kB     00:00    
(66/66): python3-pyyaml-5.1.2-3.el8.x86_64.rpm                                            25 kB/s | 197 kB     00:07    
-------------------------------------------------------------------------------------------------------------------------
Total                                                                                    1.9 MB/s |  29 MB     00:15     
Extra Packages for Enterprise Linux 8 - x86_64                                           1.3 MB/s | 1.6 kB     00:00    
Importing GPG key 0x2F86D6A1:
 Userid     : "Fedora EPEL (8) <epel@fedoraproject.org>"
 Fingerprint: 94E2 79EB 8D8F 25B2 1810 ADF1 21EA 45AB 2F86 D6A1
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-8
Key imported successfully
CentOS-8 - Gluster 8                                                                     714  B/s | 1.0 kB     00:01    
Importing GPG key 0xE451E5B5:
 Userid     : "CentOS Storage SIG (http://wiki.centos.org/SpecialInterestGroup/Storage) <security@centos.org>"
 Fingerprint: 7412 9C0B 173B 071A 3775 951A D4A2 E50B E451 E5B5
 From       : https://www.centos.org/keys/RPM-GPG-KEY-CentOS-SIG-Storage
Key imported successfully
CentOS-8 - Advanced Virtualization                                                       808  B/s | 1.0 kB     00:01    
Importing GPG key 0x61E8806C:
 Userid     : "CentOS Virtualization SIG (http://wiki.centos.org/SpecialInterestGroup/Virtualization) <security@centos.org>"
 Fingerprint: A7C8 E761 309D 2F1C 92C5 0B62 7AEB BE82 61E8 806C
 From       : https://www.centos.org/keys/RPM-GPG-KEY-CentOS-SIG-Virtualization
Key imported successfully
Copr repo for EL8_collection owned by sbonazzo                                           728  B/s | 1.0 kB     00:01    
Importing GPG key 0x119783D1:
 Userid     : "sbonazzo_EL8_collection (None) <sbonazzo#EL8_collection@copr.fedorahosted.org>"
 Fingerprint: 42A2 7AF4 DCA1 3DC7 02B2 E8BE 35CB 6F97 1197 83D1
 From       : https://copr-be.cloud.fedoraproject.org/results/sbonazzo/EL8_collection/pubkey.gpg
Key imported successfully
CentOS-8 - OpsTools - collectd - vault                                                   679  B/s | 1.0 kB     00:01    
Importing GPG key 0x51BC2A13:
 Userid     : "CentOS OpsTools SIG (https://wiki.centos.org/SpecialInterestGroup/OpsTools) <security@centos.org>"
 Fingerprint: 7872 8176 9AD7 3878 85EE A649 4FD9 5327 51BC 2A13
 From       : https://www.centos.org/keys/RPM-GPG-KEY-CentOS-SIG-OpsTools
Key imported successfully
CentOS-8 - OpenStack victoria                                                            739  B/s | 1.0 kB     00:01    
Importing GPG key 0x764429E6:
 Userid     : "CentOS Cloud SIG (http://wiki.centos.org/SpecialInterestGroup/Cloud) <security@centos.org>"
 Fingerprint: 736A F511 6D9C 40E2 AF6B 074B F9B9 FEE7 7644 29E6
 From       : https://www.centos.org/keys/RPM-GPG-KEY-CentOS-SIG-Cloud
Key imported successfully
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                 1/1 
  Running scriptlet: libvirt-libs-7.6.0-6.el8s.x86_64                                                                1/1 
  Upgrading        : libvirt-libs-7.6.0-6.el8s.x86_64                                                              1/115 
  Running scriptlet: libvirt-daemon-7.6.0-6.el8s.x86_64                                                            2/115 
  Upgrading        : libvirt-daemon-7.6.0-6.el8s.x86_64                                                            2/115 
  Running scriptlet: libvirt-daemon-7.6.0-6.el8s.x86_64                                                            2/115 
  Installing       : libgfxdr0-8.6-2.el8s.x86_64                                                                   3/115 
  Running scriptlet: libgfxdr0-8.6-2.el8s.x86_64                                                                   3/115 
  Installing       : libglusterfs0-8.6-2.el8s.x86_64                                                               4/115 
  Running scriptlet: libglusterfs0-8.6-2.el8s.x86_64                                                               4/115 
  Installing       : libgfrpc0-8.6-2.el8s.x86_64                                                                   5/115 
  Running scriptlet: libgfrpc0-8.6-2.el8s.x86_64                                                                   5/115 
  Upgrading        : qemu-img-15:6.0.0-33.el8s.x86_64                                                              6/115 
  Upgrading        : libvirt-daemon-driver-storage-core-7.6.0-6.el8s.x86_64                                        7/115 
  Installing       : libtpms-0.8.6-0.20210910git7a4d46a119.el8.0.x86_64                                            8/115 
  Upgrading        : glusterfs-client-xlators-8.6-2.el8s.x86_64                                                    9/115 
  Installing       : libgfapi0-8.6-2.el8s.x86_64                                                                  10/115 
  Running scriptlet: libgfapi0-8.6-2.el8s.x86_64                                                                  10/115 
  Upgrading        : libvirt-daemon-driver-network-7.6.0-6.el8s.x86_64                                            11/115 
  Running scriptlet: libvirt-daemon-driver-network-7.6.0-6.el8s.x86_64                                            11/115 
  Installing       : lttng-ust-2.8.1-11.el8.x86_64                                                                12/115 
  Running scriptlet: lttng-ust-2.8.1-11.el8.x86_64                                                                12/115 
  Upgrading        : librados2-2:16.2.7-1.el8s.x86_64                                                             13/115 
  Running scriptlet: librados2-2:16.2.7-1.el8s.x86_64                                                             13/115 
  Upgrading        : librbd1-2:16.2.7-1.el8s.x86_64                                                               14/115 
  Running scriptlet: librbd1-2:16.2.7-1.el8s.x86_64                                                               14/115 
  Upgrading        : libvirt-daemon-driver-storage-rbd-7.6.0-6.el8s.x86_64                                        15/115 
  Installing       : swtpm-libs-0.6.0-2.20210607gitea627b3.el8s.x86_64                                            16/115 
  Installing       : swtpm-0.6.0-2.20210607gitea627b3.el8s.x86_64                                                 17/115 
  Running scriptlet: swtpm-0.6.0-2.20210607gitea627b3.el8s.x86_64                                                 17/115 
  Upgrading        : libvirt-daemon-driver-storage-disk-7.6.0-6.el8s.x86_64                                       18/115 
  Upgrading        : libvirt-daemon-driver-storage-iscsi-7.6.0-6.el8s.x86_64                                      19/115 
  Upgrading        : libvirt-daemon-driver-storage-iscsi-direct-7.6.0-6.el8s.x86_64                               20/115 
  Upgrading        : libvirt-daemon-driver-storage-logical-7.6.0-6.el8s.x86_64                                    21/115 
  Upgrading        : libvirt-daemon-driver-storage-mpath-7.6.0-6.el8s.x86_64                                      22/115 
  Upgrading        : libvirt-daemon-driver-storage-scsi-7.6.0-6.el8s.x86_64                                       23/115 
  Running scriptlet: glusterfs-8.6-2.el8s.x86_64                                                                  24/115 
  Upgrading        : glusterfs-8.6-2.el8s.x86_64                                                                  24/115 
  Running scriptlet: glusterfs-8.6-2.el8s.x86_64                                                                  24/115 
  Installing       : libglusterd0-8.6-2.el8s.x86_64                                                               25/115 
  Running scriptlet: libglusterd0-8.6-2.el8s.x86_64                                                               25/115 
  Upgrading        : glusterfs-cli-8.6-2.el8s.x86_64                                                              26/115 
  Upgrading        : libvirt-daemon-driver-storage-gluster-7.6.0-6.el8s.x86_64                                    27/115 
  Upgrading        : libvirt-daemon-driver-storage-7.6.0-6.el8s.x86_64                                            28/115 
  Upgrading        : libvirt-daemon-driver-interface-7.6.0-6.el8s.x86_64                                          29/115 
  Upgrading        : libvirt-daemon-driver-nwfilter-7.6.0-6.el8s.x86_64                                           30/115 
  Upgrading        : libvirt-daemon-driver-secret-7.6.0-6.el8s.x86_64                                             31/115 
  Upgrading        : seavgabios-bin-1.14.0-1.el8s.noarch                                                          32/115 
  Upgrading        : seabios-bin-1.14.0-1.el8s.noarch                                                             33/115 
  Upgrading        : qemu-kvm-common-15:6.0.0-33.el8s.x86_64                                                      34/115 
  Running scriptlet: qemu-kvm-common-15:6.0.0-33.el8s.x86_64                                                      34/115 
  Installing       : qemu-kvm-ui-opengl-15:6.0.0-33.el8s.x86_64                                                   35/115 
  Installing       : qemu-kvm-ui-spice-15:6.0.0-33.el8s.x86_64                                                    36/115 
  Installing       : qemu-kvm-hw-usbredir-15:6.0.0-33.el8s.x86_64                                                 37/115 
  Upgrading        : qemu-kvm-block-curl-15:6.0.0-33.el8s.x86_64                                                  38/115 
  Upgrading        : qemu-kvm-block-gluster-15:6.0.0-33.el8s.x86_64                                               39/115 
  Upgrading        : qemu-kvm-block-iscsi-15:6.0.0-33.el8s.x86_64                                                 40/115 
  Upgrading        : qemu-kvm-block-rbd-15:6.0.0-33.el8s.x86_64                                                   41/115 
  Upgrading        : qemu-kvm-block-ssh-15:6.0.0-33.el8s.x86_64                                                   42/115 
  Upgrading        : qemu-kvm-core-15:6.0.0-33.el8s.x86_64                                                        43/115 
  Installing       : gnutls-dane-3.6.16-4.el8.x86_64                                                              44/115 
  Installing       : mdevctl-0.81-1.el8.noarch                                                                    45/115 
  Upgrading        : libvirt-daemon-driver-nodedev-7.6.0-6.el8s.x86_64                                            46/115 
  Installing       : autogen-libopts-5.18.12-8.el8.x86_64                                                         47/115 
  Installing       : gnutls-utils-3.6.16-4.el8.x86_64                                                             48/115 
  Installing       : swtpm-tools-0.6.0-2.20210607gitea627b3.el8s.x86_64                                           49/115 
  Running scriptlet: libvirt-daemon-driver-qemu-7.6.0-6.el8s.x86_64                                               50/115 
  Upgrading        : libvirt-daemon-driver-qemu-7.6.0-6.el8s.x86_64                                               50/115 
  Installing       : qemu-kvm-docs-15:6.0.0-33.el8s.x86_64                                                        51/115 
  Upgrading        : qemu-kvm-15:6.0.0-33.el8s.x86_64                                                             52/115 
  Upgrading        : libvirt-daemon-kvm-7.6.0-6.el8s.x86_64                                                       53/115 
  Upgrading        : glusterfs-fuse-8.6-2.el8s.x86_64                                                             54/115 
  Upgrading        : libvirt-daemon-config-network-7.6.0-6.el8s.x86_64                                            55/115 
  Running scriptlet: libvirt-daemon-config-network-7.6.0-6.el8s.x86_64                                            55/115 
  Installing       : libgfchangelog0-8.6-2.el8s.x86_64                                                            56/115 
  Running scriptlet: libgfchangelog0-8.6-2.el8s.x86_64                                                            56/115 
  Upgrading        : python3-six-1.15.0-2.el8.noarch                                                              57/115 
  Upgrading        : python3-requests-2.22.0-7.el8.noarch                                                         58/115 
  Upgrading        : python3-pyparsing-2.4.6-1.el8.noarch                                                         59/115 
  Upgrading        : python3-psutil-5.7.2-1.el8.x86_64                                                            60/115 
  Upgrading        : libzstd-1.4.5-6.el8.x86_64                                                                   61/115 
  Upgrading        : jansson-2.12-5.el8.x86_64                                                                    62/115 
  Upgrading        : python3-pexpect-4.7.0-4.el8.noarch                                                           63/115 
  Upgrading        : qemu-guest-agent-15:6.0.0-33.el8s.x86_64                                                     64/115 
  Running scriptlet: qemu-guest-agent-15:6.0.0-33.el8s.x86_64                                                     64/115 
  Upgrading        : python3-pyyaml-5.1.2-3.el8.x86_64                                                            65/115 
  Upgrading        : python3-cairo-1.18.1-2.el8.x86_64                                                            66/115 
  Cleanup          : libvirt-daemon-kvm-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                           67/115 
  Cleanup          : libvirt-daemon-driver-storage-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                68/115 
  Cleanup          : qemu-kvm-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                                  69/115 
  Cleanup          : libvirt-daemon-driver-qemu-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                   70/115 
  Cleanup          : libvirt-daemon-driver-interface-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64              71/115 
  Cleanup          : libvirt-daemon-driver-nodedev-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                72/115 
  Cleanup          : libvirt-daemon-driver-nwfilter-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64               73/115 
  Cleanup          : libvirt-daemon-driver-secret-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                 74/115 
  Cleanup          : libvirt-daemon-driver-storage-gluster-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64        75/115 
  Cleanup          : qemu-kvm-block-gluster-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                    76/115 
  Obsoleting       : glusterfs-api-6.0-56.4.el8.x86_64                                                            77/115 
  Cleanup          : libvirt-daemon-driver-storage-rbd-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64            78/115 
  Cleanup          : glusterfs-fuse-6.0-56.4.el8.x86_64                                                           79/115 
  Cleanup          : libvirt-daemon-config-network-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                80/115 
  Cleanup          : python3-six-1.11.0-8.el8.noarch                                                              81/115 
  Cleanup          : python3-requests-2.20.0-2.1.el8_1.noarch                                                     82/115 
  Cleanup          : python3-pyparsing-2.1.10-7.el8.noarch                                                        83/115 
  Cleanup          : python3-pexpect-4.3.1-3.el8.noarch                                                           84/115 
  Cleanup          : libvirt-daemon-driver-network-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                85/115 
  Running scriptlet: libvirt-daemon-driver-network-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                85/115 
  Cleanup          : libvirt-daemon-driver-storage-scsi-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64           86/115 
  Cleanup          : glusterfs-6.0-56.4.el8.x86_64                                                                87/115 
  Running scriptlet: glusterfs-6.0-56.4.el8.x86_64                                                                87/115 
  Cleanup          : glusterfs-client-xlators-6.0-56.4.el8.x86_64                                                 88/115 
  Cleanup          : glusterfs-cli-6.0-56.4.el8.x86_64                                                            89/115 
  Cleanup          : qemu-kvm-core-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                             90/115 
  Cleanup          : libvirt-daemon-driver-storage-disk-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64           91/115 
  Cleanup          : libvirt-daemon-driver-storage-iscsi-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64          92/115 
  Cleanup          : libvirt-daemon-driver-storage-iscsi-direct-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_6    93/115 
  Cleanup          : libvirt-daemon-driver-storage-logical-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64        94/115 
  Cleanup          : libvirt-daemon-driver-storage-mpath-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64          95/115 
  Cleanup          : libvirt-daemon-driver-storage-core-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64           96/115 
  Running scriptlet: libvirt-daemon-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                               97/115 
  Cleanup          : libvirt-daemon-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                               97/115 
  Running scriptlet: libvirt-daemon-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                               97/115 
  Cleanup          : qemu-kvm-block-rbd-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                        98/115 
  Cleanup          : librbd1-1:12.2.7-9.el8.x86_64                                                                99/115 
  Running scriptlet: librbd1-1:12.2.7-9.el8.x86_64                                                                99/115 
  Cleanup          : qemu-kvm-block-curl-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                      100/115 
  Cleanup          : qemu-kvm-block-iscsi-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                     101/115 
  Cleanup          : qemu-kvm-block-ssh-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                       102/115 
  Cleanup          : seabios-bin-1.13.0-2.module+el8.3.0+7353+9de0a3cc.noarch                                    103/115 
  Cleanup          : seavgabios-bin-1.13.0-2.module+el8.3.0+7353+9de0a3cc.noarch                                 104/115 
  Running scriptlet: qemu-kvm-common-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                          105/115 
  Cleanup          : qemu-kvm-common-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                          105/115 
  Running scriptlet: qemu-kvm-common-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                          105/115 
  Cleanup          : librados2-1:12.2.7-9.el8.x86_64                                                             106/115 
  Running scriptlet: librados2-1:12.2.7-9.el8.x86_64                                                             106/115 
  Cleanup          : libvirt-libs-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                107/115 
  Running scriptlet: libvirt-libs-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                107/115 
  Cleanup          : qemu-img-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                                 108/115 
  Obsoleting       : glusterfs-libs-6.0-56.4.el8.x86_64                                                          109/115 
  Cleanup          : python3-psutil-5.4.3-11.el8.x86_64                                                          110/115 
  Cleanup          : libzstd-1.4.4-1.el8.x86_64                                                                  111/115 
  Cleanup          : jansson-2.11-3.el8.x86_64                                                                   112/115 
  Running scriptlet: qemu-guest-agent-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                         113/115 
  Cleanup          : qemu-guest-agent-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                         113/115 
  Running scriptlet: qemu-guest-agent-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                         113/115 
  Cleanup          : python3-pyyaml-3.12-12.el8.x86_64                                                           114/115 
  Cleanup          : python3-cairo-1.16.3-6.el8.x86_64                                                           115/115 
  Running scriptlet: libvirt-daemon-7.6.0-6.el8s.x86_64                                                          115/115 
  Running scriptlet: swtpm-0.6.0-2.20210607gitea627b3.el8s.x86_64                                                115/115 
  Running scriptlet: libvirt-daemon-config-network-7.6.0-6.el8s.x86_64                                           115/115 
  Running scriptlet: python3-cairo-1.16.3-6.el8.x86_64                                                           115/115 
  Verifying        : libtpms-0.8.6-0.20210910git7a4d46a119.el8.0.x86_64                                            1/115 
  Verifying        : libgfapi0-8.6-2.el8s.x86_64                                                                   2/115 
  Verifying        : glusterfs-api-6.0-56.4.el8.x86_64                                                             3/115 
  Verifying        : libgfchangelog0-8.6-2.el8s.x86_64                                                             4/115 
  Verifying        : glusterfs-libs-6.0-56.4.el8.x86_64                                                            5/115 
  Verifying        : libgfrpc0-8.6-2.el8s.x86_64                                                                   6/115 
  Verifying        : libgfxdr0-8.6-2.el8s.x86_64                                                                   7/115 
  Verifying        : libglusterd0-8.6-2.el8s.x86_64                                                                8/115 
  Verifying        : libglusterfs0-8.6-2.el8s.x86_64                                                               9/115 
  Verifying        : qemu-kvm-docs-15:6.0.0-33.el8s.x86_64                                                        10/115 
  Verifying        : qemu-kvm-hw-usbredir-15:6.0.0-33.el8s.x86_64                                                 11/115 
  Verifying        : qemu-kvm-ui-opengl-15:6.0.0-33.el8s.x86_64                                                   12/115 
  Verifying        : qemu-kvm-ui-spice-15:6.0.0-33.el8s.x86_64                                                    13/115 
  Verifying        : swtpm-0.6.0-2.20210607gitea627b3.el8s.x86_64                                                 14/115 
  Verifying        : swtpm-libs-0.6.0-2.20210607gitea627b3.el8s.x86_64                                            15/115 
  Verifying        : swtpm-tools-0.6.0-2.20210607gitea627b3.el8s.x86_64                                           16/115 
  Verifying        : lttng-ust-2.8.1-11.el8.x86_64                                                                17/115 
  Verifying        : autogen-libopts-5.18.12-8.el8.x86_64                                                         18/115 
  Verifying        : mdevctl-0.81-1.el8.noarch                                                                    19/115 
  Verifying        : gnutls-utils-3.6.16-4.el8.x86_64                                                             20/115 
  Verifying        : gnutls-dane-3.6.16-4.el8.x86_64                                                              21/115 
  Verifying        : librados2-2:16.2.7-1.el8s.x86_64                                                             22/115 
  Verifying        : librados2-1:12.2.7-9.el8.x86_64                                                              23/115 
  Verifying        : librbd1-2:16.2.7-1.el8s.x86_64                                                               24/115 
  Verifying        : librbd1-1:12.2.7-9.el8.x86_64                                                                25/115 
  Verifying        : glusterfs-8.6-2.el8s.x86_64                                                                  26/115 
  Verifying        : glusterfs-6.0-56.4.el8.x86_64                                                                27/115 
  Verifying        : glusterfs-cli-8.6-2.el8s.x86_64                                                              28/115 
  Verifying        : glusterfs-cli-6.0-56.4.el8.x86_64                                                            29/115 
  Verifying        : glusterfs-client-xlators-8.6-2.el8s.x86_64                                                   30/115 
  Verifying        : glusterfs-client-xlators-6.0-56.4.el8.x86_64                                                 31/115 
  Verifying        : glusterfs-fuse-8.6-2.el8s.x86_64                                                             32/115 
  Verifying        : glusterfs-fuse-6.0-56.4.el8.x86_64                                                           33/115 
  Verifying        : python3-cairo-1.18.1-2.el8.x86_64                                                            34/115 
  Verifying        : python3-cairo-1.16.3-6.el8.x86_64                                                            35/115 
  Verifying        : python3-pyyaml-5.1.2-3.el8.x86_64                                                            36/115 
  Verifying        : python3-pyyaml-3.12-12.el8.x86_64                                                            37/115 
  Verifying        : libvirt-daemon-7.6.0-6.el8s.x86_64                                                           38/115 
  Verifying        : libvirt-daemon-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                               39/115 
  Verifying        : libvirt-daemon-config-network-7.6.0-6.el8s.x86_64                                            40/115 
  Verifying        : libvirt-daemon-config-network-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                41/115 
  Verifying        : libvirt-daemon-driver-interface-7.6.0-6.el8s.x86_64                                          42/115 
  Verifying        : libvirt-daemon-driver-interface-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64              43/115 
  Verifying        : libvirt-daemon-driver-network-7.6.0-6.el8s.x86_64                                            44/115 
  Verifying        : libvirt-daemon-driver-network-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                45/115 
  Verifying        : libvirt-daemon-driver-nodedev-7.6.0-6.el8s.x86_64                                            46/115 
  Verifying        : libvirt-daemon-driver-nodedev-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                47/115 
  Verifying        : libvirt-daemon-driver-nwfilter-7.6.0-6.el8s.x86_64                                           48/115 
  Verifying        : libvirt-daemon-driver-nwfilter-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64               49/115 
  Verifying        : libvirt-daemon-driver-qemu-7.6.0-6.el8s.x86_64                                               50/115 
  Verifying        : libvirt-daemon-driver-qemu-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                   51/115 
  Verifying        : libvirt-daemon-driver-secret-7.6.0-6.el8s.x86_64                                             52/115 
  Verifying        : libvirt-daemon-driver-secret-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                 53/115 
  Verifying        : libvirt-daemon-driver-storage-7.6.0-6.el8s.x86_64                                            54/115 
  Verifying        : libvirt-daemon-driver-storage-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                55/115 
  Verifying        : libvirt-daemon-driver-storage-core-7.6.0-6.el8s.x86_64                                       56/115 
  Verifying        : libvirt-daemon-driver-storage-core-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64           57/115 
  Verifying        : libvirt-daemon-driver-storage-disk-7.6.0-6.el8s.x86_64                                       58/115 
  Verifying        : libvirt-daemon-driver-storage-disk-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64           59/115 
  Verifying        : libvirt-daemon-driver-storage-gluster-7.6.0-6.el8s.x86_64                                    60/115 
  Verifying        : libvirt-daemon-driver-storage-gluster-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64        61/115 
  Verifying        : libvirt-daemon-driver-storage-iscsi-7.6.0-6.el8s.x86_64                                      62/115 
  Verifying        : libvirt-daemon-driver-storage-iscsi-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64          63/115 
  Verifying        : libvirt-daemon-driver-storage-iscsi-direct-7.6.0-6.el8s.x86_64                               64/115 
  Verifying        : libvirt-daemon-driver-storage-iscsi-direct-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_6    65/115 
  Verifying        : libvirt-daemon-driver-storage-logical-7.6.0-6.el8s.x86_64                                    66/115 
  Verifying        : libvirt-daemon-driver-storage-logical-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64        67/115 
  Verifying        : libvirt-daemon-driver-storage-mpath-7.6.0-6.el8s.x86_64                                      68/115 
  Verifying        : libvirt-daemon-driver-storage-mpath-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64          69/115 
  Verifying        : libvirt-daemon-driver-storage-rbd-7.6.0-6.el8s.x86_64                                        70/115 
  Verifying        : libvirt-daemon-driver-storage-rbd-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64            71/115 
  Verifying        : libvirt-daemon-driver-storage-scsi-7.6.0-6.el8s.x86_64                                       72/115 
  Verifying        : libvirt-daemon-driver-storage-scsi-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64           73/115 
  Verifying        : libvirt-daemon-kvm-7.6.0-6.el8s.x86_64                                                       74/115 
  Verifying        : libvirt-daemon-kvm-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                           75/115 
  Verifying        : libvirt-libs-7.6.0-6.el8s.x86_64                                                             76/115 
  Verifying        : libvirt-libs-6.0.0-37.1.module+el8.5.0+13858+39fdc467.x86_64                                 77/115 
  Verifying        : qemu-guest-agent-15:6.0.0-33.el8s.x86_64                                                     78/115 
  Verifying        : qemu-guest-agent-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                          79/115 
  Verifying        : qemu-img-15:6.0.0-33.el8s.x86_64                                                             80/115 
  Verifying        : qemu-img-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                                  81/115 
  Verifying        : qemu-kvm-15:6.0.0-33.el8s.x86_64                                                             82/115 
  Verifying        : qemu-kvm-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                                  83/115 
  Verifying        : qemu-kvm-block-curl-15:6.0.0-33.el8s.x86_64                                                  84/115 
  Verifying        : qemu-kvm-block-curl-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                       85/115 
  Verifying        : qemu-kvm-block-gluster-15:6.0.0-33.el8s.x86_64                                               86/115 
  Verifying        : qemu-kvm-block-gluster-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                    87/115 
  Verifying        : qemu-kvm-block-iscsi-15:6.0.0-33.el8s.x86_64                                                 88/115 
  Verifying        : qemu-kvm-block-iscsi-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                      89/115 
  Verifying        : qemu-kvm-block-rbd-15:6.0.0-33.el8s.x86_64                                                   90/115 
  Verifying        : qemu-kvm-block-rbd-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                        91/115 
  Verifying        : qemu-kvm-block-ssh-15:6.0.0-33.el8s.x86_64                                                   92/115 
  Verifying        : qemu-kvm-block-ssh-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                        93/115 
  Verifying        : qemu-kvm-common-15:6.0.0-33.el8s.x86_64                                                      94/115 
  Verifying        : qemu-kvm-common-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                           95/115 
  Verifying        : qemu-kvm-core-15:6.0.0-33.el8s.x86_64                                                        96/115 
  Verifying        : qemu-kvm-core-15:4.2.0-59.module+el8.5.0+13495+8166cdf8.1.x86_64                             97/115 
  Verifying        : seabios-bin-1.14.0-1.el8s.noarch                                                             98/115 
  Verifying        : seabios-bin-1.13.0-2.module+el8.3.0+7353+9de0a3cc.noarch                                     99/115 
  Verifying        : seavgabios-bin-1.14.0-1.el8s.noarch                                                         100/115 
  Verifying        : seavgabios-bin-1.13.0-2.module+el8.3.0+7353+9de0a3cc.noarch                                 101/115 
  Verifying        : python3-pexpect-4.7.0-4.el8.noarch                                                          102/115 
  Verifying        : python3-pexpect-4.3.1-3.el8.noarch                                                          103/115 
  Verifying        : jansson-2.12-5.el8.x86_64                                                                   104/115 
  Verifying        : jansson-2.11-3.el8.x86_64                                                                   105/115 
  Verifying        : libzstd-1.4.5-6.el8.x86_64                                                                  106/115 
  Verifying        : libzstd-1.4.4-1.el8.x86_64                                                                  107/115 
  Verifying        : python3-psutil-5.7.2-1.el8.x86_64                                                           108/115 
  Verifying        : python3-psutil-5.4.3-11.el8.x86_64                                                          109/115 
  Verifying        : python3-pyparsing-2.4.6-1.el8.noarch                                                        110/115 
  Verifying        : python3-pyparsing-2.1.10-7.el8.noarch                                                       111/115 
  Verifying        : python3-requests-2.22.0-7.el8.noarch                                                        112/115 
  Verifying        : python3-requests-2.20.0-2.1.el8_1.noarch                                                    113/115 
  Verifying        : python3-six-1.15.0-2.el8.noarch                                                             114/115 
  Verifying        : python3-six-1.11.0-8.el8.noarch                                                             115/115 
Installed products updated.

Upgraded:
  glusterfs-8.6-2.el8s.x86_64                                                                                            
  glusterfs-cli-8.6-2.el8s.x86_64                                                                                        
  glusterfs-client-xlators-8.6-2.el8s.x86_64                                                                             
  glusterfs-fuse-8.6-2.el8s.x86_64                                                                                       
  jansson-2.12-5.el8.x86_64                                                                                              
  librados2-2:16.2.7-1.el8s.x86_64                                                                                       
  librbd1-2:16.2.7-1.el8s.x86_64                                                                                         
  libvirt-daemon-7.6.0-6.el8s.x86_64                                                                                     
  libvirt-daemon-config-network-7.6.0-6.el8s.x86_64                                                                      
  libvirt-daemon-driver-interface-7.6.0-6.el8s.x86_64                                                                    
  libvirt-daemon-driver-network-7.6.0-6.el8s.x86_64                                                                      
  libvirt-daemon-driver-nodedev-7.6.0-6.el8s.x86_64                                                                      
  libvirt-daemon-driver-nwfilter-7.6.0-6.el8s.x86_64                                                                     
  libvirt-daemon-driver-qemu-7.6.0-6.el8s.x86_64                                                                         
  libvirt-daemon-driver-secret-7.6.0-6.el8s.x86_64                                                                       
  libvirt-daemon-driver-storage-7.6.0-6.el8s.x86_64                                                                      
  libvirt-daemon-driver-storage-core-7.6.0-6.el8s.x86_64                                                                 
  libvirt-daemon-driver-storage-disk-7.6.0-6.el8s.x86_64                                                                 
  libvirt-daemon-driver-storage-gluster-7.6.0-6.el8s.x86_64                                                              
  libvirt-daemon-driver-storage-iscsi-7.6.0-6.el8s.x86_64                                                                
  libvirt-daemon-driver-storage-iscsi-direct-7.6.0-6.el8s.x86_64                                                         
  libvirt-daemon-driver-storage-logical-7.6.0-6.el8s.x86_64                                                              
  libvirt-daemon-driver-storage-mpath-7.6.0-6.el8s.x86_64                                                                
  libvirt-daemon-driver-storage-rbd-7.6.0-6.el8s.x86_64                                                                  
  libvirt-daemon-driver-storage-scsi-7.6.0-6.el8s.x86_64                                                                 
  libvirt-daemon-kvm-7.6.0-6.el8s.x86_64                                                                                 
  libvirt-libs-7.6.0-6.el8s.x86_64                                                                                       
  libzstd-1.4.5-6.el8.x86_64                                                                                             
  python3-cairo-1.18.1-2.el8.x86_64                                                                                      
  python3-pexpect-4.7.0-4.el8.noarch                                                                                     
  python3-psutil-5.7.2-1.el8.x86_64                                                                                      
  python3-pyparsing-2.4.6-1.el8.noarch                                                                                   
  python3-pyyaml-5.1.2-3.el8.x86_64                                                                                      
  python3-requests-2.22.0-7.el8.noarch                                                                                   
  python3-six-1.15.0-2.el8.noarch                                                                                        
  qemu-guest-agent-15:6.0.0-33.el8s.x86_64                                                                               
  qemu-img-15:6.0.0-33.el8s.x86_64                                                                                       
  qemu-kvm-15:6.0.0-33.el8s.x86_64                                                                                       
  qemu-kvm-block-curl-15:6.0.0-33.el8s.x86_64                                                                            
  qemu-kvm-block-gluster-15:6.0.0-33.el8s.x86_64                                                                         
  qemu-kvm-block-iscsi-15:6.0.0-33.el8s.x86_64                                                                           
  qemu-kvm-block-rbd-15:6.0.0-33.el8s.x86_64                                                                             
  qemu-kvm-block-ssh-15:6.0.0-33.el8s.x86_64                                                                             
  qemu-kvm-common-15:6.0.0-33.el8s.x86_64                                                                                
  qemu-kvm-core-15:6.0.0-33.el8s.x86_64                                                                                  
  seabios-bin-1.14.0-1.el8s.noarch                                                                                       
  seavgabios-bin-1.14.0-1.el8s.noarch                                                                                    
Installed:
  autogen-libopts-5.18.12-8.el8.x86_64                        gnutls-dane-3.6.16-4.el8.x86_64                            
  gnutls-utils-3.6.16-4.el8.x86_64                            libgfapi0-8.6-2.el8s.x86_64                                
  libgfchangelog0-8.6-2.el8s.x86_64                           libgfrpc0-8.6-2.el8s.x86_64                                
  libgfxdr0-8.6-2.el8s.x86_64                                 libglusterd0-8.6-2.el8s.x86_64                             
  libglusterfs0-8.6-2.el8s.x86_64                             libtpms-0.8.6-0.20210910git7a4d46a119.el8.0.x86_64         
  lttng-ust-2.8.1-11.el8.x86_64                               mdevctl-0.81-1.el8.noarch                                  
  qemu-kvm-docs-15:6.0.0-33.el8s.x86_64                       qemu-kvm-hw-usbredir-15:6.0.0-33.el8s.x86_64               
  qemu-kvm-ui-opengl-15:6.0.0-33.el8s.x86_64                  qemu-kvm-ui-spice-15:6.0.0-33.el8s.x86_64                  
  swtpm-0.6.0-2.20210607gitea627b3.el8s.x86_64                swtpm-libs-0.6.0-2.20210607gitea627b3.el8s.x86_64          
  swtpm-tools-0.6.0-2.20210607gitea627b3.el8s.x86_64         

Complete!
</pre>

#### Installing ovirt engine
```
sudo dnf install -y ovirt-engine
```
The expected output is
<pre>
[root@tektutor ~]# dnf install -y ovirt-engine
Updating Subscription Management repositories.
Last metadata expiration check: 0:01:08 ago on Sat 12 Feb 2022 04:28:13 PM PST.
Dependencies resolved.
============================================================================================================================================
 Package                                    Arch   Version                                   Repository                                Size
============================================================================================================================================
Installing:
 ovirt-engine                               noarch 4.4.10.6-1.el8                            ovirt-4.4                                 13 M
Installing dependencies:
 ansible                                    noarch 2.9.27-2.el8                              ovirt-4.4-centos-ovirt44                  17 M
 ansible-runner-service                     noarch 1.0.7-1.el8                               ovirt-4.4-centos-ovirt44                  97 k
 aopalliance                                noarch 1.0-17.module+el8+2598+06babf2e           codeready-builder-for-rhel-8-x86_64-rpms  17 k
 apache-commons-codec                       noarch 1.11-3.module+el8+2598+06babf2e           codeready-builder-for-rhel-8-x86_64-rpms 289 k
 apache-commons-collections                 noarch 3.2.2-10.module+el8+2598+06babf2e         codeready-builder-for-rhel-8-x86_64-rpms 537 k
 apache-commons-compress                    noarch 1.18-1.module+el8+2598+06babf2e           codeready-builder-for-rhel-8-x86_64-rpms 526 k
 apache-commons-configuration               noarch 1.10-1.el8                                ovirt-4.4-centos-ovirt44                 353 k
 apache-commons-io                          noarch 1:2.6-3.module+el8+2598+06babf2e          codeready-builder-for-rhel-8-x86_64-rpms 224 k
 apache-commons-jxpath                      noarch 1.3-29.module+el8+2598+06babf2e           codeready-builder-for-rhel-8-x86_64-rpms 295 k
 apache-commons-lang                        noarch 2.6-21.module+el8+2598+06babf2e           codeready-builder-for-rhel-8-x86_64-rpms 282 k
 apache-commons-logging                     noarch 1.2-13.module+el8+2598+06babf2e           codeready-builder-for-rhel-8-x86_64-rpms  85 k
 apache-sshd                                noarch 2.6.0-2.el8                               ovirt-4.4-centos-ovirt44                 2.3 M
 apr                                        x86_64 1.6.3-12.el8                              rhel-8-for-x86_64-appstream-rpms         130 k
 apr-util                                   x86_64 1.6.1-6.el8                               rhel-8-for-x86_64-appstream-rpms         105 k
 bea-stax-api                               noarch 1.2.0-16.module+el8+2468+c564cec5         rhel-8-for-x86_64-appstream-rpms          37 k
 blosc                                      x86_64 1.17.0-1.el8                              ovirt-4.4-centos-openstack-victoria       60 k
 ceph-common                                x86_64 2:16.2.7-1.el8s                           ovirt-4.4-centos-ceph-pacific             24 M
 collectd                                   x86_64 5.12.0-7.el8s                             ovirt-4.4-centos-opstools                707 k
 collectd-disk                              x86_64 5.12.0-7.el8s                             ovirt-4.4-centos-opstools                 33 k
 collectd-postgresql                        x86_64 5.12.0-7.el8s                             ovirt-4.4-centos-opstools                 43 k
 collectd-write_http                        x86_64 5.12.0-7.el8s                             ovirt-4.4-centos-opstools                 42 k
 collectd-write_syslog                      x86_64 5.12.0-7.el8s                             ovirt-4.4-centos-opstools                 33 k
 ebay-cors-filter                           noarch 1.0.1-4.el8                               ovirt-4.4-centos-ovirt44                 105 k
 ed25519-java                               noarch 0.3.0-1.el8                               ovirt-4.4-centos-ovirt44                  70 k
 fuse3-libs                                 x86_64 3.2.1-12.el8                              rhel-8-for-x86_64-baseos-rpms             94 k
 git-core                                   x86_64 2.27.0-1.el8                              rhel-8-for-x86_64-appstream-rpms         5.7 M
 glassfish-fastinfoset                      noarch 1.2.13-9.module+el8+2468+c564cec5         rhel-8-for-x86_64-appstream-rpms         354 k
 glassfish-jaxb-api                         noarch 2.2.12-8.module+el8+2468+c564cec5         rhel-8-for-x86_64-appstream-rpms         102 k
 glassfish-jaxb-core                        noarch 2.2.11-11.module+el8+2468+c564cec5        rhel-8-for-x86_64-appstream-rpms         158 k
 glassfish-jaxb-runtime                     noarch 2.2.11-11.module+el8+2468+c564cec5        rhel-8-for-x86_64-appstream-rpms         936 k
 glassfish-jaxb-txw2                        noarch 2.2.11-11.module+el8+2468+c564cec5        rhel-8-for-x86_64-appstream-rpms          90 k
 gperftools-libs                            x86_64 2.9.1-1.el8s                              ovirt-4.4-centos-ceph-pacific            311 k
 grafana                                    x86_64 7.5.9-5.el8_5                             rhel-8-for-x86_64-appstream-rpms          41 M
 graphviz                                   x86_64 2.40.1-43.el8                             rhel-8-for-x86_64-appstream-rpms         1.7 M
 hdf5                                       x86_64 1.10.5-5.el8                              ovirt-4.4-centos-openstack-victoria      2.1 M
 httpcomponents-client                      noarch 4.5.5-4.module+el8+2598+06babf2e          codeready-builder-for-rhel-8-x86_64-rpms 718 k
 httpcomponents-core                        noarch 4.4.10-3.module+el8+2598+06babf2e         codeready-builder-for-rhel-8-x86_64-rpms 638 k
 httpd                                      x86_64 2.4.37-43.module+el8.5.0+13806+b30d9eec.1 rhel-8-for-x86_64-appstream-rpms         1.4 M
 httpd-filesystem                           noarch 2.4.37-43.module+el8.5.0+13806+b30d9eec.1 rhel-8-for-x86_64-appstream-rpms          40 k
 httpd-tools                                x86_64 2.4.37-43.module+el8.5.0+13806+b30d9eec.1 rhel-8-for-x86_64-appstream-rpms         107 k
 istack-commons-runtime                     noarch 2.21-9.el8+7                              rhel-8-for-x86_64-appstream-rpms          44 k
 jackson-annotations                        noarch 2.10.0-1.module+el8.2.0+5059+3eb3af25     rhel-8-for-x86_64-appstream-rpms          71 k
 jackson-core                               noarch 2.10.0-1.module+el8.2.0+5059+3eb3af25     rhel-8-for-x86_64-appstream-rpms         345 k
 jackson-databind                           noarch 2.10.0-1.module+el8.2.0+5059+3eb3af25     rhel-8-for-x86_64-appstream-rpms         1.3 M
 jackson-jaxrs-json-provider                noarch 2.9.9-1.module+el8.1.0+3832+9784644d      rhel-8-for-x86_64-appstream-rpms          24 k
 jackson-jaxrs-providers                    noarch 2.9.9-1.module+el8.1.0+3832+9784644d      rhel-8-for-x86_64-appstream-rpms          45 k
 jackson-module-jaxb-annotations            noarch 2.7.6-4.module+el8+2468+c564cec5          rhel-8-for-x86_64-appstream-rpms          46 k
 java-11-openjdk-headless                   x86_64 1:11.0.14.0.9-2.el8_5                     rhel-8-for-x86_64-appstream-rpms          40 M
 java-client-kubevirt                       noarch 0.5.0-1.el8                               ovirt-4.4                                 22 M
 jboss-annotations-1.2-api                  noarch 1.0.0-4.el8                               rhel-8-for-x86_64-appstream-rpms          41 k
 jboss-jaxrs-2.0-api                        noarch 1.0.0-6.el8                               rhel-8-for-x86_64-appstream-rpms         113 k
 jboss-logging                              noarch 3.3.0-5.el8                               rhel-8-for-x86_64-appstream-rpms          71 k
 jboss-logging-tools                        noarch 2.0.1-6.el8                               rhel-8-for-x86_64-appstream-rpms         174 k
 jcl-over-slf4j                             noarch 1.7.25-4.module+el8+2598+06babf2e         codeready-builder-for-rhel-8-x86_64-rpms  32 k
 jdeparser                                  noarch 2.0.0-5.el8                               rhel-8-for-x86_64-appstream-rpms         217 k
 leveldb                                    x86_64 1.20-1.el8s                               ovirt-4.4-centos-ceph-pacific            165 k
 libXaw                                     x86_64 1.0.13-10.el8                             rhel-8-for-x86_64-appstream-rpms         194 k
 libaec                                     x86_64 1.0.2-3.el8                               codeready-builder-for-rhel-8-x86_64-rpms  39 k
 libcephfs2                                 x86_64 2:16.2.7-1.el8s                           ovirt-4.4-centos-ceph-pacific            847 k
 libcgroup-tools                            x86_64 0.41-19.el8                               rhel-8-for-x86_64-baseos-rpms             93 k
 libgfortran                                x86_64 8.5.0-4.el8_5                             rhel-8-for-x86_64-baseos-rpms            643 k
 liblognorm                                 x86_64 2.0.5-2.el8                               rhel-8-for-x86_64-appstream-rpms          88 k
 liboath                                    x86_64 2.6.2-4.el8s                              ovirt-4.4-centos-ceph-pacific             59 k
 libpq                                      x86_64 13.3-1.el8_4                              rhel-8-for-x86_64-appstream-rpms         197 k
 libqhull                                   x86_64 2015.2-5.el8                              codeready-builder-for-rhel-8-x86_64-rpms 169 k
 libquadmath                                x86_64 8.5.0-4.el8_5                             rhel-8-for-x86_64-baseos-rpms            170 k
 librabbitmq                                x86_64 0.9.0-3.el8                               rhel-8-for-x86_64-baseos-rpms             47 k
 libradosstriper1                           x86_64 2:16.2.7-1.el8s                           ovirt-4.4-centos-ceph-pacific            526 k
 librdkafka                                 x86_64 0.11.4-1.el8                              rhel-8-for-x86_64-appstream-rpms         353 k
 librgw2                                    x86_64 2:16.2.7-1.el8s                           ovirt-4.4-centos-ceph-pacific            3.8 M
 libsodium                                  x86_64 1.0.18-2.el8                              ovirt-4.4-epel                           162 k
 libunwind                                  x86_64 1.4.0-5.el8s                              ovirt-4.4-centos-ceph-pacific             75 k
 mod_http2                                  x86_64 1.15.7-3.module+el8.4.0+8625+d397f3da     rhel-8-for-x86_64-appstream-rpms         154 k
 mod_ssl                                    x86_64 1:2.4.37-43.module+el8.5.0+13806+b30d9eec.1
                                                                                             rhel-8-for-x86_64-appstream-rpms         136 k
 nodejs                                     x86_64 1:10.24.0-1.module+el8.3.0+10166+b07ac28e rhel-8-for-x86_64-appstream-rpms         8.8 M
 novnc                                      noarch 1.1.0-6.el8                               ovirt-4.4-centos-ovirt44                 949 k
 npm                                        x86_64 1:6.14.11-1.10.24.0.1.module+el8.3.0+10166+b07ac28e
                                                                                             rhel-8-for-x86_64-appstream-rpms         3.7 M
 ongres-scram                               noarch 1.0.0~beta.2-5.el8                        rhel-8-for-x86_64-appstream-rpms          46 k
 ongres-scram-client                        noarch 1.0.0~beta.2-5.el8                        rhel-8-for-x86_64-appstream-rpms          24 k
 openblas                                   x86_64 0.3.12-1.el8                              rhel-8-for-x86_64-appstream-rpms         4.6 M
 openblas-threads                           x86_64 0.3.12-1.el8                              rhel-8-for-x86_64-appstream-rpms         4.7 M
 openstack-java-cinder-client               noarch 3.2.9-9.el8                               ovirt-4.4-centos-ovirt44                  38 k
 openstack-java-cinder-model                noarch 3.2.9-9.el8                               ovirt-4.4-centos-ovirt44                  38 k
 openstack-java-client                      noarch 3.2.9-9.el8                               ovirt-4.4-centos-ovirt44                  26 k
 openstack-java-glance-client               noarch 3.2.9-9.el8                               ovirt-4.4-centos-ovirt44                  37 k
 openstack-java-glance-model                noarch 3.2.9-9.el8                               ovirt-4.4-centos-ovirt44                  27 k
 openstack-java-keystone-client             noarch 3.2.9-9.el8                               ovirt-4.4-centos-ovirt44                  55 k
 openstack-java-keystone-model              noarch 3.2.9-9.el8                               ovirt-4.4-centos-ovirt44                  58 k
 openstack-java-quantum-client              noarch 3.2.9-9.el8                               ovirt-4.4-centos-ovirt44                  38 k
 openstack-java-quantum-model               noarch 3.2.9-9.el8                               ovirt-4.4-centos-ovirt44                  36 k
 openstack-java-resteasy-connector          noarch 3.2.9-9.el8                               ovirt-4.4-centos-ovirt44                  25 k
 openvswitch-selinux-extra-policy           noarch 1.0-28.el8                                ovirt-4.4-centos-nfv-openvswitch          15 k
 openvswitch2.11                            x86_64 2.11.3-90.el8s                            ovirt-4.4-centos-nfv-openvswitch          12 M
 otopi-common                               noarch 1.9.6-1.el8                               ovirt-4.4                                 94 k
 ovirt-ansible-collection                   noarch 1.6.6-1.el8                               ovirt-4.4                                289 k
 ovirt-cockpit-sso                          noarch 0.1.4-2.el8                               ovirt-4.4                                 21 k
 ovirt-dependencies                         noarch 4.4.2-1.el8                               ovirt-4.4                                 12 M
 ovirt-engine-backend                       noarch 4.4.10.6-1.el8                            ovirt-4.4                                7.3 M
 ovirt-engine-dbscripts                     noarch 4.4.10.6-1.el8                            ovirt-4.4                                353 k
 ovirt-engine-dwh                           noarch 4.4.10-1.el8                              ovirt-4.4                                2.2 M
 ovirt-engine-dwh-grafana-integration-setup noarch 4.4.10-1.el8                              ovirt-4.4                                 87 k
 ovirt-engine-dwh-setup                     noarch 4.4.10-1.el8                              ovirt-4.4                                 95 k
 ovirt-engine-extension-aaa-jdbc            noarch 1.2.0-1.el8                               ovirt-4.4                                193 k
 ovirt-engine-extensions-api                noarch 1.0.1-1.el8                               ovirt-4.4                                 51 k
 ovirt-engine-metrics                       noarch 1.4.4-1.el8                               ovirt-4.4                                 92 k
 ovirt-engine-restapi                       noarch 4.4.10.6-1.el8                            ovirt-4.4                                5.4 M
 ovirt-engine-setup                         noarch 4.4.10.6-1.el8                            ovirt-4.4                                 18 k
 ovirt-engine-setup-base                    noarch 4.4.10.6-1.el8                            ovirt-4.4                                115 k
 ovirt-engine-setup-plugin-cinderlib        noarch 4.4.10.6-1.el8                            ovirt-4.4                                 39 k
 ovirt-engine-setup-plugin-imageio          noarch 4.4.10.6-1.el8                            ovirt-4.4                                 26 k
 ovirt-engine-setup-plugin-ovirt-engine     noarch 4.4.10.6-1.el8                            ovirt-4.4                                203 k
 ovirt-engine-setup-plugin-ovirt-engine-common
                                            noarch 4.4.10.6-1.el8                            ovirt-4.4                                122 k
 ovirt-engine-setup-plugin-vmconsole-proxy-helper
                                            noarch 4.4.10.6-1.el8                            ovirt-4.4                                 38 k
 ovirt-engine-setup-plugin-websocket-proxy  noarch 4.4.10.6-1.el8                            ovirt-4.4                                 39 k
 ovirt-engine-tools                         noarch 4.4.10.6-1.el8                            ovirt-4.4                                325 k
 ovirt-engine-tools-backup                  noarch 4.4.10.6-1.el8                            ovirt-4.4                                 39 k
 ovirt-engine-ui-extensions                 noarch 1.2.7-1.el8                               ovirt-4.4                                 13 M
 ovirt-engine-vmconsole-proxy-helper        noarch 4.4.10.6-1.el8                            ovirt-4.4                                 25 k
 ovirt-engine-webadmin-portal               noarch 4.4.10.6-1.el8                            ovirt-4.4                                116 M
 ovirt-engine-websocket-proxy               noarch 4.4.10.6-1.el8                            ovirt-4.4                                 32 k
 ovirt-engine-wildfly                       x86_64 23.0.2-1.el8                              ovirt-4.4                                196 M
 ovirt-engine-wildfly-overlay               noarch 23.0.2-1.el8                              ovirt-4.4                                9.4 k
 ovirt-imageio-common                       x86_64 2.3.0-1.el8                               ovirt-4.4                                158 k
 ovirt-imageio-daemon                       x86_64 2.3.0-1.el8                               ovirt-4.4                                 15 k
 ovirt-openvswitch                          noarch 2.11-1.el8                                ovirt-4.4-centos-ovirt44                 7.6 k
 ovirt-openvswitch-ovn                      noarch 2.11-1.el8                                ovirt-4.4-centos-ovirt44                 6.5 k
 ovirt-openvswitch-ovn-central              noarch 2.11-1.el8                                ovirt-4.4-centos-ovirt44                 6.6 k
 ovirt-openvswitch-ovn-common               noarch 2.11-1.el8                                ovirt-4.4-centos-ovirt44                 6.6 k
 ovirt-provider-ovn                         noarch 1.2.34-1.el8                              ovirt-4.4                                148 k
 ovirt-python-openvswitch                   noarch 2.11-1.el8                                ovirt-4.4-centos-ovirt44                 6.6 k
 ovirt-vmconsole                            noarch 1.0.9-1.el8                               ovirt-4.4                                 39 k
 ovirt-vmconsole-proxy                      noarch 1.0.9-1.el8                               ovirt-4.4                                 24 k
 ovirt-web-ui                               noarch 1.7.2-1.el8                               ovirt-4.4                                 11 M
 ovn2.11                                    x86_64 2.11.1-57.el8s                            ovirt-4.4-centos-nfv-openvswitch         2.5 M
 ovn2.11-central                            x86_64 2.11.1-57.el8s                            ovirt-4.4-centos-nfv-openvswitch         966 k
 perl-Filter                                x86_64 2:1.58-2.el8                              rhel-8-for-x86_64-appstream-rpms          82 k
 perl-Text-Unidecode                        noarch 1.30-5.el8                                rhel-8-for-x86_64-appstream-rpms         149 k
 perl-XML-Parser                            x86_64 2.44-11.el8                               rhel-8-for-x86_64-appstream-rpms         226 k
 perl-XML-XPath                             noarch 1.42-3.el8                                rhel-8-for-x86_64-appstream-rpms          88 k
 perl-encoding                              x86_64 4:2.22-3.el8                              rhel-8-for-x86_64-appstream-rpms          68 k
 perl-open                                  noarch 1.11-420.el8                              rhel-8-for-x86_64-appstream-rpms          77 k
 pki-servlet-4.0-api                        noarch 1:9.0.30-3.module+el8.5.0+11388+9e95fe00  rhel-8-for-x86_64-appstream-rpms         282 k
 platform-python-devel                      x86_64 3.6.8-41.el8                              rhel-8-for-x86_64-appstream-rpms         249 k
 postgresql                                 x86_64 12.9-1.module+el8.5.0+13373+4554acc4      rhel-8-for-x86_64-appstream-rpms         1.5 M
 postgresql-contrib                         x86_64 12.9-1.module+el8.5.0+13373+4554acc4      rhel-8-for-x86_64-appstream-rpms         868 k
 postgresql-jdbc                            noarch 42.2.3-3.el8_2                            rhel-8-for-x86_64-appstream-rpms         710 k
 postgresql-server                          x86_64 12.9-1.module+el8.5.0+13373+4554acc4      rhel-8-for-x86_64-appstream-rpms         5.6 M
 publicsuffix-list                          noarch 20180723-1.el8                            rhel-8-for-x86_64-baseos-rpms             79 k
 python-oslo-concurrency-lang               noarch 4.3.1-1.el8                               ovirt-4.4-centos-openstack-victoria       13 k
 python-oslo-db-lang                        noarch 8.4.1-1.el8                               ovirt-4.4-centos-openstack-victoria       13 k
 python-oslo-i18n-lang                      noarch 5.0.1-2.el8                               ovirt-4.4-centos-openstack-victoria       13 k
 python-oslo-log-lang                       noarch 4.4.0-2.el8                               ovirt-4.4-centos-openstack-victoria       13 k
 python-oslo-middleware-lang                noarch 4.1.1-2.el8                               ovirt-4.4-centos-openstack-victoria       12 k
 python-oslo-privsep-lang                   noarch 2.4.0-2.el8                               ovirt-4.4-centos-openstack-victoria       12 k
 python-oslo-utils-lang                     noarch 4.6.0-2.el8                               ovirt-4.4-centos-openstack-victoria       12 k
 python-oslo-versionedobjects-lang          noarch 2.3.0-2.el8                               ovirt-4.4-centos-openstack-victoria       12 k
 python-rpm-macros                          noarch 3-41.el8                                  rhel-8-for-x86_64-appstream-rpms          15 k
 python-srpm-macros                         noarch 3-41.el8                                  rhel-8-for-x86_64-appstream-rpms          15 k
 python3-Bottleneck                         x86_64 1.2.1-13.el8                              ovirt-4.4-centos-openstack-victoria      131 k
 python3-PyMySQL                            noarch 0.10.1-2.module+el8.4.0+9657+a4b6a102     rhel-8-for-x86_64-appstream-rpms          98 k
 python3-alembic                            noarch 1.4.2-5.el8                               ovirt-4.4-centos-openstack-victoria      811 k
 python3-amqp                               noarch 2.6.1-1.el8                               ovirt-4.4-centos-openstack-victoria       96 k
 python3-aniso8601                          noarch 8.0.0-1.el8                               ovirt-4.4-centos-ovirt44                  79 k
 python3-ansible-runner                     noarch 1.4.6-1.el8                               ovirt-4.4-centos-ovirt44                 100 k
 python3-automaton                          noarch 2.2.0-1.el8                               ovirt-4.4-centos-openstack-victoria       41 k
 python3-babel                              noarch 2.5.1-7.el8                               rhel-8-for-x86_64-appstream-rpms         4.8 M
 python3-bcrypt                             x86_64 3.1.7-3.el8                               ovirt-4.4-centos-ovirt44                  44 k
 python3-cachetools                         noarch 4.1.1-2.el8                               ovirt-4.4-centos-openstack-victoria       34 k
 python3-ceph-argparse                      x86_64 2:16.2.7-1.el8s                           ovirt-4.4-centos-ceph-pacific             71 k
 python3-ceph-common                        x86_64 2:16.2.7-1.el8s                           ovirt-4.4-centos-ceph-pacific            109 k
 python3-cephfs                             x86_64 2:16.2.7-1.el8s                           ovirt-4.4-centos-ceph-pacific            240 k
 python3-cffi                               x86_64 1.11.5-5.el8                              rhel-8-for-x86_64-baseos-rpms            238 k
 python3-cinder-common                      noarch 1:17.2.0-1.el8                            ovirt-4.4-centos-openstack-victoria      3.9 M
 python3-cinderlib                          noarch 1:3.0.0-1.el8                             ovirt-4.4-centos-openstack-victoria       78 k
 python3-click                              noarch 6.7-8.el8                                 rhel-8-for-x86_64-appstream-rpms         131 k
 python3-cryptography                       x86_64 3.2.1-5.el8                               rhel-8-for-x86_64-baseos-rpms            559 k
 python3-cycler                             noarch 0.10.0-13.el8                             ovirt-4.4-centos-openstack-victoria       20 k
 python3-daemon                             noarch 2.2.3-7.el8                               ovirt-4.4-centos-ovirt44                  41 k
 python3-debtcollector                      noarch 1.22.0-2.el8                              ovirt-4.4-centos-ovirt44                  32 k
 python3-distro                             noarch 1.4.0-2.module+el8.1.0+3334+5cb623d7      rhel-8-for-x86_64-appstream-rpms          37 k
 python3-dnf-plugin-versionlock             noarch 4.0.21-4.el8_5                            rhel-8-for-x86_64-baseos-rpms             62 k
 python3-dns                                noarch 1.15.0-10.el8                             rhel-8-for-x86_64-baseos-rpms            253 k
 python3-docutils                           noarch 0.14-12.module+el8.1.0+3334+5cb623d7      rhel-8-for-x86_64-appstream-rpms         1.6 M
 python3-editor                             noarch 1.0.4-4.el8                               ovirt-4.4-centos-openstack-victoria       19 k
 python3-eventlet                           noarch 0.25.2-3.1.el8                            ovirt-4.4-centos-openstack-victoria      385 k
 python3-fasteners                          noarch 0.14.1-20.el8                             ovirt-4.4-centos-openstack-victoria       44 k
 python3-flask                              noarch 1:0.12.2-4.el8                            rhel-8-for-x86_64-appstream-rpms         141 k
 python3-flask-restful                      noarch 0.3.7-5.el8                               ovirt-4.4-centos-ovirt44                 122 k
 python3-fluidity-sm                        noarch 0.2.0-16.el8                              ovirt-4.4-centos-ovirt44                  20 k
 python3-funcsigs                           noarch 1.0.2-17.el8                              ovirt-4.4-centos-ovirt44                  30 k
 python3-futurist                           noarch 2.3.0-2.el8                               ovirt-4.4-centos-openstack-victoria       63 k
 python3-greenlet                           x86_64 0.4.13-4.el8                              rhel-8-for-x86_64-appstream-rpms          31 k
 python3-httplib2                           noarch 0.10.3-4.el8                              codeready-builder-for-rhel-8-x86_64-rpms 108 k
 python3-importlib-metadata                 noarch 1.7.0-1.el8                               ovirt-4.4-centos-openstack-victoria       46 k
 python3-iso8601                            noarch 0.1.12-3.el8                              ovirt-4.4-centos-openstack-victoria       25 k
 python3-itsdangerous                       noarch 0.24-14.el8                               rhel-8-for-x86_64-appstream-rpms          31 k
 python3-jinja2                             noarch 2.10.1-3.el8                              rhel-8-for-x86_64-appstream-rpms         538 k
 python3-jmespath                           noarch 0.9.0-11.el8                              rhel-8-for-x86_64-appstream-rpms          45 k
 python3-jsonschema                         noarch 2.6.0-4.el8                               rhel-8-for-x86_64-appstream-rpms          82 k
 python3-kazoo                              noarch 2.8.0-1.el8                               ovirt-4.4-centos-openstack-victoria      164 k
 python3-kiwisolver                         x86_64 1.1.0-4.el8                               ovirt-4.4-centos-openstack-victoria       78 k
 python3-kombu                              noarch 1:4.6.11-2.el8                            ovirt-4.4-centos-openstack-victoria      341 k
 python3-lexicon                            noarch 1.0.0-9.el8                               ovirt-4.4-centos-ovirt44                  19 k
 python3-lockfile                           noarch 1:0.11.0-16.el8                           ovirt-4.4-centos-ovirt44                  33 k
 python3-mako                               noarch 1.0.6-13.el8                              rhel-8-for-x86_64-appstream-rpms         157 k
 python3-markupsafe                         x86_64 0.23-19.el8                               rhel-8-for-x86_64-appstream-rpms          39 k
 python3-matplotlib                         x86_64 3.1.1-2.el8                               ovirt-4.4-centos-openstack-victoria      3.3 M
 python3-matplotlib-data                    noarch 3.1.1-2.el8                               ovirt-4.4-centos-openstack-victoria      1.8 M
 python3-matplotlib-data-fonts              noarch 3.1.1-2.el8                               ovirt-4.4-centos-openstack-victoria      2.4 M
 python3-matplotlib-tk                      x86_64 3.1.1-2.el8                               ovirt-4.4-centos-openstack-victoria       64 k
 python3-migrate                            noarch 0.13.0-1.el8                              ovirt-4.4-centos-openstack-victoria      238 k
 python3-mock                               noarch 2.0.0-11.el8                              codeready-builder-for-rhel-8-x86_64-rpms  59 k
 python3-mod_wsgi                           x86_64 4.6.4-4.el8                               rhel-8-for-x86_64-appstream-rpms         2.5 M
 python3-monotonic                          noarch 1.5-5.el8                                 ovirt-4.4-copr:copr.fedorainfracloud.org:sbonazzo:EL8_collection
                                                                                                                                       19 k
 python3-msgpack                            x86_64 1.0.0-2.el8                               ovirt-4.4-centos-openstack-victoria       93 k
 python3-netaddr                            noarch 0.7.19-8.el8                              rhel-8-for-x86_64-appstream-rpms         1.5 M
 python3-netifaces                          x86_64 0.10.6-4.el8                              rhel-8-for-x86_64-appstream-rpms          25 k
 python3-networkx                           noarch 2.5-1.el8                                 ovirt-4.4-centos-openstack-victoria      2.5 M
 python3-notario                            noarch 0.0.16-4.el8                              ovirt-4.4-centos-ovirt44                  82 k
 python3-numexpr                            x86_64 2.7.1-1.el8                               ovirt-4.4-centos-openstack-victoria      198 k
 python3-numpy                              x86_64 1:1.14.3-10.el8                           rhel-8-for-x86_64-appstream-rpms         3.7 M
 python3-numpy-f2py                         x86_64 1:1.14.3-10.el8                           rhel-8-for-x86_64-appstream-rpms         225 k
 python3-openvswitch2.11                    x86_64 2.11.3-90.el8s                            ovirt-4.4-centos-nfv-openvswitch         314 k
 python3-os-brick                           noarch 4.0.4-1.el8                               ovirt-4.4-centos-openstack-victoria      1.1 M
 python3-os-win                             noarch 5.2.0-1.el8                               ovirt-4.4-centos-openstack-victoria      449 k
 python3-oslo-concurrency                   noarch 4.3.1-1.el8                               ovirt-4.4-centos-openstack-victoria       41 k
 python3-oslo-config                        noarch 2:8.3.4-1.el8                             ovirt-4.4-centos-openstack-victoria      224 k
 python3-oslo-context                       noarch 3.1.2-1.el8                               ovirt-4.4-centos-openstack-victoria       25 k
 python3-oslo-db                            noarch 8.4.1-1.el8                               ovirt-4.4-centos-openstack-victoria      147 k
 python3-oslo-i18n                          noarch 5.0.1-2.el8                               ovirt-4.4-centos-openstack-victoria       58 k
 python3-oslo-log                           noarch 4.4.0-2.el8                               ovirt-4.4-centos-openstack-victoria       63 k
 python3-oslo-messaging                     noarch 12.5.2-1.el8                              ovirt-4.4-centos-openstack-victoria      226 k
 python3-oslo-middleware                    noarch 4.1.1-2.el8                               ovirt-4.4-centos-openstack-victoria       53 k
 python3-oslo-privsep                       noarch 2.4.0-2.el8                               ovirt-4.4-centos-openstack-victoria       40 k
 python3-oslo-rootwrap                      noarch 6.2.0-2.el8                               ovirt-4.4-centos-openstack-victoria       44 k
 python3-oslo-serialization                 noarch 4.0.1-2.el8                               ovirt-4.4-centos-openstack-victoria       32 k
 python3-oslo-service                       noarch 2.4.0-2.el8                               ovirt-4.4-centos-openstack-victoria       70 k
 python3-oslo-utils                         noarch 4.6.0-2.el8                               ovirt-4.4-centos-openstack-victoria       79 k
 python3-oslo-versionedobjects              noarch 2.3.0-2.el8                               ovirt-4.4-centos-openstack-victoria       78 k
 python3-otopi                              noarch 1.9.6-1.el8                               ovirt-4.4                                109 k
 python3-ovirt-engine-lib                   noarch 4.4.10.6-1.el8                            ovirt-4.4                                 40 k
 python3-ovirt-engine-sdk4                  x86_64 4.4.15-1.el8                              ovirt-4.4                                570 k
 python3-ovirt-setup-lib                    noarch 1.3.2-1.el8                               ovirt-4.4                                 25 k
 python3-ovsdbapp                           noarch 0.17.5-1.el8                              ovirt-4.4-centos-ovirt44                 115 k
 python3-packaging                          noarch 20.4-1.el8                                ovirt-4.4-centos-openstack-victoria       67 k
 python3-pandas                             x86_64 0.25.3-1.el8                              ovirt-4.4-centos-openstack-victoria       11 M
 python3-paramiko                           noarch 2.7.2-2.el8                               ovirt-4.4-centos-ovirt44                 314 k
 python3-passlib                            noarch 1.7.1-6.el8                               ovirt-4.4-centos-ovirt44                 747 k
 python3-paste                              noarch 3.2.4-1.el8                               ovirt-4.4-centos-openstack-victoria      811 k
 python3-paste-deploy                       noarch 2.1.0-3.el8                               ovirt-4.4-centos-openstack-victoria       40 k
 python3-pbr                                noarch 5.4.3-2.el8                               ovirt-4.4-centos-ovirt44                  90 k
 python3-pillow                             x86_64 5.1.1-16.el8                              rhel-8-for-x86_64-appstream-rpms         632 k
 python3-prettytable                        noarch 0.7.2-14.el8                              rhel-8-for-x86_64-appstream-rpms          44 k
 python3-psycopg2                           x86_64 2.7.5-7.el8                               rhel-8-for-x86_64-appstream-rpms         172 k
 python3-pyOpenSSL                          noarch 19.0.0-1.el8                              rhel-8-for-x86_64-appstream-rpms         103 k
 python3-pycparser                          noarch 2.14-14.el8                               rhel-8-for-x86_64-baseos-rpms            109 k
 python3-pydot                              noarch 1.4.1-1.el8                               ovirt-4.4-centos-openstack-victoria       51 k
 python3-pygraphviz                         x86_64 1.5-9.el8                                 ovirt-4.4-centos-openstack-victoria       91 k
 python3-pynacl                             x86_64 1.3.0-5.el8                               ovirt-4.4-epel                           100 k
 python3-pyngus                             noarch 2.3.0-4.el8                               ovirt-4.4-centos-openstack-victoria       54 k
 python3-qpid-proton                        x86_64 0.36.0-1.el8                              ovirt-4.4-epel                           413 k
 python3-rados                              x86_64 2:16.2.7-1.el8s                           ovirt-4.4-centos-ceph-pacific            412 k
 python3-rbd                                x86_64 2:16.2.7-1.el8s                           ovirt-4.4-centos-ceph-pacific            393 k
 python3-redis                              noarch 3.3.8-1.el8                               ovirt-4.4-centos-openstack-victoria      131 k
 python3-repoze-lru                         noarch 0.7-6.el8                                 ovirt-4.4-centos-openstack-victoria       33 k
 python3-requests_ntlm                      noarch 1.1.0-8.el8                               ovirt-4.4-copr:copr.fedorainfracloud.org:sbonazzo:EL8_collection
                                                                                                                                       19 k
 python3-rfc3986                            noarch 1.4.0-3.el8                               ovirt-4.4-centos-openstack-victoria       52 k
 python3-rgw                                x86_64 2:16.2.7-1.el8s                           ovirt-4.4-centos-ceph-pacific            140 k
 python3-routes                             noarch 2.4.1-12.el8                              ovirt-4.4-centos-openstack-victoria      196 k
 python3-rpm-generators                     noarch 5-7.el8                                   rhel-8-for-x86_64-appstream-rpms          25 k
 python3-rpm-macros                         noarch 3-41.el8                                  rhel-8-for-x86_64-appstream-rpms          14 k
 python3-scipy                              x86_64 1.0.0-21.module+el8.5.0+10916+41bd434d    rhel-8-for-x86_64-appstream-rpms          14 M
 python3-sqlalchemy                         x86_64 1.3.2-2.module+el8.3.0+6646+6b4b10ec      rhel-8-for-x86_64-appstream-rpms         1.9 M
 python3-sqlparse                           noarch 0.3.1-3.el8                               ovirt-4.4-centos-openstack-victoria       88 k
 python3-statsd                             noarch 3.2.1-16.el8                              ovirt-4.4-centos-openstack-victoria       35 k
 python3-stevedore                          noarch 3.2.2-2.el8                               ovirt-4.4-centos-openstack-victoria       67 k
 python3-tables                             x86_64 3.5.2-6.el8                               ovirt-4.4-centos-openstack-victoria      1.4 M
 python3-tabulate                           noarch 0.8.7-4.el8                               ovirt-4.4-centos-openstack-victoria       50 k
 python3-taskflow                           noarch 4.5.0-2.el8                               ovirt-4.4-centos-openstack-victoria      689 k
 python3-tempita                            noarch 0.5.1-25.el8                              ovirt-4.4-centos-openstack-victoria       39 k
 python3-tenacity                           noarch 6.2.0-1.el8                               ovirt-4.4-centos-openstack-victoria       50 k
 python3-tkinter                            x86_64 3.6.8-41.el8                              rhel-8-for-x86_64-appstream-rpms         372 k
 python3-tooz                               noarch 2.7.2-1.el8                               ovirt-4.4-centos-openstack-victoria      109 k
 python3-vine                               noarch 1.3.0-4.el8                               ovirt-4.4-centos-openstack-victoria       34 k
 python3-voluptuous                         noarch 0.11.7-2.el8                              ovirt-4.4-copr:copr.fedorainfracloud.org:sbonazzo:EL8_collection
                                                                                                                                       60 k
 python3-webob                              noarch 1.8.6-3.el8                               ovirt-4.4-centos-openstack-victoria      253 k
 python3-websocket-client                   noarch 0.56.0-5.el8                              ovirt-4.4-centos-ovirt44                  61 k
 python3-websockify                         noarch 0.8.0-15.el8                              ovirt-4.4-centos-ovirt44                  54 k
 python3-werkzeug                           noarch 0.12.2-4.el8                              rhel-8-for-x86_64-appstream-rpms         457 k
 python3-wrapt                              x86_64 1.11.2-4.el8                              ovirt-4.4-copr:copr.fedorainfracloud.org:sbonazzo:EL8_collection
                                                                                                                                       54 k
 python3-xmltodict                          noarch 0.12.0-4.el8                              ovirt-4.4-copr:copr.fedorainfracloud.org:sbonazzo:EL8_collection
                                                                                                                                       25 k
 python3-yappi                              x86_64 1.2.5-1.el8                               ovirt-4.4-centos-openstack-victoria       50 k
 python3-zake                               noarch 0.2.2-18.el8                              ovirt-4.4-centos-openstack-victoria       46 k
 python3-zipp                               noarch 0.5.1-3.el8                               ovirt-4.4-copr:copr.fedorainfracloud.org:sbonazzo:EL8_collection
                                                                                                                                       14 k
 python3-zstd                               x86_64 1.4.5.1-1.el8                             ovirt-4.4-centos-openstack-victoria       18 k
 qpid-proton-c                              x86_64 0.36.0-1.el8                              ovirt-4.4-epel                           218 k
 redhat-logos-httpd                         noarch 84.5-1.el8                                rhel-8-for-x86_64-baseos-rpms             29 k
 relaxngDatatype                            noarch 2011.1-7.module+el8+2468+c564cec5         rhel-8-for-x86_64-appstream-rpms          28 k
 resteasy                                   noarch 3.0.26-6.module+el8.4.0+8891+bb8828ef     rhel-8-for-x86_64-appstream-rpms         1.1 M
 rhel-system-roles                          noarch 1.7.3-2.el8                               rhel-8-for-x86_64-appstream-rpms         1.3 M
 rsyslog-elasticsearch                      x86_64 8.2102.0-5.el8                            rhel-8-for-x86_64-appstream-rpms          33 k
 rsyslog-mmjsonparse                        x86_64 8.2102.0-5.el8                            rhel-8-for-x86_64-appstream-rpms          22 k
 rsyslog-mmnormalize                        x86_64 8.2102.0-5.el8                            rhel-8-for-x86_64-appstream-rpms          23 k
 slf4j                                      noarch 1.7.25-4.module+el8+2598+06babf2e         codeready-builder-for-rhel-8-x86_64-rpms  77 k
 slf4j-jdk14                                noarch 1.7.25-4.module+el8+2598+06babf2e         codeready-builder-for-rhel-8-x86_64-rpms  25 k
 snmp4j                                     noarch 3.6.4-0.1.el8                             ovirt-4.4-centos-ovirt44                 627 k
 sshpass                                    x86_64 1.06-9.el8                                ovirt-4.4-epel                            27 k
 stax-ex                                    noarch 1.7.7-8.module+el8+2468+c564cec5          rhel-8-for-x86_64-appstream-rpms          56 k
 sysfsutils                                 x86_64 2.1.0-24.el8                              rhel-8-for-x86_64-appstream-rpms          49 k
 tcl                                        x86_64 1:8.6.8-2.el8                             rhel-8-for-x86_64-baseos-rpms            1.1 M
 texlive-base                               noarch 7:20180414-23.el8                         rhel-8-for-x86_64-appstream-rpms         2.4 M
 texlive-dvipng                             x86_64 7:20180414-23.el8                         rhel-8-for-x86_64-appstream-rpms         332 k
 texlive-kpathsea                           x86_64 7:20180414-23.el8                         rhel-8-for-x86_64-appstream-rpms         1.1 M
 texlive-lib                                x86_64 7:20180414-23.el8                         rhel-8-for-x86_64-appstream-rpms         541 k
 texlive-tetex                              noarch 7:20180414-23.el8                         rhel-8-for-x86_64-appstream-rpms         402 k
 texlive-texlive.infra                      noarch 7:20180414-23.el8                         rhel-8-for-x86_64-appstream-rpms         280 k
 tk                                         x86_64 1:8.6.8-1.el8                             rhel-8-for-x86_64-appstream-rpms         1.6 M
 uuid                                       x86_64 1.6.2-43.el8                              rhel-8-for-x86_64-appstream-rpms          64 k
 vdsm-jsonrpc-java                          noarch 1.6.0-1.el8                               ovirt-4.4                                130 k
 ws-commons-util                            noarch 1.0.2-1.el8                               ovirt-4.4-centos-ovirt44                  45 k
 xmlrpc-client                              noarch 3.1.3-1.el8                               ovirt-4.4-centos-ovirt44                  60 k
 xmlrpc-common                              noarch 3.1.3-1.el8                               ovirt-4.4-centos-ovirt44                 110 k
 xmlstreambuffer                            noarch 1.5.4-8.module+el8+2468+c564cec5          rhel-8-for-x86_64-appstream-rpms          87 k
 xorg-x11-fonts-ISO8859-1-100dpi            noarch 7.5-19.el8                                rhel-8-for-x86_64-appstream-rpms         1.1 M
 xsom                                       noarch 0-19.20110809svn.module+el8+2468+c564cec5 rhel-8-for-x86_64-appstream-rpms         399 k
Installing weak dependencies:
 apr-util-bdb                               x86_64 1.6.1-6.el8                               rhel-8-for-x86_64-appstream-rpms          25 k
 apr-util-openssl                           x86_64 1.6.1-6.el8                               rhel-8-for-x86_64-appstream-rpms          27 k
 grafana-pcp                                x86_64 3.1.0-1.el8                               rhel-8-for-x86_64-appstream-rpms         9.4 M
 nodejs-full-i18n                           x86_64 1:10.24.0-1.module+el8.3.0+10166+b07ac28e rhel-8-for-x86_64-appstream-rpms         7.3 M
 python3-invoke                             noarch 1.4.0-1.el8                               ovirt-4.4-centos-ovirt44                 157 k
 python3-pyasn1                             noarch 0.4.6-3.el8                               ovirt-4.4-centos-opstools-vault          140 k
 python3-wcwidth                            noarch 0.1.7-14.el8                              ovirt-4.4-copr:copr.fedorainfracloud.org:sbonazzo:EL8_collection
                                                                                                                                       33 k
 python3-winrm                              noarch 0.3.0-7.el8                               ovirt-4.4-copr:copr.fedorainfracloud.org:sbonazzo:EL8_collection
                                                                                                                                       60 k
Enabling module streams:
 httpd                                             2.4                                                                                     
 nodejs                                            10                                                                                      

Transaction Summary
============================================================================================================================================
Install  334 Packages

Total download size: 706 M
Installed size: 1.9 G
Downloading Packages:
(1/334): leveldb-1.20-1.el8s.x86_64.rpm                                                                     364 kB/s | 165 kB     00:00    
(2/334): gperftools-libs-2.9.1-1.el8s.x86_64.rpm                                                            571 kB/s | 311 kB     00:00    
(3/334): liboath-2.6.2-4.el8s.x86_64.rpm                                                                    457 kB/s |  59 kB     00:00    
(4/334): libcephfs2-16.2.7-1.el8s.x86_64.rpm                                                                2.6 MB/s | 847 kB     00:00    
(5/334): libradosstriper1-16.2.7-1.el8s.x86_64.rpm                                                          2.4 MB/s | 526 kB     00:00    
(6/334): libunwind-1.4.0-5.el8s.x86_64.rpm                                                                  570 kB/s |  75 kB     00:00    
(7/334): python3-ceph-argparse-16.2.7-1.el8s.x86_64.rpm                                                     501 kB/s |  71 kB     00:00    
(8/334): python3-ceph-common-16.2.7-1.el8s.x86_64.rpm                                                       657 kB/s | 109 kB     00:00    
(9/334): python3-cephfs-16.2.7-1.el8s.x86_64.rpm                                                            1.2 MB/s | 240 kB     00:00    
(10/334): python3-rados-16.2.7-1.el8s.x86_64.rpm                                                            1.8 MB/s | 412 kB     00:00    
(11/334): python3-rbd-16.2.7-1.el8s.x86_64.rpm                                                              1.9 MB/s | 393 kB     00:00    
(12/334): python3-rgw-16.2.7-1.el8s.x86_64.rpm                                                              714 kB/s | 140 kB     00:00    
(13/334): librgw2-16.2.7-1.el8s.x86_64.rpm                                                                  2.5 MB/s | 3.8 MB     00:01    
(14/334): ceph-common-16.2.7-1.el8s.x86_64.rpm                                                              6.8 MB/s |  24 MB     00:03    
(15/334): otopi-common-1.9.6-1.el8.noarch.rpm                                                                35 kB/s |  94 kB     00:02    
(16/334): ovirt-ansible-collection-1.6.6-1.el8.noarch.rpm                                                   129 kB/s | 289 kB     00:02    
(17/334): ovirt-cockpit-sso-0.1.4-2.el8.noarch.rpm                                                           22 kB/s |  21 kB     00:00    
(18/334): ovirt-dependencies-4.4.2-1.el8.noarch.rpm                                                         2.5 MB/s |  12 MB     00:04    
(19/334): java-client-kubevirt-0.5.0-1.el8.noarch.rpm                                                       2.6 MB/s |  22 MB     00:08    
(20/334): ovirt-engine-dbscripts-4.4.10.6-1.el8.noarch.rpm                                                  199 kB/s | 353 kB     00:01    
(21/334): ovirt-engine-4.4.10.6-1.el8.noarch.rpm                                                            1.6 MB/s |  13 MB     00:08    
(22/334): ovirt-engine-dwh-grafana-integration-setup-4.4.10-1.el8.noarch.rpm                                 69 kB/s |  87 kB     00:01    
(23/334): ovirt-engine-dwh-4.4.10-1.el8.noarch.rpm                                                          708 kB/s | 2.2 MB     00:03    
(24/334): ovirt-engine-dwh-setup-4.4.10-1.el8.noarch.rpm                                                     59 kB/s |  95 kB     00:01    
(25/334): ovirt-engine-extension-aaa-jdbc-1.2.0-1.el8.noarch.rpm                                            125 kB/s | 193 kB     00:01    
(26/334): ovirt-engine-extensions-api-1.0.1-1.el8.noarch.rpm                                                 41 kB/s |  51 kB     00:01    
(27/334): ovirt-engine-metrics-1.4.4-1.el8.noarch.rpm                                                        71 kB/s |  92 kB     00:01    
(28/334): ovirt-engine-backend-4.4.10.6-1.el8.noarch.rpm                                                    871 kB/s | 7.3 MB     00:08    
(29/334): ovirt-engine-setup-4.4.10.6-1.el8.noarch.rpm                                                       20 kB/s |  18 kB     00:00    
(30/334): ovirt-engine-setup-plugin-cinderlib-4.4.10.6-1.el8.noarch.rpm                                      41 kB/s |  39 kB     00:00    
(31/334): ovirt-engine-setup-base-4.4.10.6-1.el8.noarch.rpm                                                  74 kB/s | 115 kB     00:01    
(32/334): ovirt-engine-setup-plugin-imageio-4.4.10.6-1.el8.noarch.rpm                                        29 kB/s |  26 kB     00:00    
(33/334): ovirt-engine-setup-plugin-ovirt-engine-4.4.10.6-1.el8.noarch.rpm                                  110 kB/s | 203 kB     00:01    
(34/334): ovirt-engine-setup-plugin-ovirt-engine-common-4.4.10.6-1.el8.noarch.rpm                            79 kB/s | 122 kB     00:01    
(35/334): ovirt-engine-setup-plugin-vmconsole-proxy-helper-4.4.10.6-1.el8.noarch.rpm                         38 kB/s |  38 kB     00:00    
(36/334): ovirt-engine-restapi-4.4.10.6-1.el8.noarch.rpm                                                    1.0 MB/s | 5.4 MB     00:05    
(37/334): ovirt-engine-setup-plugin-websocket-proxy-4.4.10.6-1.el8.noarch.rpm                                41 kB/s |  39 kB     00:00    
(38/334): ovirt-engine-tools-backup-4.4.10.6-1.el8.noarch.rpm                                                43 kB/s |  39 kB     00:00    
(39/334): ovirt-engine-vmconsole-proxy-helper-4.4.10.6-1.el8.noarch.rpm                                      27 kB/s |  25 kB     00:00    
(40/334): ovirt-engine-tools-4.4.10.6-1.el8.noarch.rpm                                                      159 kB/s | 325 kB     00:02    
(41/334): ovirt-engine-websocket-proxy-4.4.10.6-1.el8.noarch.rpm                                             34 kB/s |  32 kB     00:00    
(42/334): ovirt-engine-ui-extensions-1.2.7-1.el8.noarch.rpm                                                 1.2 MB/s |  13 MB     00:10    
(43/334): ovirt-engine-wildfly-overlay-23.0.2-1.el8.noarch.rpm                                               15 kB/s | 9.4 kB     00:00    
(44/334): ovirt-imageio-common-2.3.0-1.el8.x86_64.rpm                                                        98 kB/s | 158 kB     00:01    
(45/334): ovirt-imageio-daemon-2.3.0-1.el8.x86_64.rpm                                                        16 kB/s |  15 kB     00:00    
(46/334): ovirt-provider-ovn-1.2.34-1.el8.noarch.rpm                                                         99 kB/s | 148 kB     00:01    
(47/334): ovirt-vmconsole-1.0.9-1.el8.noarch.rpm                                                             40 kB/s |  39 kB     00:00    
(48/334): ovirt-vmconsole-proxy-1.0.9-1.el8.noarch.rpm                                                       25 kB/s |  24 kB     00:00    
(49/334): ovirt-web-ui-1.7.2-1.el8.noarch.rpm                                                               1.0 MB/s |  11 MB     00:11    
(50/334): python3-otopi-1.9.6-1.el8.noarch.rpm                                                               72 kB/s | 109 kB     00:01    
(51/334): python3-ovirt-engine-lib-4.4.10.6-1.el8.noarch.rpm                                                 44 kB/s |  40 kB     00:00    
(52/334): python3-ovirt-engine-sdk4-4.4.15-1.el8.x86_64.rpm                                                 265 kB/s | 570 kB     00:02    
(53/334): python3-ovirt-setup-lib-1.3.2-1.el8.noarch.rpm                                                     26 kB/s |  25 kB     00:00    
(54/334): vdsm-jsonrpc-java-1.6.0-1.el8.noarch.rpm                                                           81 kB/s | 130 kB     00:01    
(55/334): libsodium-1.0.18-2.el8.x86_64.rpm                                                                 1.0 MB/s | 162 kB     00:00    
(56/334): python3-pynacl-1.3.0-5.el8.x86_64.rpm                                                             1.9 MB/s | 100 kB     00:00    
(57/334): python3-qpid-proton-0.36.0-1.el8.x86_64.rpm                                                       3.4 MB/s | 413 kB     00:00    
(58/334): qpid-proton-c-0.36.0-1.el8.x86_64.rpm                                                             2.7 MB/s | 218 kB     00:00    
(59/334): sshpass-1.06-9.el8.x86_64.rpm                                                                     649 kB/s |  27 kB     00:00    
(60/334): python3-monotonic-1.5-5.el8.noarch.rpm                                                            8.4 kB/s |  19 kB     00:02    
(61/334): python3-requests_ntlm-1.1.0-8.el8.noarch.rpm                                                       57 kB/s |  19 kB     00:00    
(62/334): python3-voluptuous-0.11.7-2.el8.noarch.rpm                                                         90 kB/s |  60 kB     00:00    
(63/334): python3-wcwidth-0.1.7-14.el8.noarch.rpm                                                            99 kB/s |  33 kB     00:00    
(64/334): python3-winrm-0.3.0-7.el8.noarch.rpm                                                              175 kB/s |  60 kB     00:00    
(65/334): python3-wrapt-1.11.2-4.el8.x86_64.rpm                                                             160 kB/s |  54 kB     00:00    
(66/334): python3-xmltodict-0.12.0-4.el8.noarch.rpm                                                          69 kB/s |  25 kB     00:00    
(67/334): python3-zipp-0.5.1-3.el8.noarch.rpm                                                                41 kB/s |  14 kB     00:00    
(68/334): ansible-2.9.27-2.el8.noarch.rpm                                                                   4.2 MB/s |  17 MB     00:04    
(69/334): ansible-runner-service-1.0.7-1.el8.noarch.rpm                                                     1.0 MB/s |  97 kB     00:00    
(70/334): apache-commons-configuration-1.10-1.el8.noarch.rpm                                                3.1 MB/s | 353 kB     00:00    
(71/334): apache-sshd-2.6.0-2.el8.noarch.rpm                                                                5.6 MB/s | 2.3 MB     00:00    
(72/334): ebay-cors-filter-1.0.1-4.el8.noarch.rpm                                                           1.1 MB/s | 105 kB     00:00    
(73/334): ed25519-java-0.3.0-1.el8.noarch.rpm                                                               758 kB/s |  70 kB     00:00    
(74/334): novnc-1.1.0-6.el8.noarch.rpm                                                                      4.6 MB/s | 949 kB     00:00    
(75/334): openstack-java-cinder-client-3.2.9-9.el8.noarch.rpm                                               436 kB/s |  38 kB     00:00    
(76/334): openstack-java-cinder-model-3.2.9-9.el8.noarch.rpm                                                419 kB/s |  38 kB     00:00    
(77/334): openstack-java-client-3.2.9-9.el8.noarch.rpm                                                      389 kB/s |  26 kB     00:00    
(78/334): openstack-java-glance-client-3.2.9-9.el8.noarch.rpm                                               410 kB/s |  37 kB     00:00    
(79/334): openstack-java-glance-model-3.2.9-9.el8.noarch.rpm                                                330 kB/s |  27 kB     00:00    
(80/334): openstack-java-keystone-client-3.2.9-9.el8.noarch.rpm                                             651 kB/s |  55 kB     00:00    
(81/334): openstack-java-keystone-model-3.2.9-9.el8.noarch.rpm                                              598 kB/s |  58 kB     00:00    
(82/334): openstack-java-quantum-client-3.2.9-9.el8.noarch.rpm                                              467 kB/s |  38 kB     00:00    
(83/334): openstack-java-quantum-model-3.2.9-9.el8.noarch.rpm                                               450 kB/s |  36 kB     00:00    
(84/334): openstack-java-resteasy-connector-3.2.9-9.el8.noarch.rpm                                          376 kB/s |  25 kB     00:00    
(85/334): ovirt-openvswitch-2.11-1.el8.noarch.rpm                                                           104 kB/s | 7.6 kB     00:00    
(86/334): ovirt-openvswitch-ovn-2.11-1.el8.noarch.rpm                                                        68 kB/s | 6.5 kB     00:00    
(87/334): ovirt-openvswitch-ovn-central-2.11-1.el8.noarch.rpm                                                90 kB/s | 6.6 kB     00:00    
(88/334): ovirt-openvswitch-ovn-common-2.11-1.el8.noarch.rpm                                                105 kB/s | 6.6 kB     00:00    
(89/334): ovirt-python-openvswitch-2.11-1.el8.noarch.rpm                                                     83 kB/s | 6.6 kB     00:00    
(90/334): python3-aniso8601-8.0.0-1.el8.noarch.rpm                                                          680 kB/s |  79 kB     00:00    
(91/334): python3-ansible-runner-1.4.6-1.el8.noarch.rpm                                                     1.1 MB/s | 100 kB     00:00    
(92/334): python3-bcrypt-3.1.7-3.el8.x86_64.rpm                                                             551 kB/s |  44 kB     00:00    
(93/334): python3-daemon-2.2.3-7.el8.noarch.rpm                                                             457 kB/s |  41 kB     00:00    
(94/334): python3-debtcollector-1.22.0-2.el8.noarch.rpm                                                     338 kB/s |  32 kB     00:00    
(95/334): python3-flask-restful-0.3.7-5.el8.noarch.rpm                                                      1.2 MB/s | 122 kB     00:00    
(96/334): python3-fluidity-sm-0.2.0-16.el8.noarch.rpm                                                       259 kB/s |  20 kB     00:00    
(97/334): python3-funcsigs-1.0.2-17.el8.noarch.rpm                                                          400 kB/s |  30 kB     00:00    
(98/334): python3-invoke-1.4.0-1.el8.noarch.rpm                                                             747 kB/s | 157 kB     00:00    
(99/334): python3-lexicon-1.0.0-9.el8.noarch.rpm                                                            243 kB/s |  19 kB     00:00    
(100/334): python3-lockfile-0.11.0-16.el8.noarch.rpm                                                        374 kB/s |  33 kB     00:00    
(101/334): python3-notario-0.0.16-4.el8.noarch.rpm                                                          824 kB/s |  82 kB     00:00    
(102/334): python3-ovsdbapp-0.17.5-1.el8.noarch.rpm                                                         1.1 MB/s | 115 kB     00:00    
(103/334): python3-paramiko-2.7.2-2.el8.noarch.rpm                                                          2.2 MB/s | 314 kB     00:00    
(104/334): python3-passlib-1.7.1-6.el8.noarch.rpm                                                           4.4 MB/s | 747 kB     00:00    
(105/334): python3-pbr-5.4.3-2.el8.noarch.rpm                                                               1.0 MB/s |  90 kB     00:00    
(106/334): python3-websocket-client-0.56.0-5.el8.noarch.rpm                                                 744 kB/s |  61 kB     00:00    
(107/334): python3-websockify-0.8.0-15.el8.noarch.rpm                                                       662 kB/s |  54 kB     00:00    
(108/334): snmp4j-3.6.4-0.1.el8.noarch.rpm                                                                  4.3 MB/s | 627 kB     00:00    
(109/334): ws-commons-util-1.0.2-1.el8.noarch.rpm                                                           503 kB/s |  45 kB     00:00    
(110/334): xmlrpc-client-3.1.3-1.el8.noarch.rpm                                                             604 kB/s |  60 kB     00:00    
(111/334): xmlrpc-common-3.1.3-1.el8.noarch.rpm                                                             1.0 MB/s | 110 kB     00:00    
(112/334): collectd-5.12.0-7.el8s.x86_64.rpm                                                                2.2 MB/s | 707 kB     00:00    
(113/334): collectd-disk-5.12.0-7.el8s.x86_64.rpm                                                           582 kB/s |  33 kB     00:00    
(114/334): collectd-postgresql-5.12.0-7.el8s.x86_64.rpm                                                     529 kB/s |  43 kB     00:00    
(115/334): collectd-write_http-5.12.0-7.el8s.x86_64.rpm                                                     650 kB/s |  42 kB     00:00    
(116/334): collectd-write_syslog-5.12.0-7.el8s.x86_64.rpm                                                   625 kB/s |  33 kB     00:00    
(117/334): ovirt-engine-webadmin-portal-4.4.10.6-1.el8.noarch.rpm                                           1.9 MB/s | 116 MB     01:02    
(118/334): openvswitch-selinux-extra-policy-1.0-28.el8.noarch.rpm                                           126 kB/s |  15 kB     00:00    
(119/334): openvswitch2.11-2.11.3-90.el8s.x86_64.rpm                                                        6.8 MB/s |  12 MB     00:01    
(120/334): ovn2.11-2.11.1-57.el8s.x86_64.rpm                                                                6.0 MB/s | 2.5 MB     00:00    
(121/334): ovn2.11-central-2.11.1-57.el8s.x86_64.rpm                                                        6.6 MB/s | 966 kB     00:00    
(122/334): python3-openvswitch2.11-2.11.3-90.el8s.x86_64.rpm                                                3.0 MB/s | 314 kB     00:00    
(123/334): blosc-1.17.0-1.el8.x86_64.rpm                                                                    418 kB/s |  60 kB     00:00    
(124/334): hdf5-1.10.5-5.el8.x86_64.rpm                                                                     4.7 MB/s | 2.1 MB     00:00    
(125/334): python-oslo-concurrency-lang-4.3.1-1.el8.noarch.rpm                                              202 kB/s |  13 kB     00:00    
(126/334): python-oslo-db-lang-8.4.1-1.el8.noarch.rpm                                                       236 kB/s |  13 kB     00:00    
(127/334): python-oslo-i18n-lang-5.0.1-2.el8.noarch.rpm                                                     308 kB/s |  13 kB     00:00    
(128/334): python-oslo-log-lang-4.4.0-2.el8.noarch.rpm                                                      241 kB/s |  13 kB     00:00    
(129/334): python-oslo-middleware-lang-4.1.1-2.el8.noarch.rpm                                               230 kB/s |  12 kB     00:00    
(130/334): python-oslo-privsep-lang-2.4.0-2.el8.noarch.rpm                                                  248 kB/s |  12 kB     00:00    
(131/334): python-oslo-utils-lang-4.6.0-2.el8.noarch.rpm                                                    231 kB/s |  12 kB     00:00    
(132/334): python-oslo-versionedobjects-lang-2.3.0-2.el8.noarch.rpm                                         266 kB/s |  12 kB     00:00    
(133/334): python3-Bottleneck-1.2.1-13.el8.x86_64.rpm                                                       1.8 MB/s | 131 kB     00:00    
(134/334): python3-alembic-1.4.2-5.el8.noarch.rpm                                                           5.8 MB/s | 811 kB     00:00    
(135/334): python3-amqp-2.6.1-1.el8.noarch.rpm                                                              1.2 MB/s |  96 kB     00:00    
(136/334): python3-automaton-2.2.0-1.el8.noarch.rpm                                                         841 kB/s |  41 kB     00:00    
(137/334): python3-cachetools-4.1.1-2.el8.noarch.rpm                                                        734 kB/s |  34 kB     00:00    
(138/334): python3-cinder-common-17.2.0-1.el8.noarch.rpm                                                    6.1 MB/s | 3.9 MB     00:00    
(139/334): python3-cinderlib-3.0.0-1.el8.noarch.rpm                                                         1.4 MB/s |  78 kB     00:00    
(140/334): python3-cycler-0.10.0-13.el8.noarch.rpm                                                          319 kB/s |  20 kB     00:00    
(141/334): python3-editor-1.0.4-4.el8.noarch.rpm                                                            404 kB/s |  19 kB     00:00    
(142/334): python3-eventlet-0.25.2-3.1.el8.noarch.rpm                                                       4.1 MB/s | 385 kB     00:00    
(143/334): python3-fasteners-0.14.1-20.el8.noarch.rpm                                                       739 kB/s |  44 kB     00:00    
(144/334): python3-futurist-2.3.0-2.el8.noarch.rpm                                                          1.0 MB/s |  63 kB     00:00    
(145/334): python3-importlib-metadata-1.7.0-1.el8.noarch.rpm                                                816 kB/s |  46 kB     00:00    
(146/334): python3-iso8601-0.1.12-3.el8.noarch.rpm                                                          491 kB/s |  25 kB     00:00    
(147/334): python3-kazoo-2.8.0-1.el8.noarch.rpm                                                             2.1 MB/s | 164 kB     00:00    
(148/334): python3-kiwisolver-1.1.0-4.el8.x86_64.rpm                                                        1.1 MB/s |  78 kB     00:00    
(149/334): python3-kombu-4.6.11-2.el8.noarch.rpm                                                            3.3 MB/s | 341 kB     00:00    
(150/334): python3-matplotlib-3.1.1-2.el8.x86_64.rpm                                                        6.5 MB/s | 3.3 MB     00:00    
(151/334): python3-matplotlib-data-3.1.1-2.el8.noarch.rpm                                                   5.4 MB/s | 1.8 MB     00:00    
(152/334): python3-matplotlib-data-fonts-3.1.1-2.el8.noarch.rpm                                             6.8 MB/s | 2.4 MB     00:00    
(153/334): python3-matplotlib-tk-3.1.1-2.el8.x86_64.rpm                                                     553 kB/s |  64 kB     00:00    
(154/334): python3-migrate-0.13.0-1.el8.noarch.rpm                                                          3.6 MB/s | 238 kB     00:00    
(155/334): python3-msgpack-1.0.0-2.el8.x86_64.rpm                                                           1.3 MB/s |  93 kB     00:00    
(156/334): python3-networkx-2.5-1.el8.noarch.rpm                                                            6.0 MB/s | 2.5 MB     00:00    
(157/334): python3-numexpr-2.7.1-1.el8.x86_64.rpm                                                           2.6 MB/s | 198 kB     00:00    
(158/334): python3-os-brick-4.0.4-1.el8.noarch.rpm                                                          6.3 MB/s | 1.1 MB     00:00    
(159/334): python3-os-win-5.2.0-1.el8.noarch.rpm                                                            4.4 MB/s | 449 kB     00:00    
(160/334): python3-oslo-concurrency-4.3.1-1.el8.noarch.rpm                                                  634 kB/s |  41 kB     00:00    
(161/334): python3-oslo-config-8.3.4-1.el8.noarch.rpm                                                       3.1 MB/s | 224 kB     00:00    
(162/334): python3-oslo-context-3.1.2-1.el8.noarch.rpm                                                      444 kB/s |  25 kB     00:00    
(163/334): python3-oslo-db-8.4.1-1.el8.noarch.rpm                                                           2.4 MB/s | 147 kB     00:00    
(164/334): python3-oslo-i18n-5.0.1-2.el8.noarch.rpm                                                         996 kB/s |  58 kB     00:00    
(165/334): python3-oslo-log-4.4.0-2.el8.noarch.rpm                                                          1.1 MB/s |  63 kB     00:00    
(166/334): python3-oslo-messaging-12.5.2-1.el8.noarch.rpm                                                   3.5 MB/s | 226 kB     00:00    
(167/334): python3-oslo-middleware-4.1.1-2.el8.noarch.rpm                                                   988 kB/s |  53 kB     00:00    
(168/334): python3-oslo-privsep-2.4.0-2.el8.noarch.rpm                                                      766 kB/s |  40 kB     00:00    
(169/334): python3-oslo-rootwrap-6.2.0-2.el8.noarch.rpm                                                     810 kB/s |  44 kB     00:00    
(170/334): python3-oslo-serialization-4.0.1-2.el8.noarch.rpm                                                656 kB/s |  32 kB     00:00    
(171/334): python3-oslo-service-2.4.0-2.el8.noarch.rpm                                                      1.3 MB/s |  70 kB     00:00    
(172/334): python3-oslo-utils-4.6.0-2.el8.noarch.rpm                                                        1.6 MB/s |  79 kB     00:00    
(173/334): python3-oslo-versionedobjects-2.3.0-2.el8.noarch.rpm                                             1.5 MB/s |  78 kB     00:00    
(174/334): python3-packaging-20.4-1.el8.noarch.rpm                                                          1.1 MB/s |  67 kB     00:00    
(175/334): python3-pandas-0.25.3-1.el8.x86_64.rpm                                                           4.6 MB/s |  11 MB     00:02    
(176/334): python3-paste-3.2.4-1.el8.noarch.rpm                                                             5.2 MB/s | 811 kB     00:00    
(177/334): python3-paste-deploy-2.1.0-3.el8.noarch.rpm                                                      759 kB/s |  40 kB     00:00    
(178/334): python3-pydot-1.4.1-1.el8.noarch.rpm                                                             800 kB/s |  51 kB     00:00    
(179/334): python3-pygraphviz-1.5-9.el8.x86_64.rpm                                                          1.4 MB/s |  91 kB     00:00    
(180/334): python3-pyngus-2.3.0-4.el8.noarch.rpm                                                            1.2 MB/s |  54 kB     00:00    
(181/334): python3-redis-3.3.8-1.el8.noarch.rpm                                                             2.1 MB/s | 131 kB     00:00    
(182/334): python3-repoze-lru-0.7-6.el8.noarch.rpm                                                          725 kB/s |  33 kB     00:00    
(183/334): python3-rfc3986-1.4.0-3.el8.noarch.rpm                                                           868 kB/s |  52 kB     00:00    
(184/334): python3-routes-2.4.1-12.el8.noarch.rpm                                                           802 kB/s | 196 kB     00:00    
(185/334): python3-sqlparse-0.3.1-3.el8.noarch.rpm                                                          1.3 MB/s |  88 kB     00:00    
(186/334): python3-statsd-3.2.1-16.el8.noarch.rpm                                                           571 kB/s |  35 kB     00:00    
(187/334): python3-stevedore-3.2.2-2.el8.noarch.rpm                                                         596 kB/s |  67 kB     00:00    
(188/334): python3-tables-3.5.2-6.el8.x86_64.rpm                                                            2.6 MB/s | 1.4 MB     00:00    
(189/334): python3-tabulate-0.8.7-4.el8.noarch.rpm                                                          905 kB/s |  50 kB     00:00    
(190/334): python3-taskflow-4.5.0-2.el8.noarch.rpm                                                          5.3 MB/s | 689 kB     00:00    
(191/334): python3-tempita-0.5.1-25.el8.noarch.rpm                                                          546 kB/s |  39 kB     00:00    
(192/334): python3-tenacity-6.2.0-1.el8.noarch.rpm                                                          878 kB/s |  50 kB     00:00    
(193/334): python3-tooz-2.7.2-1.el8.noarch.rpm                                                              1.7 MB/s | 109 kB     00:00    
(194/334): python3-vine-1.3.0-4.el8.noarch.rpm                                                              698 kB/s |  34 kB     00:00    
(195/334): python3-webob-1.8.6-3.el8.noarch.rpm                                                             3.3 MB/s | 253 kB     00:00    
(196/334): python3-yappi-1.2.5-1.el8.x86_64.rpm                                                             759 kB/s |  50 kB     00:00    
(197/334): python3-zake-0.2.2-18.el8.noarch.rpm                                                             1.0 MB/s |  46 kB     00:00    
(198/334): python3-zstd-1.4.5.1-1.el8.x86_64.rpm                                                            307 kB/s |  18 kB     00:00    
(199/334): python3-prettytable-0.7.2-14.el8.noarch.rpm                                                       70 kB/s |  44 kB     00:00    
(200/334): python3-netaddr-0.7.19-8.el8.noarch.rpm                                                          2.6 MB/s | 1.5 MB     00:00    
(201/334): xorg-x11-fonts-ISO8859-1-100dpi-7.5-19.el8.noarch.rpm                                            2.2 MB/s | 1.1 MB     00:00    
(202/334): ovirt-engine-wildfly-23.0.2-1.el8.x86_64.rpm                                                     2.5 MB/s | 196 MB     01:16    
(203/334): glassfish-jaxb-api-2.2.12-8.module+el8+2468+c564cec5.noarch.rpm                                  113 kB/s | 102 kB     00:00    
[MIRROR] python3-pyasn1-0.4.6-3.el8.noarch.rpm: Curl error (28): Timeout was reached for https://vault.centos.org/8.5.2111/opstools/x86_64/collectd-5/Packages/p/python3-pyasn1-0.4.6-3.el8.noarch.rpm [Connection timed out after 30000 milliseconds]
(204/334): glassfish-jaxb-core-2.2.11-11.module+el8+2468+c564cec5.noarch.rpm                                371 kB/s | 158 kB     00:00    
(205/334): perl-Text-Unidecode-1.30-5.el8.noarch.rpm                                                        268 kB/s | 149 kB     00:00    
(206/334): xmlstreambuffer-1.5.4-8.module+el8+2468+c564cec5.noarch.rpm                                      231 kB/s |  87 kB     00:00    
(207/334): python3-itsdangerous-0.24-14.el8.noarch.rpm                                                       87 kB/s |  31 kB     00:00    
(208/334): python3-mako-1.0.6-13.el8.noarch.rpm                                                             402 kB/s | 157 kB     00:00    
(209/334): python3-jmespath-0.9.0-11.el8.noarch.rpm                                                         141 kB/s |  45 kB     00:00    
(210/334): bea-stax-api-1.2.0-16.module+el8+2468+c564cec5.noarch.rpm                                        105 kB/s |  37 kB     00:00    
(211/334): stax-ex-1.7.7-8.module+el8+2468+c564cec5.noarch.rpm                                              168 kB/s |  56 kB     00:00    
(212/334): perl-XML-XPath-1.42-3.el8.noarch.rpm                                                             271 kB/s |  88 kB     00:00    
(213/334): glassfish-fastinfoset-1.2.13-9.module+el8+2468+c564cec5.noarch.rpm                               768 kB/s | 354 kB     00:00    
(214/334): jdeparser-2.0.0-5.el8.noarch.rpm                                                                 462 kB/s | 217 kB     00:00    
(215/334): jboss-logging-tools-2.0.1-6.el8.noarch.rpm                                                       453 kB/s | 174 kB     00:00    
(216/334): glassfish-jaxb-txw2-2.2.11-11.module+el8+2468+c564cec5.noarch.rpm                                266 kB/s |  90 kB     00:00    
(217/334): python3-click-6.7-8.el8.noarch.rpm                                                               346 kB/s | 131 kB     00:00    
(218/334): python3-jsonschema-2.6.0-4.el8.noarch.rpm                                                        254 kB/s |  82 kB     00:00    
(219/334): xsom-0-19.20110809svn.module+el8+2468+c564cec5.noarch.rpm                                        983 kB/s | 399 kB     00:00    
(220/334): jboss-logging-3.3.0-5.el8.noarch.rpm                                                             154 kB/s |  71 kB     00:00    
(221/334): ongres-scram-client-1.0.0~beta.2-5.el8.noarch.rpm                                                 40 kB/s |  24 kB     00:00    
(222/334): jackson-module-jaxb-annotations-2.7.6-4.module+el8+2468+c564cec5.noarch.rpm                      145 kB/s |  46 kB     00:00    
(223/334): relaxngDatatype-2011.1-7.module+el8+2468+c564cec5.noarch.rpm                                      67 kB/s |  28 kB     00:00    
(224/334): python3-werkzeug-0.12.2-4.el8.noarch.rpm                                                         1.2 MB/s | 457 kB     00:00    
(225/334): jboss-jaxrs-2.0-api-1.0.0-6.el8.noarch.rpm                                                       260 kB/s | 113 kB     00:00    
(226/334): glassfish-jaxb-runtime-2.2.11-11.module+el8+2468+c564cec5.noarch.rpm                             2.2 MB/s | 936 kB     00:00    
(227/334): jboss-annotations-1.2-api-1.0.0-4.el8.noarch.rpm                                                  90 kB/s |  41 kB     00:00    
(228/334): istack-commons-runtime-2.21-9.el8+7.noarch.rpm                                                   122 kB/s |  44 kB     00:00    
(229/334): apr-util-openssl-1.6.1-6.el8.x86_64.rpm                                                           81 kB/s |  27 kB     00:00    
(230/334): ongres-scram-1.0.0~beta.2-5.el8.noarch.rpm                                                        70 kB/s |  46 kB     00:00    
(231/334): apr-util-bdb-1.6.1-6.el8.x86_64.rpm                                                               66 kB/s |  25 kB     00:00    
(232/334): python3-psycopg2-2.7.5-7.el8.x86_64.rpm                                                          338 kB/s | 172 kB     00:00    
(233/334): libXaw-1.0.13-10.el8.x86_64.rpm                                                                  530 kB/s | 194 kB     00:00    
(234/334): perl-encoding-2.22-3.el8.x86_64.rpm                                                               98 kB/s |  68 kB     00:00    
(235/334): tk-8.6.8-1.el8.x86_64.rpm                                                                        3.2 MB/s | 1.6 MB     00:00    
(236/334): perl-XML-Parser-2.44-11.el8.x86_64.rpm                                                           649 kB/s | 226 kB     00:00    
(237/334): python3-netifaces-0.10.6-4.el8.x86_64.rpm                                                         53 kB/s |  25 kB     00:00    
(238/334): apr-util-1.6.1-6.el8.x86_64.rpm                                                                  272 kB/s | 105 kB     00:00    
(239/334): perl-Filter-1.58-2.el8.x86_64.rpm                                                                251 kB/s |  82 kB     00:00    
(240/334): python3-markupsafe-0.23-19.el8.x86_64.rpm                                                        114 kB/s |  39 kB     00:00    
(241/334): sysfsutils-2.1.0-24.el8.x86_64.rpm                                                               152 kB/s |  49 kB     00:00    
(242/334): librdkafka-0.11.4-1.el8.x86_64.rpm                                                               952 kB/s | 353 kB     00:00    
(243/334): jackson-jaxrs-providers-2.9.9-1.module+el8.1.0+3832+9784644d.noarch.rpm                          135 kB/s |  45 kB     00:00    
(244/334): python3-docutils-0.14-12.module+el8.1.0+3334+5cb623d7.noarch.rpm                                 3.4 MB/s | 1.6 MB     00:00    
(245/334): python3-distro-1.4.0-2.module+el8.1.0+3334+5cb623d7.noarch.rpm                                    39 kB/s |  37 kB     00:00    
(246/334): jackson-jaxrs-json-provider-2.9.9-1.module+el8.1.0+3832+9784644d.noarch.rpm                       56 kB/s |  24 kB     00:00    
(247/334): jackson-annotations-2.10.0-1.module+el8.2.0+5059+3eb3af25.noarch.rpm                             152 kB/s |  71 kB     00:00    
(248/334): jackson-databind-2.10.0-1.module+el8.2.0+5059+3eb3af25.noarch.rpm                                2.8 MB/s | 1.3 MB     00:00    
(249/334): jackson-core-2.10.0-1.module+el8.2.0+5059+3eb3af25.noarch.rpm                                    669 kB/s | 345 kB     00:00    
(250/334): python3-greenlet-0.4.13-4.el8.x86_64.rpm                                                          93 kB/s |  31 kB     00:00    
(251/334): python3-mod_wsgi-4.6.4-4.el8.x86_64.rpm                                                          4.3 MB/s | 2.5 MB     00:00    
(252/334): postgresql-jdbc-42.2.3-3.el8_2.noarch.rpm                                                        948 kB/s | 710 kB     00:00    
(253/334): python3-flask-0.12.2-4.el8.noarch.rpm                                                            270 kB/s | 141 kB     00:00    
(254/334): python3-sqlalchemy-1.3.2-2.module+el8.3.0+6646+6b4b10ec.x86_64.rpm                               3.6 MB/s | 1.9 MB     00:00    
(255/334): git-core-2.27.0-1.el8.x86_64.rpm                                                                 4.1 MB/s | 5.7 MB     00:01    
(256/334): nodejs-full-i18n-10.24.0-1.module+el8.3.0+10166+b07ac28e.x86_64.rpm                              4.7 MB/s | 7.3 MB     00:01    
(257/334): npm-6.14.11-1.10.24.0.1.module+el8.3.0+10166+b07ac28e.x86_64.rpm                                 3.5 MB/s | 3.7 MB     00:01    
(258/334): nodejs-10.24.0-1.module+el8.3.0+10166+b07ac28e.x86_64.rpm                                        5.8 MB/s | 8.8 MB     00:01    
(259/334): python3-PyMySQL-0.10.1-2.module+el8.4.0+9657+a4b6a102.noarch.rpm                                 261 kB/s |  98 kB     00:00    
(260/334): mod_http2-1.15.7-3.module+el8.4.0+8625+d397f3da.x86_64.rpm                                       460 kB/s | 154 kB     00:00    
(261/334): resteasy-3.0.26-6.module+el8.4.0+8891+bb8828ef.noarch.rpm                                        2.1 MB/s | 1.1 MB     00:00    
(262/334): openblas-0.3.12-1.el8.x86_64.rpm                                                                 4.9 MB/s | 4.6 MB     00:00    
(263/334): python-srpm-macros-3-41.el8.noarch.rpm                                                            39 kB/s |  15 kB     00:00    
(264/334): openblas-threads-0.3.12-1.el8.x86_64.rpm                                                         5.5 MB/s | 4.7 MB     00:00    
(265/334): uuid-1.6.2-43.el8.x86_64.rpm                                                                     185 kB/s |  64 kB     00:00    
(266/334): python-rpm-macros-3-41.el8.noarch.rpm                                                             48 kB/s |  15 kB     00:00    
(267/334): python3-pyOpenSSL-19.0.0-1.el8.noarch.rpm                                                        318 kB/s | 103 kB     00:00    
(268/334): python3-rpm-macros-3-41.el8.noarch.rpm                                                            20 kB/s |  14 kB     00:00    
(269/334): libpq-13.3-1.el8_4.x86_64.rpm                                                                    567 kB/s | 197 kB     00:00    
(270/334): pki-servlet-4.0-api-9.0.30-3.module+el8.5.0+11388+9e95fe00.noarch.rpm                            440 kB/s | 282 kB     00:00    
(271/334): liblognorm-2.0.5-2.el8.x86_64.rpm                                                                225 kB/s |  88 kB     00:00    
(272/334): python3-scipy-1.0.0-21.module+el8.5.0+10916+41bd434d.x86_64.rpm                                  8.2 MB/s |  14 MB     00:01    
(273/334): rsyslog-mmnormalize-8.2102.0-5.el8.x86_64.rpm                                                     57 kB/s |  23 kB     00:00    
(274/334): platform-python-devel-3.6.8-41.el8.x86_64.rpm                                                    682 kB/s | 249 kB     00:00    
(275/334): texlive-base-20180414-23.el8.noarch.rpm                                                          4.1 MB/s | 2.4 MB     00:00    
(276/334): texlive-tetex-20180414-23.el8.noarch.rpm                                                         1.1 MB/s | 402 kB     00:00    
(277/334): python3-tkinter-3.6.8-41.el8.x86_64.rpm                                                          1.0 MB/s | 372 kB     00:00    
(278/334): graphviz-2.40.1-43.el8.x86_64.rpm                                                                2.7 MB/s | 1.7 MB     00:00    
(279/334): python3-numpy-1.14.3-10.el8.x86_64.rpm                                                           3.5 MB/s | 3.7 MB     00:01    
(280/334): python3-rpm-generators-5-7.el8.noarch.rpm                                                         78 kB/s |  25 kB     00:00    
(281/334): texlive-dvipng-20180414-23.el8.x86_64.rpm                                                        914 kB/s | 332 kB     00:00    
(282/334): perl-open-1.11-420.el8.noarch.rpm                                                                214 kB/s |  77 kB     00:00    
(283/334): python3-babel-2.5.1-7.el8.noarch.rpm                                                             6.1 MB/s | 4.8 MB     00:00    
(284/334): python3-jinja2-2.10.1-3.el8.noarch.rpm                                                           1.2 MB/s | 538 kB     00:00    
(285/334): rsyslog-elasticsearch-8.2102.0-5.el8.x86_64.rpm                                                   99 kB/s |  33 kB     00:00    
(286/334): apr-1.6.3-12.el8.x86_64.rpm                                                                      356 kB/s | 130 kB     00:00    
(287/334): python3-pillow-5.1.1-16.el8.x86_64.rpm                                                           1.6 MB/s | 632 kB     00:00    
(288/334): python3-numpy-f2py-1.14.3-10.el8.x86_64.rpm                                                      660 kB/s | 225 kB     00:00    
(289/334): rhel-system-roles-1.7.3-2.el8.noarch.rpm                                                         2.8 MB/s | 1.3 MB     00:00    
(290/334): texlive-texlive.infra-20180414-23.el8.noarch.rpm                                                 490 kB/s | 280 kB     00:00    
(291/334): texlive-lib-20180414-23.el8.x86_64.rpm                                                           1.3 MB/s | 541 kB     00:00    
(292/334): rsyslog-mmjsonparse-8.2102.0-5.el8.x86_64.rpm                                                     65 kB/s |  22 kB     00:00    
(293/334): texlive-kpathsea-20180414-23.el8.x86_64.rpm                                                      2.5 MB/s | 1.1 MB     00:00    
(294/334): postgresql-contrib-12.9-1.module+el8.5.0+13373+4554acc4.x86_64.rpm                               1.7 MB/s | 868 kB     00:00    
(295/334): grafana-pcp-3.1.0-1.el8.x86_64.rpm                                                               6.5 MB/s | 9.4 MB     00:01    
(296/334): postgresql-server-12.9-1.module+el8.5.0+13373+4554acc4.x86_64.rpm                                4.6 MB/s | 5.6 MB     00:01    
(297/334): postgresql-12.9-1.module+el8.5.0+13373+4554acc4.x86_64.rpm                                       2.7 MB/s | 1.5 MB     00:00    
[MIRROR] python3-pyasn1-0.4.6-3.el8.noarch.rpm: Curl error (28): Timeout was reached for https://vault.centos.org/8.5.2111/opstools/x86_64/collectd-5/Packages/p/python3-pyasn1-0.4.6-3.el8.noarch.rpm [Operation timed out after 30000 milliseconds with 0 out of 0 bytes received]
(298/334): java-11-openjdk-headless-11.0.14.0.9-2.el8_5.x86_64.rpm                                          5.6 MB/s |  40 MB     00:07    
(299/334): httpd-tools-2.4.37-43.module+el8.5.0+13806+b30d9eec.1.x86_64.rpm                                 174 kB/s | 107 kB     00:00    
(300/334): grafana-7.5.9-5.el8_5.x86_64.rpm                                                                 5.0 MB/s |  41 MB     00:08    
(301/334): httpd-filesystem-2.4.37-43.module+el8.5.0+13806+b30d9eec.1.noarch.rpm                            107 kB/s |  40 kB     00:00    
(302/334): mod_ssl-2.4.37-43.module+el8.5.0+13806+b30d9eec.1.x86_64.rpm                                     385 kB/s | 136 kB     00:00    
(303/334): httpd-2.4.37-43.module+el8.5.0+13806+b30d9eec.1.x86_64.rpm                                       2.9 MB/s | 1.4 MB     00:00    
(304/334): libaec-1.0.2-3.el8.x86_64.rpm                                                                    117 kB/s |  39 kB     00:00    
(305/334): libqhull-2015.2-5.el8.x86_64.rpm                                                                  74 kB/s | 169 kB     00:02    
(306/334): apache-commons-compress-1.18-1.module+el8+2598+06babf2e.noarch.rpm                               166 kB/s | 526 kB     00:03    
(307/334): python3-mock-2.0.0-11.el8.noarch.rpm                                                              30 kB/s |  59 kB     00:01    
(308/334): aopalliance-1.0-17.module+el8+2598+06babf2e.noarch.rpm                                           8.2 kB/s |  17 kB     00:02    
(309/334): apache-commons-jxpath-1.3-29.module+el8+2598+06babf2e.noarch.rpm                                 151 kB/s | 295 kB     00:01    
(310/334): apache-commons-lang-2.6-21.module+el8+2598+06babf2e.noarch.rpm                                   141 kB/s | 282 kB     00:02    
(311/334): apache-commons-codec-1.11-3.module+el8+2598+06babf2e.noarch.rpm                                  119 kB/s | 289 kB     00:02    
(312/334): httpcomponents-client-4.5.5-4.module+el8+2598+06babf2e.noarch.rpm                                345 kB/s | 718 kB     00:02    
(313/334): httpcomponents-core-4.4.10-3.module+el8+2598+06babf2e.noarch.rpm                                 305 kB/s | 638 kB     00:02    
(314/334): slf4j-1.7.25-4.module+el8+2598+06babf2e.noarch.rpm                                                30 kB/s |  77 kB     00:02    
(315/334): slf4j-jdk14-1.7.25-4.module+el8+2598+06babf2e.noarch.rpm                                          14 kB/s |  25 kB     00:01    
(316/334): jcl-over-slf4j-1.7.25-4.module+el8+2598+06babf2e.noarch.rpm                                       16 kB/s |  32 kB     00:02    
(317/334): apache-commons-collections-3.2.2-10.module+el8+2598+06babf2e.noarch.rpm                          202 kB/s | 537 kB     00:02    
(318/334): apache-commons-io-2.6-3.module+el8+2598+06babf2e.noarch.rpm                                      115 kB/s | 224 kB     00:01    
(319/334): python3-pyasn1-0.4.6-3.el8.noarch.rpm                                                            1.8 kB/s | 140 kB     01:19    
(320/334): apache-commons-logging-1.2-13.module+el8+2598+06babf2e.noarch.rpm                                 45 kB/s |  85 kB     00:01    
(321/334): tcl-8.6.8-2.el8.x86_64.rpm                                                                       2.1 MB/s | 1.1 MB     00:00    
(322/334): fuse3-libs-3.2.1-12.el8.x86_64.rpm                                                               273 kB/s |  94 kB     00:00    
(323/334): python3-cffi-1.11.5-5.el8.x86_64.rpm                                                             662 kB/s | 238 kB     00:00    
(324/334): python3-pycparser-2.14-14.el8.noarch.rpm                                                         319 kB/s | 109 kB     00:00    
(325/334): libcgroup-tools-0.41-19.el8.x86_64.rpm                                                           269 kB/s |  93 kB     00:00    
(326/334): publicsuffix-list-20180723-1.el8.noarch.rpm                                                      234 kB/s |  79 kB     00:00    
(327/334): python3-httplib2-0.10.3-4.el8.noarch.rpm                                                          55 kB/s | 108 kB     00:01    
(328/334): python3-dns-1.15.0-10.el8.noarch.rpm                                                             704 kB/s | 253 kB     00:00    
(329/334): librabbitmq-0.9.0-3.el8.x86_64.rpm                                                               139 kB/s |  47 kB     00:00    
(330/334): redhat-logos-httpd-84.5-1.el8.noarch.rpm                                                          89 kB/s |  29 kB     00:00    
(331/334): python3-cryptography-3.2.1-5.el8.x86_64.rpm                                                      1.4 MB/s | 559 kB     00:00    
(332/334): libquadmath-8.5.0-4.el8_5.x86_64.rpm                                                             471 kB/s | 170 kB     00:00    
(333/334): libgfortran-8.5.0-4.el8_5.x86_64.rpm                                                             1.6 MB/s | 643 kB     00:00    
(334/334): python3-dnf-plugin-versionlock-4.0.21-4.el8_5.noarch.rpm                                         183 kB/s |  62 kB     00:00    
--------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                       4.4 MB/s | 706 MB     02:42     
Latest oVirt 4.4 Release                                                                                    2.2 MB/s | 2.5 kB     00:00    
Importing GPG key 0xFE590CB7:
 Userid     : "oVirt <infra@ovirt.org>"
 Fingerprint: 31A5 D783 7FAD 7CB2 86CD 3469 AB8C 4F9D FE59 0CB7
 From       : /etc/pki/rpm-gpg/RPM-GPG-ovirt-4.4
Key imported successfully
Extra Packages for Enterprise Linux 8 - x86_64                                                              1.3 kB/s | 1.6 kB     00:01    
Importing GPG key 0x2F86D6A1:
 Userid     : "Fedora EPEL (8) <epel@fedoraproject.org>"
 Fingerprint: 94E2 79EB 8D8F 25B2 1810 ADF1 21EA 45AB 2F86 D6A1
 From       : https://download.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-8
Key imported successfully
CentOS-8 - NFV OpenvSwitch                                                                                  778  B/s | 1.2 kB     00:01    
Importing GPG key 0x9D2A76A7:
 Userid     : "CentOS NFV SIG (https://wiki.centos.org/SpecialInterestGroup/NFV) <security@centos.org>"
 Fingerprint: 3515 4228 1749 01BE FA8E 69A6 2146 5E28 9D2A 76A7
 From       : https://www.centos.org/keys/RPM-GPG-KEY-CentOS-SIG-NFV
Key imported successfully
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Running scriptlet: java-11-openjdk-headless-1:11.0.14.0.9-2.el8_5.x86_64                                                              1/1 
  Running scriptlet: npm-1:6.14.11-1.10.24.0.1.module+el8.3.0+10166+b07ac28e.x86_64                                                     1/1 
  Running scriptlet: ovirt-openvswitch-2.11-1.el8.noarch                                                                                1/1 
Failed to get unit file state for openvswitch.service: No such file or directory
Failed to get unit file state for ovn-northd.service: No such file or directory
Failed to get unit file state for ovirt-provider-ovn.service: No such file or directory
Failed to get unit file state for ovn-controller.service: No such file or directory

  Preparing        :                                                                                                                    1/1 
  Installing       : java-11-openjdk-headless-1:11.0.14.0.9-2.el8_5.x86_64                                                            1/334 
  Running scriptlet: java-11-openjdk-headless-1:11.0.14.0.9-2.el8_5.x86_64                                                            1/334 
  Installing       : python3-netaddr-0.7.19-8.el8.noarch                                                                              2/334 
  Installing       : slf4j-1.7.25-4.module+el8+2598+06babf2e.noarch                                                                   3/334 
  Installing       : jackson-core-2.10.0-1.module+el8.2.0+5059+3eb3af25.noarch                                                        4/334 
  Installing       : libpq-13.3-1.el8_4.x86_64                                                                                        5/334 
  Installing       : python3-jmespath-0.9.0-11.el8.noarch                                                                             6/334 
  Installing       : collectd-5.12.0-7.el8s.x86_64                                                                                    7/334 
  Running scriptlet: collectd-5.12.0-7.el8s.x86_64                                                                                    7/334 
  Installing       : python3-sqlalchemy-1.3.2-2.module+el8.3.0+6646+6b4b10ec.x86_64                                                   8/334 
  Installing       : python3-greenlet-0.4.13-4.el8.x86_64                                                                             9/334 
  Installing       : jackson-annotations-2.10.0-1.module+el8.2.0+5059+3eb3af25.noarch                                                10/334 
  Installing       : jackson-databind-2.10.0-1.module+el8.2.0+5059+3eb3af25.noarch                                                   11/334 
  Installing       : openstack-java-client-3.2.9-9.el8.noarch                                                                        12/334 
  Installing       : python3-webob-1.8.6-3.el8.noarch                                                                                13/334 
  Installing       : python3-tenacity-6.2.0-1.el8.noarch                                                                             14/334 
  Installing       : python3-rados-2:16.2.7-1.el8s.x86_64                                                                            15/334 
  Installing       : postgresql-12.9-1.module+el8.5.0+13373+4554acc4.x86_64                                                          16/334 
  Running scriptlet: postgresql-server-12.9-1.module+el8.5.0+13373+4554acc4.x86_64                                                   17/334 
  Installing       : postgresql-server-12.9-1.module+el8.5.0+13373+4554acc4.x86_64                                                   17/334 
  Running scriptlet: postgresql-server-12.9-1.module+el8.5.0+13373+4554acc4.x86_64                                                   17/334 
  Installing       : python3-ovirt-setup-lib-1.3.2-1.el8.noarch                                                                      18/334 
  Installing       : apache-commons-lang-2.6-21.module+el8+2598+06babf2e.noarch                                                      19/334 
  Installing       : apr-1.6.3-12.el8.x86_64                                                                                         20/334 
  Running scriptlet: apr-1.6.3-12.el8.x86_64                                                                                         20/334 
  Installing       : apr-util-bdb-1.6.1-6.el8.x86_64                                                                                 21/334 
  Installing       : apr-util-1.6.1-6.el8.x86_64                                                                                     22/334 
  Running scriptlet: apr-util-1.6.1-6.el8.x86_64                                                                                     22/334 
  Installing       : apr-util-openssl-1.6.1-6.el8.x86_64                                                                             23/334 
  Running scriptlet: texlive-base-7:20180414-23.el8.noarch                                                                           24/334 
  Installing       : texlive-base-7:20180414-23.el8.noarch                                                                           24/334 
  Installing       : python3-msgpack-1.0.0-2.el8.x86_64                                                                              25/334 
  Installing       : python3-iso8601-0.1.12-3.el8.noarch                                                                             26/334 
  Installing       : python3-futurist-2.3.0-2.el8.noarch                                                                             27/334 
  Installing       : python3-fasteners-0.14.1-20.el8.noarch                                                                          28/334 
  Installing       : ovn2.11-2.11.1-57.el8s.x86_64                                                                                   29/334 
  Installing       : ovirt-openvswitch-ovn-2.11-1.el8.noarch                                                                         30/334 
  Installing       : openstack-java-cinder-model-3.2.9-9.el8.noarch                                                                  31/334 
  Installing       : openstack-java-glance-model-3.2.9-9.el8.noarch                                                                  32/334 
  Installing       : openstack-java-keystone-model-3.2.9-9.el8.noarch                                                                33/334 
  Installing       : openstack-java-quantum-model-3.2.9-9.el8.noarch                                                                 34/334 
  Installing       : python3-psycopg2-2.7.5-7.el8.x86_64                                                                             35/334 
  Installing       : slf4j-jdk14-1.7.25-4.module+el8+2598+06babf2e.noarch                                                            36/334 
  Installing       : ovirt-engine-extensions-api-1.0.1-1.el8.noarch                                                                  37/334 
  Installing       : python3-matplotlib-data-fonts-3.1.1-2.el8.noarch                                                                38/334 
  Installing       : python3-matplotlib-data-3.1.1-2.el8.noarch                                                                      39/334 
  Installing       : libquadmath-8.5.0-4.el8_5.x86_64                                                                                40/334 
  Running scriptlet: libquadmath-8.5.0-4.el8_5.x86_64                                                                                40/334 
  Installing       : libgfortran-8.5.0-4.el8_5.x86_64                                                                                41/334 
  Running scriptlet: libgfortran-8.5.0-4.el8_5.x86_64                                                                                41/334 
  Installing       : openblas-0.3.12-1.el8.x86_64                                                                                    42/334 
  Running scriptlet: openblas-0.3.12-1.el8.x86_64                                                                                    42/334 
  Installing       : openblas-threads-0.3.12-1.el8.x86_64                                                                            43/334 
  Running scriptlet: openblas-threads-0.3.12-1.el8.x86_64                                                                            43/334 
  Installing       : python3-numpy-1:1.14.3-10.el8.x86_64                                                                            44/334 
  Installing       : python3-numexpr-2.7.1-1.el8.x86_64                                                                              45/334 
  Installing       : librabbitmq-0.9.0-3.el8.x86_64                                                                                  46/334 
  Installing       : tcl-1:8.6.8-2.el8.x86_64                                                                                        47/334 
  Running scriptlet: tcl-1:8.6.8-2.el8.x86_64                                                                                        47/334 
  Installing       : apache-commons-logging-1.2-13.module+el8+2598+06babf2e.noarch                                                   48/334 
  Installing       : apache-commons-codec-1.11-3.module+el8+2598+06babf2e.noarch                                                     49/334 
  Installing       : ovirt-engine-extension-aaa-jdbc-1.2.0-1.el8.noarch                                                              50/334 
  Running scriptlet: httpd-filesystem-2.4.37-43.module+el8.5.0+13806+b30d9eec.1.noarch                                               51/334 
  Installing       : httpd-filesystem-2.4.37-43.module+el8.5.0+13806+b30d9eec.1.noarch                                               51/334 
  Installing       : texlive-lib-7:20180414-23.el8.x86_64                                                                            52/334 
  Installing       : python3-pillow-5.1.1-16.el8.x86_64                                                                              53/334 
  Installing       : python3-babel-2.5.1-7.el8.noarch                                                                                54/334 
  Installing       : liblognorm-2.0.5-2.el8.x86_64                                                                                   55/334 
  Running scriptlet: liblognorm-2.0.5-2.el8.x86_64                                                                                   55/334 
  Installing       : librdkafka-0.11.4-1.el8.x86_64                                                                                  56/334 
  Running scriptlet: librdkafka-0.11.4-1.el8.x86_64                                                                                  56/334 
  Installing       : sysfsutils-2.1.0-24.el8.x86_64                                                                                  57/334 
  Installing       : python3-markupsafe-0.23-19.el8.x86_64                                                                           58/334 
  Installing       : python3-jinja2-2.10.1-3.el8.noarch                                                                              59/334 
  Installing       : perl-XML-Parser-2.44-11.el8.x86_64                                                                              60/334 
  Installing       : python3-prettytable-0.7.2-14.el8.noarch                                                                         61/334 
  Installing       : python3-vine-1.3.0-4.el8.noarch                                                                                 62/334 
  Installing       : python3-amqp-2.6.1-1.el8.noarch                                                                                 63/334 
  Installing       : python3-tempita-0.5.1-25.el8.noarch                                                                             64/334 
  Installing       : python3-cachetools-4.1.1-2.el8.noarch                                                                           65/334 
  Installing       : ws-commons-util-1.0.2-1.el8.noarch                                                                              66/334 
  Installing       : python3-websockify-0.8.0-15.el8.noarch                                                                          67/334 
  Installing       : python3-lockfile-1:0.11.0-16.el8.noarch                                                                         68/334 
  Installing       : python3-funcsigs-1.0.2-17.el8.noarch                                                                            69/334 
  Running scriptlet: ovirt-vmconsole-1.0.9-1.el8.noarch                                                                              70/334 
  Installing       : ovirt-vmconsole-1.0.9-1.el8.noarch                                                                              70/334 
  Running scriptlet: ovirt-vmconsole-1.0.9-1.el8.noarch                                                                              70/334 
  Installing       : ovirt-engine-wildfly-23.0.2-1.el8.x86_64                                                                        71/334 
  Installing       : python3-ceph-argparse-2:16.2.7-1.el8s.x86_64                                                                    72/334 
  Installing       : liboath-2.6.2-4.el8s.x86_64                                                                                     73/334 
  Installing       : libcephfs2-2:16.2.7-1.el8s.x86_64                                                                               74/334 
  Running scriptlet: libcephfs2-2:16.2.7-1.el8s.x86_64                                                                               74/334 
  Installing       : python3-cephfs-2:16.2.7-1.el8s.x86_64                                                                           75/334 
  Installing       : librgw2-2:16.2.7-1.el8s.x86_64                                                                                  76/334 
  Running scriptlet: librgw2-2:16.2.7-1.el8s.x86_64                                                                                  76/334 
  Installing       : python3-rgw-2:16.2.7-1.el8s.x86_64                                                                              77/334 
  Installing       : ovirt-engine-wildfly-overlay-23.0.2-1.el8.noarch                                                                78/334 
  Installing       : ovirt-vmconsole-proxy-1.0.9-1.el8.noarch                                                                        79/334 
  Running scriptlet: ovirt-vmconsole-proxy-1.0.9-1.el8.noarch                                                                        79/334 
  Installing       : xmlrpc-common-3.1.3-1.el8.noarch                                                                                80/334 
  Installing       : xmlrpc-client-3.1.3-1.el8.noarch                                                                                81/334 
  Installing       : python3-mako-1.0.6-13.el8.noarch                                                                                82/334 
  Installing       : rsyslog-mmnormalize-8.2102.0-5.el8.x86_64                                                                       83/334 
  Installing       : apache-commons-configuration-1.10-1.el8.noarch                                                                  84/334 
  Running scriptlet: tk-1:8.6.8-1.el8.x86_64                                                                                         85/334 
  Installing       : tk-1:8.6.8-1.el8.x86_64                                                                                         85/334 
  Running scriptlet: tk-1:8.6.8-1.el8.x86_64                                                                                         85/334 
  Installing       : python3-tkinter-3.6.8-41.el8.x86_64                                                                             86/334 
  Installing       : openstack-java-quantum-client-3.2.9-9.el8.noarch                                                                87/334 
  Installing       : openstack-java-keystone-client-3.2.9-9.el8.noarch                                                               88/334 
  Installing       : openstack-java-glance-client-3.2.9-9.el8.noarch                                                                 89/334 
  Installing       : openstack-java-cinder-client-3.2.9-9.el8.noarch                                                                 90/334 
  Installing       : ovirt-openvswitch-ovn-common-2.11-1.el8.noarch                                                                  91/334 
  Running scriptlet: ovn2.11-central-2.11.1-57.el8s.x86_64                                                                           92/334 
  Installing       : ovn2.11-central-2.11.1-57.el8s.x86_64                                                                           92/334 
  Running scriptlet: ovn2.11-central-2.11.1-57.el8s.x86_64                                                                           92/334 
  Installing       : ovirt-openvswitch-ovn-central-2.11-1.el8.noarch                                                                 93/334 
  Installing       : httpd-tools-2.4.37-43.module+el8.5.0+13806+b30d9eec.1.x86_64                                                    94/334 
  Installing       : vdsm-jsonrpc-java-1.6.0-1.el8.noarch                                                                            95/334 
  Installing       : python3-rbd-2:16.2.7-1.el8s.x86_64                                                                              96/334 
  Installing       : jackson-module-jaxb-annotations-2.7.6-4.module+el8+2468+c564cec5.noarch                                         97/334 
  Installing       : jackson-jaxrs-providers-2.9.9-1.module+el8.1.0+3832+9784644d.noarch                                             98/334 
  Installing       : jackson-jaxrs-json-provider-2.9.9-1.module+el8.1.0+3832+9784644d.noarch                                         99/334 
  Installing       : collectd-disk-5.12.0-7.el8s.x86_64                                                                             100/334 
  Installing       : collectd-postgresql-5.12.0-7.el8s.x86_64                                                                       101/334 
  Installing       : collectd-write_http-5.12.0-7.el8s.x86_64                                                                       102/334 
  Installing       : collectd-write_syslog-5.12.0-7.el8s.x86_64                                                                     103/334 
  Installing       : rhel-system-roles-1.7.3-2.el8.noarch                                                                           104/334 
  Installing       : apache-sshd-2.6.0-2.el8.noarch                                                                                 105/334 
  Installing       : jcl-over-slf4j-1.7.25-4.module+el8+2598+06babf2e.noarch                                                        106/334 
  Installing       : snmp4j-3.6.4-0.1.el8.noarch                                                                                    107/334 
  Running scriptlet: grafana-7.5.9-5.el8_5.x86_64                                                                                   108/334 
  Installing       : grafana-7.5.9-5.el8_5.x86_64                                                                                   108/334 
  Running scriptlet: grafana-7.5.9-5.el8_5.x86_64                                                                                   108/334 
  Installing       : grafana-pcp-3.1.0-1.el8.x86_64                                                                                 109/334 
  Installing       : npm-1:6.14.11-1.10.24.0.1.module+el8.3.0+10166+b07ac28e.x86_64                                                 110/334 
  Installing       : nodejs-1:10.24.0-1.module+el8.3.0+10166+b07ac28e.x86_64                                                        111/334 
  Installing       : nodejs-full-i18n-1:10.24.0-1.module+el8.3.0+10166+b07ac28e.x86_64                                              112/334 
  Installing       : novnc-1.1.0-6.el8.noarch                                                                                       113/334 
  Installing       : python3-dnf-plugin-versionlock-4.0.21-4.el8_5.noarch                                                           114/334 
  Installing       : redhat-logos-httpd-84.5-1.el8.noarch                                                                           115/334 
  Installing       : mod_http2-1.15.7-3.module+el8.4.0+8625+d397f3da.x86_64                                                         116/334 
  Installing       : httpd-2.4.37-43.module+el8.5.0+13806+b30d9eec.1.x86_64                                                         117/334 
  Running scriptlet: httpd-2.4.37-43.module+el8.5.0+13806+b30d9eec.1.x86_64                                                         117/334 
  Installing       : mod_ssl-1:2.4.37-43.module+el8.5.0+13806+b30d9eec.1.x86_64                                                     118/334 
  Installing       : python3-mod_wsgi-4.6.4-4.el8.x86_64                                                                            119/334 
  Installing       : python3-dns-1.15.0-10.el8.noarch                                                                               120/334 
  Installing       : publicsuffix-list-20180723-1.el8.noarch                                                                        121/334 
  Installing       : libcgroup-tools-0.41-19.el8.x86_64                                                                             122/334 
  Running scriptlet: libcgroup-tools-0.41-19.el8.x86_64                                                                             122/334 
  Installing       : python3-pycparser-2.14-14.el8.noarch                                                                           123/334 
  Installing       : python3-cffi-1.11.5-5.el8.x86_64                                                                               124/334 
  Installing       : python3-cryptography-3.2.1-5.el8.x86_64                                                                        125/334 
  Installing       : python3-pyOpenSSL-19.0.0-1.el8.noarch                                                                          126/334 
  Installing       : python3-paste-3.2.4-1.el8.noarch                                                                               127/334 
  Installing       : python3-paste-deploy-2.1.0-3.el8.noarch                                                                        128/334 
  Installing       : python3-PyMySQL-0.10.1-2.module+el8.4.0+9657+a4b6a102.noarch                                                   129/334 
  Installing       : python3-bcrypt-3.1.7-3.el8.x86_64                                                                              130/334 
  Installing       : fuse3-libs-3.2.1-12.el8.x86_64                                                                                 131/334 
  Running scriptlet: fuse3-libs-3.2.1-12.el8.x86_64                                                                                 131/334 
  Installing       : python3-httplib2-0.10.3-4.el8.noarch                                                                           132/334 
  Installing       : apache-commons-io-1:2.6-3.module+el8+2598+06babf2e.noarch                                                      133/334 
  Installing       : apache-commons-collections-3.2.2-10.module+el8+2598+06babf2e.noarch                                            134/334 
  Installing       : httpcomponents-core-4.4.10-3.module+el8+2598+06babf2e.noarch                                                   135/334 
  Installing       : httpcomponents-client-4.5.5-4.module+el8+2598+06babf2e.noarch                                                  136/334 
  Installing       : apache-commons-jxpath-1.3-29.module+el8+2598+06babf2e.noarch                                                   137/334 
  Installing       : aopalliance-1.0-17.module+el8+2598+06babf2e.noarch                                                             138/334 
  Installing       : python3-mock-2.0.0-11.el8.noarch                                                                               139/334 
  Installing       : apache-commons-compress-1.18-1.module+el8+2598+06babf2e.noarch                                                 140/334 
  Installing       : libqhull-2015.2-5.el8.x86_64                                                                                   141/334 
  Running scriptlet: libqhull-2015.2-5.el8.x86_64                                                                                   141/334 
  Installing       : libaec-1.0.2-3.el8.x86_64                                                                                      142/334 
  Installing       : hdf5-1.10.5-5.el8.x86_64                                                                                       143/334 
  Installing       : rsyslog-mmjsonparse-8.2102.0-5.el8.x86_64                                                                      144/334 
  Installing       : rsyslog-elasticsearch-8.2102.0-5.el8.x86_64                                                                    145/334 
  Installing       : python3-rpm-generators-5-7.el8.noarch                                                                          146/334 
  Installing       : pki-servlet-4.0-api-1:9.0.30-3.module+el8.5.0+11388+9e95fe00.noarch                                            147/334 
  Installing       : uuid-1.6.2-43.el8.x86_64                                                                                       148/334 
  Running scriptlet: uuid-1.6.2-43.el8.x86_64                                                                                       148/334 
  Installing       : postgresql-contrib-12.9-1.module+el8.5.0+13373+4554acc4.x86_64                                                 149/334 
  Installing       : python-srpm-macros-3-41.el8.noarch                                                                             150/334 
  Installing       : python-rpm-macros-3-41.el8.noarch                                                                              151/334 
  Installing       : python3-rpm-macros-3-41.el8.noarch                                                                             152/334 
  Installing       : platform-python-devel-3.6.8-41.el8.x86_64                                                                      153/334 
  Installing       : python3-numpy-f2py-1:1.14.3-10.el8.x86_64                                                                      154/334 
  Running scriptlet: python3-numpy-f2py-1:1.14.3-10.el8.x86_64                                                                      154/334 
  Installing       : python3-scipy-1.0.0-21.module+el8.5.0+10916+41bd434d.x86_64                                                    155/334 
  Installing       : python3-Bottleneck-1.2.1-13.el8.x86_64                                                                         156/334 
  Installing       : git-core-2.27.0-1.el8.x86_64                                                                                   157/334 
  Installing       : python3-pbr-5.4.3-2.el8.noarch                                                                                 158/334 
  Installing       : python3-automaton-2.2.0-1.el8.noarch                                                                           159/334 
  Installing       : python3-docutils-0.14-12.module+el8.1.0+3334+5cb623d7.noarch                                                   160/334 
  Installing       : python3-daemon-2.2.3-7.el8.noarch                                                                              161/334 
  Installing       : python3-ovirt-engine-lib-4.4.10.6-1.el8.noarch                                                                 162/334 
  Installing       : python3-ansible-runner-1.4.6-1.el8.noarch                                                                      163/334 
  Installing       : python3-distro-1.4.0-2.module+el8.1.0+3334+5cb623d7.noarch                                                     164/334 
  Installing       : perl-Filter-2:1.58-2.el8.x86_64                                                                                165/334 
  Installing       : perl-encoding-4:2.22-3.el8.x86_64                                                                              166/334 
  Installing       : perl-open-1.11-420.el8.noarch                                                                                  167/334 
  Installing       : perl-XML-XPath-1.42-3.el8.noarch                                                                               168/334 
  Installing       : python3-netifaces-0.10.6-4.el8.x86_64                                                                          169/334 
  Installing       : libXaw-1.0.13-10.el8.x86_64                                                                                    170/334 
  Installing       : ongres-scram-1.0.0~beta.2-5.el8.noarch                                                                         171/334 
  Installing       : ongres-scram-client-1.0.0~beta.2-5.el8.noarch                                                                  172/334 
  Installing       : postgresql-jdbc-42.2.3-3.el8_2.noarch                                                                          173/334 
  Installing       : istack-commons-runtime-2.21-9.el8+7.noarch                                                                     174/334 
  Installing       : jboss-annotations-1.2-api-1.0.0-4.el8.noarch                                                                   175/334 
  Installing       : python3-werkzeug-0.12.2-4.el8.noarch                                                                           176/334 
  Installing       : jboss-jaxrs-2.0-api-1.0.0-6.el8.noarch                                                                         177/334 
  Installing       : relaxngDatatype-2011.1-7.module+el8+2468+c564cec5.noarch                                                       178/334 
  Installing       : xsom-0-19.20110809svn.module+el8+2468+c564cec5.noarch                                                          179/334 
  Installing       : jboss-logging-3.3.0-5.el8.noarch                                                                               180/334 
  Installing       : python3-jsonschema-2.6.0-4.el8.noarch                                                                          181/334 
  Installing       : glassfish-jaxb-txw2-2.2.11-11.module+el8+2468+c564cec5.noarch                                                  182/334 
  Installing       : python3-click-6.7-8.el8.noarch                                                                                 183/334 
  Installing       : jdeparser-2.0.0-5.el8.noarch                                                                                   184/334 
  Installing       : jboss-logging-tools-2.0.1-6.el8.noarch                                                                         185/334 
  Installing       : bea-stax-api-1.2.0-16.module+el8+2468+c564cec5.noarch                                                          186/334 
  Installing       : stax-ex-1.7.7-8.module+el8+2468+c564cec5.noarch                                                                187/334 
  Installing       : xmlstreambuffer-1.5.4-8.module+el8+2468+c564cec5.noarch                                                        188/334 
  Installing       : glassfish-fastinfoset-1.2.13-9.module+el8+2468+c564cec5.noarch                                                 189/334 
  Installing       : python3-itsdangerous-0.24-14.el8.noarch                                                                        190/334 
  Installing       : python3-flask-1:0.12.2-4.el8.noarch                                                                            191/334 
  Installing       : perl-Text-Unidecode-1.30-5.el8.noarch                                                                          192/334 
  Installing       : texlive-texlive.infra-7:20180414-23.el8.noarch                                                                 193/334 
  Installing       : texlive-tetex-7:20180414-23.el8.noarch                                                                         194/334 
  Installing       : texlive-kpathsea-7:20180414-23.el8.x86_64                                                                      195/334 
  Running scriptlet: texlive-kpathsea-7:20180414-23.el8.x86_64                                                                      195/334 
  Installing       : texlive-dvipng-7:20180414-23.el8.x86_64                                                                        196/334 
  Running scriptlet: texlive-dvipng-7:20180414-23.el8.x86_64                                                                        196/334 
  Installing       : glassfish-jaxb-api-2.2.12-8.module+el8+2468+c564cec5.noarch                                                    197/334 
  Installing       : glassfish-jaxb-core-2.2.11-11.module+el8+2468+c564cec5.noarch                                                  198/334 
  Installing       : glassfish-jaxb-runtime-2.2.11-11.module+el8+2468+c564cec5.noarch                                               199/334 
  Installing       : resteasy-3.0.26-6.module+el8.4.0+8891+bb8828ef.noarch                                                          200/334 
  Installing       : openstack-java-resteasy-connector-3.2.9-9.el8.noarch                                                           201/334 
  Installing       : xorg-x11-fonts-ISO8859-1-100dpi-7.5-19.el8.noarch                                                              202/334 
  Running scriptlet: xorg-x11-fonts-ISO8859-1-100dpi-7.5-19.el8.noarch                                                              202/334 
  Installing       : graphviz-2.40.1-43.el8.x86_64                                                                                  203/334 
  Running scriptlet: graphviz-2.40.1-43.el8.x86_64                                                                                  203/334 
  Installing       : python3-pydot-1.4.1-1.el8.noarch                                                                               204/334 
  Installing       : python3-pygraphviz-1.5-9.el8.x86_64                                                                            205/334 
  Installing       : python3-zstd-1.4.5.1-1.el8.x86_64                                                                              206/334 
  Installing       : python3-yappi-1.2.5-1.el8.x86_64                                                                               207/334 
  Installing       : python3-statsd-3.2.1-16.el8.noarch                                                                             208/334 
  Installing       : python3-sqlparse-0.3.1-3.el8.noarch                                                                            209/334 
  Installing       : python3-migrate-0.13.0-1.el8.noarch                                                                            210/334 
  Installing       : python3-rfc3986-1.4.0-3.el8.noarch                                                                             211/334 
  Installing       : python3-repoze-lru-0.7-6.el8.noarch                                                                            212/334 
  Installing       : python3-routes-2.4.1-12.el8.noarch                                                                             213/334 
  Installing       : python3-redis-3.3.8-1.el8.noarch                                                                               214/334 
  Installing       : python3-packaging-20.4-1.el8.noarch                                                                            215/334 
  Installing       : python3-oslo-rootwrap-6.2.0-2.el8.noarch                                                                       216/334 
  Installing       : python3-kiwisolver-1.1.0-4.el8.x86_64                                                                          217/334 
  Installing       : python3-kazoo-2.8.0-1.el8.noarch                                                                               218/334 
  Installing       : python3-zake-0.2.2-18.el8.noarch                                                                               219/334 
  Installing       : python3-editor-1.0.4-4.el8.noarch                                                                              220/334 
  Installing       : python3-alembic-1.4.2-5.el8.noarch                                                                             221/334 
  Installing       : python3-cycler-0.10.0-13.el8.noarch                                                                            222/334 
  Installing       : python3-matplotlib-tk-3.1.1-2.el8.x86_64                                                                       223/334 
  Installing       : python3-matplotlib-3.1.1-2.el8.x86_64                                                                          224/334 
  Installing       : python-oslo-versionedobjects-lang-2.3.0-2.el8.noarch                                                           225/334 
  Installing       : python-oslo-utils-lang-4.6.0-2.el8.noarch                                                                      226/334 
  Installing       : python-oslo-privsep-lang-2.4.0-2.el8.noarch                                                                    227/334 
  Installing       : python-oslo-middleware-lang-4.1.1-2.el8.noarch                                                                 228/334 
  Installing       : python-oslo-log-lang-4.4.0-2.el8.noarch                                                                        229/334 
  Installing       : python-oslo-i18n-lang-5.0.1-2.el8.noarch                                                                       230/334 
  Installing       : python3-oslo-i18n-5.0.1-2.el8.noarch                                                                           231/334 
  Installing       : python-oslo-db-lang-8.4.1-1.el8.noarch                                                                         232/334 
  Installing       : python-oslo-concurrency-lang-4.3.1-1.el8.noarch                                                                233/334 
  Installing       : blosc-1.17.0-1.el8.x86_64                                                                                      234/334 
  Installing       : python3-tables-3.5.2-6.el8.x86_64                                                                              235/334 
  Installing       : python3-pandas-0.25.3-1.el8.x86_64                                                                             236/334 
  Installing       : python3-networkx-2.5-1.el8.noarch                                                                              237/334 
  Running scriptlet: openvswitch-selinux-extra-policy-1.0-28.el8.noarch                                                             238/334 
  Installing       : openvswitch-selinux-extra-policy-1.0-28.el8.noarch                                                             238/334 
  Running scriptlet: openvswitch-selinux-extra-policy-1.0-28.el8.noarch                                                             238/334 
  Running scriptlet: openvswitch2.11-2.11.3-90.el8s.x86_64                                                                          239/334 
  Installing       : openvswitch2.11-2.11.3-90.el8s.x86_64                                                                          239/334 
  Running scriptlet: openvswitch2.11-2.11.3-90.el8s.x86_64                                                                          239/334 
  Installing       : ovirt-openvswitch-2.11-1.el8.noarch                                                                            240/334 
  Installing       : python3-openvswitch2.11-2.11.3-90.el8s.x86_64                                                                  241/334 
  Installing       : ovirt-python-openvswitch-2.11-1.el8.noarch                                                                     242/334 
  Installing       : python3-ovsdbapp-0.17.5-1.el8.noarch                                                                           243/334 
  Installing       : ovirt-provider-ovn-1.2.34-1.el8.noarch                                                                         244/334 
  Running scriptlet: ovirt-provider-ovn-1.2.34-1.el8.noarch                                                                         244/334 
  Installing       : python3-pyasn1-0.4.6-3.el8.noarch                                                                              245/334 
  Installing       : python3-websocket-client-0.56.0-5.el8.noarch                                                                   246/334 
  Installing       : python3-passlib-1.7.1-6.el8.noarch                                                                             247/334 
  Installing       : python3-notario-0.0.16-4.el8.noarch                                                                            248/334 
  Installing       : python3-lexicon-1.0.0-9.el8.noarch                                                                             249/334 
  Installing       : python3-fluidity-sm-0.2.0-16.el8.noarch                                                                        250/334 
  Installing       : python3-invoke-1.4.0-1.el8.noarch                                                                              251/334 
  Installing       : python3-aniso8601-8.0.0-1.el8.noarch                                                                           252/334 
  Installing       : python3-flask-restful-0.3.7-5.el8.noarch                                                                       253/334 
  Installing       : ed25519-java-0.3.0-1.el8.noarch                                                                                254/334 
  Installing       : ebay-cors-filter-1.0.1-4.el8.noarch                                                                            255/334 
  Installing       : python3-zipp-0.5.1-3.el8.noarch                                                                                256/334 
  Installing       : python3-importlib-metadata-1.7.0-1.el8.noarch                                                                  257/334 
  Installing       : python3-stevedore-3.2.2-2.el8.noarch                                                                           258/334 
  Installing       : python3-kombu-1:4.6.11-2.el8.noarch                                                                            259/334 
  Installing       : python3-xmltodict-0.12.0-4.el8.noarch                                                                          260/334 
  Installing       : python3-wrapt-1.11.2-4.el8.x86_64                                                                              261/334 
  Installing       : python3-debtcollector-1.22.0-2.el8.noarch                                                                      262/334 
  Installing       : python3-oslo-utils-4.6.0-2.el8.noarch                                                                          263/334 
  Installing       : python3-oslo-config-2:8.3.4-1.el8.noarch                                                                       264/334 
  Installing       : python3-oslo-serialization-4.0.1-2.el8.noarch                                                                  265/334 
  Installing       : python3-oslo-concurrency-4.3.1-1.el8.noarch                                                                    266/334 
  Installing       : python3-oslo-context-3.1.2-1.el8.noarch                                                                        267/334 
  Installing       : python3-oslo-log-4.4.0-2.el8.noarch                                                                            268/334 
  Installing       : python3-oslo-middleware-4.1.1-2.el8.noarch                                                                     269/334 
  Installing       : python3-taskflow-4.5.0-2.el8.noarch                                                                            270/334 
  Installing       : python3-oslo-db-8.4.1-1.el8.noarch                                                                             271/334 
  Installing       : python3-wcwidth-0.1.7-14.el8.noarch                                                                            272/334 
  Installing       : python3-tabulate-0.8.7-4.el8.noarch                                                                            273/334 
  Installing       : python3-voluptuous-0.11.7-2.el8.noarch                                                                         274/334 
  Installing       : python3-tooz-2.7.2-1.el8.noarch                                                                                275/334 
  Installing       : python3-requests_ntlm-1.1.0-8.el8.noarch                                                                       276/334 
  Installing       : python3-winrm-0.3.0-7.el8.noarch                                                                               277/334 
  Installing       : python3-monotonic-1.5-5.el8.noarch                                                                             278/334 
  Installing       : python3-eventlet-0.25.2-3.1.el8.noarch                                                                         279/334 
  Installing       : python3-oslo-service-2.4.0-2.el8.noarch                                                                        280/334 
  Installing       : python3-oslo-privsep-2.4.0-2.el8.noarch                                                                        281/334 
  Installing       : python3-os-win-5.2.0-1.el8.noarch                                                                              282/334 
  Installing       : python3-os-brick-4.0.4-1.el8.noarch                                                                            283/334 
  Installing       : sshpass-1.06-9.el8.x86_64                                                                                      284/334 
  Installing       : qpid-proton-c-0.36.0-1.el8.x86_64                                                                              285/334 
  Running scriptlet: qpid-proton-c-0.36.0-1.el8.x86_64                                                                              285/334 
  Installing       : python3-qpid-proton-0.36.0-1.el8.x86_64                                                                        286/334 
  Installing       : python3-pyngus-2.3.0-4.el8.noarch                                                                              287/334 
  Installing       : python3-oslo-messaging-12.5.2-1.el8.noarch                                                                     288/334 
  Installing       : python3-oslo-versionedobjects-2.3.0-2.el8.noarch                                                               289/334 
  Installing       : libsodium-1.0.18-2.el8.x86_64                                                                                  290/334 
  Installing       : python3-pynacl-1.3.0-5.el8.x86_64                                                                              291/334 
  Installing       : python3-paramiko-2.7.2-2.el8.noarch                                                                            292/334 
  Installing       : ansible-2.9.27-2.el8.noarch                                                                                    293/334 
  Installing       : ovirt-engine-metrics-1.4.4-1.el8.noarch                                                                        294/334 
  Installing       : ansible-runner-service-1.0.7-1.el8.noarch                                                                      295/334 
  Running scriptlet: ansible-runner-service-1.0.7-1.el8.noarch                                                                      295/334 
  Installing       : python3-cinder-common-1:17.2.0-1.el8.noarch                                                                    296/334 
  Installing       : python3-cinderlib-1:3.0.0-1.el8.noarch                                                                         297/334 
  Installing       : python3-ovirt-engine-sdk4-4.4.15-1.el8.x86_64                                                                  298/334 
  Installing       : ovirt-ansible-collection-1.6.6-1.el8.noarch                                                                    299/334 
  Installing       : ovirt-web-ui-1.7.2-1.el8.noarch                                                                                300/334 
  Installing       : ovirt-imageio-common-2.3.0-1.el8.x86_64                                                                        301/334 
  Running scriptlet: ovirt-imageio-daemon-2.3.0-1.el8.x86_64                                                                        302/334 
  Installing       : ovirt-imageio-daemon-2.3.0-1.el8.x86_64                                                                        302/334 
  Running scriptlet: ovirt-imageio-daemon-2.3.0-1.el8.x86_64                                                                        302/334 
  Installing       : ovirt-dependencies-4.4.2-1.el8.noarch                                                                          303/334 
  Installing       : ovirt-cockpit-sso-0.1.4-2.el8.noarch                                                                           304/334 
  Running scriptlet: ovirt-cockpit-sso-0.1.4-2.el8.noarch                                                                           304/334 
  Installing       : otopi-common-1.9.6-1.el8.noarch                                                                                305/334 
  Installing       : python3-otopi-1.9.6-1.el8.noarch                                                                               306/334 
  Running scriptlet: ovirt-engine-setup-base-4.4.10.6-1.el8.noarch                                                                  307/334 
  Installing       : ovirt-engine-setup-base-4.4.10.6-1.el8.noarch                                                                  307/334 
  Installing       : ovirt-engine-setup-plugin-ovirt-engine-common-4.4.10.6-1.el8.noarch                                            308/334 
  Running scriptlet: ovirt-engine-dwh-4.4.10-1.el8.noarch                                                                           309/334 
  Installing       : ovirt-engine-dwh-4.4.10-1.el8.noarch                                                                           309/334 
  Running scriptlet: ovirt-engine-dwh-4.4.10-1.el8.noarch                                                                           309/334 
  Installing       : ovirt-engine-dwh-grafana-integration-setup-4.4.10-1.el8.noarch                                                 310/334 
  Installing       : ovirt-engine-dwh-setup-4.4.10-1.el8.noarch                                                                     311/334 
  Installing       : ovirt-engine-setup-plugin-websocket-proxy-4.4.10.6-1.el8.noarch                                                312/334 
  Running scriptlet: ovirt-engine-websocket-proxy-4.4.10.6-1.el8.noarch                                                             313/334 
  Installing       : ovirt-engine-websocket-proxy-4.4.10.6-1.el8.noarch                                                             313/334 
  Running scriptlet: ovirt-engine-websocket-proxy-4.4.10.6-1.el8.noarch                                                             313/334 
  Installing       : ovirt-engine-tools-backup-4.4.10.6-1.el8.noarch                                                                314/334 
  Installing       : java-client-kubevirt-0.5.0-1.el8.noarch                                                                        315/334 
  Installing       : python3-ceph-common-2:16.2.7-1.el8s.x86_64                                                                     316/334 
  Installing       : libunwind-1.4.0-5.el8s.x86_64                                                                                  317/334 
  Installing       : gperftools-libs-2.9.1-1.el8s.x86_64                                                                            318/334 
  Installing       : libradosstriper1-2:16.2.7-1.el8s.x86_64                                                                        319/334 
  Running scriptlet: libradosstriper1-2:16.2.7-1.el8s.x86_64                                                                        319/334 
  Installing       : leveldb-1.20-1.el8s.x86_64                                                                                     320/334 
  Running scriptlet: leveldb-1.20-1.el8s.x86_64                                                                                     320/334 
  Running scriptlet: ceph-common-2:16.2.7-1.el8s.x86_64                                                                             321/334 
  Installing       : ceph-common-2:16.2.7-1.el8s.x86_64                                                                             321/334 
  Running scriptlet: ceph-common-2:16.2.7-1.el8s.x86_64                                                                             321/334 
  Running scriptlet: ovirt-engine-backend-4.4.10.6-1.el8.noarch                                                                     322/334 
  Installing       : ovirt-engine-backend-4.4.10.6-1.el8.noarch                                                                     322/334 
  Running scriptlet: ovirt-engine-backend-4.4.10.6-1.el8.noarch                                                                     322/334 
  Installing       : ovirt-engine-dbscripts-4.4.10.6-1.el8.noarch                                                                   323/334 
  Installing       : ovirt-engine-restapi-4.4.10.6-1.el8.noarch                                                                     324/334 
  Installing       : ovirt-engine-setup-4.4.10.6-1.el8.noarch                                                                       325/334 
  Running scriptlet: ovirt-engine-setup-4.4.10.6-1.el8.noarch                                                                       325/334 
  Installing       : ovirt-engine-setup-plugin-cinderlib-4.4.10.6-1.el8.noarch                                                      326/334 
  Installing       : ovirt-engine-setup-plugin-imageio-4.4.10.6-1.el8.noarch                                                        327/334 
  Installing       : ovirt-engine-vmconsole-proxy-helper-4.4.10.6-1.el8.noarch                                                      328/334 
  Installing       : ovirt-engine-setup-plugin-vmconsole-proxy-helper-4.4.10.6-1.el8.noarch                                         329/334 
  Running scriptlet: ovirt-engine-setup-plugin-ovirt-engine-4.4.10.6-1.el8.noarch                                                   330/334 
  Installing       : ovirt-engine-setup-plugin-ovirt-engine-4.4.10.6-1.el8.noarch                                                   330/334 
  Running scriptlet: ovirt-engine-tools-4.4.10.6-1.el8.noarch                                                                       331/334 
  Installing       : ovirt-engine-tools-4.4.10.6-1.el8.noarch                                                                       331/334 
  Running scriptlet: ovirt-engine-tools-4.4.10.6-1.el8.noarch                                                                       331/334 
  Installing       : ovirt-engine-ui-extensions-1.2.7-1.el8.noarch                                                                  332/334 
  Installing       : ovirt-engine-webadmin-portal-4.4.10.6-1.el8.noarch                                                             333/334 
  Running scriptlet: ovirt-engine-4.4.10.6-1.el8.noarch                                                                             334/334 
  Installing       : ovirt-engine-4.4.10.6-1.el8.noarch                                                                             334/334 
  Running scriptlet: texlive-base-7:20180414-23.el8.noarch                                                                          334/334 
  Running scriptlet: ovirt-vmconsole-1.0.9-1.el8.noarch                                                                             334/334 
  Running scriptlet: ovirt-vmconsole-proxy-1.0.9-1.el8.noarch                                                                       334/334 
  Running scriptlet: ovn2.11-central-2.11.1-57.el8s.x86_64                                                                          334/334 
  Running scriptlet: grafana-pcp-3.1.0-1.el8.x86_64                                                                                 334/334 
  Running scriptlet: httpd-2.4.37-43.module+el8.5.0+13806+b30d9eec.1.x86_64                                                         334/334 
  Running scriptlet: openvswitch-selinux-extra-policy-1.0-28.el8.noarch                                                             334/334 
  Running scriptlet: ovirt-openvswitch-2.11-1.el8.noarch                                                                            334/334 
  Running scriptlet: ovirt-imageio-daemon-2.3.0-1.el8.x86_64                                                                        334/334 
  Running scriptlet: ovirt-engine-setup-4.4.10.6-1.el8.noarch                                                                       334/334 
  Running scriptlet: ovirt-engine-4.4.10.6-1.el8.noarch                                                                             334/334 
[/usr/lib/tmpfiles.d/postgresql.conf:1] Line references path below legacy directory /var/run/, updating /var/run/postgresql → /run/postgresql; please update the tmpfiles.d/ drop-in file accordingly.

  Running scriptlet: texlive-kpathsea-7:20180414-23.el8.x86_64                                                                      334/334 
  Verifying        : ceph-common-2:16.2.7-1.el8s.x86_64                                                                               1/334 
  Verifying        : gperftools-libs-2.9.1-1.el8s.x86_64                                                                              2/334 
  Verifying        : leveldb-1.20-1.el8s.x86_64                                                                                       3/334 
  Verifying        : libcephfs2-2:16.2.7-1.el8s.x86_64                                                                                4/334 
  Verifying        : liboath-2.6.2-4.el8s.x86_64                                                                                      5/334 
  Verifying        : libradosstriper1-2:16.2.7-1.el8s.x86_64                                                                          6/334 
  Verifying        : librgw2-2:16.2.7-1.el8s.x86_64                                                                                   7/334 
  Verifying        : libunwind-1.4.0-5.el8s.x86_64                                                                                    8/334 
  Verifying        : python3-ceph-argparse-2:16.2.7-1.el8s.x86_64                                                                     9/334 
  Verifying        : python3-ceph-common-2:16.2.7-1.el8s.x86_64                                                                      10/334 
  Verifying        : python3-cephfs-2:16.2.7-1.el8s.x86_64                                                                           11/334 
  Verifying        : python3-rados-2:16.2.7-1.el8s.x86_64                                                                            12/334 
  Verifying        : python3-rbd-2:16.2.7-1.el8s.x86_64                                                                              13/334 
  Verifying        : python3-rgw-2:16.2.7-1.el8s.x86_64                                                                              14/334 
  Verifying        : java-client-kubevirt-0.5.0-1.el8.noarch                                                                         15/334 
  Verifying        : otopi-common-1.9.6-1.el8.noarch                                                                                 16/334 
  Verifying        : ovirt-ansible-collection-1.6.6-1.el8.noarch                                                                     17/334 
  Verifying        : ovirt-cockpit-sso-0.1.4-2.el8.noarch                                                                            18/334 
  Verifying        : ovirt-dependencies-4.4.2-1.el8.noarch                                                                           19/334 
  Verifying        : ovirt-engine-4.4.10.6-1.el8.noarch                                                                              20/334 
  Verifying        : ovirt-engine-backend-4.4.10.6-1.el8.noarch                                                                      21/334 
  Verifying        : ovirt-engine-dbscripts-4.4.10.6-1.el8.noarch                                                                    22/334 
  Verifying        : ovirt-engine-dwh-4.4.10-1.el8.noarch                                                                            23/334 
  Verifying        : ovirt-engine-dwh-grafana-integration-setup-4.4.10-1.el8.noarch                                                  24/334 
  Verifying        : ovirt-engine-dwh-setup-4.4.10-1.el8.noarch                                                                      25/334 
  Verifying        : ovirt-engine-extension-aaa-jdbc-1.2.0-1.el8.noarch                                                              26/334 
  Verifying        : ovirt-engine-extensions-api-1.0.1-1.el8.noarch                                                                  27/334 
  Verifying        : ovirt-engine-metrics-1.4.4-1.el8.noarch                                                                         28/334 
  Verifying        : ovirt-engine-restapi-4.4.10.6-1.el8.noarch                                                                      29/334 
  Verifying        : ovirt-engine-setup-4.4.10.6-1.el8.noarch                                                                        30/334 
  Verifying        : ovirt-engine-setup-base-4.4.10.6-1.el8.noarch                                                                   31/334 
  Verifying        : ovirt-engine-setup-plugin-cinderlib-4.4.10.6-1.el8.noarch                                                       32/334 
  Verifying        : ovirt-engine-setup-plugin-imageio-4.4.10.6-1.el8.noarch                                                         33/334 
  Verifying        : ovirt-engine-setup-plugin-ovirt-engine-4.4.10.6-1.el8.noarch                                                    34/334 
  Verifying        : ovirt-engine-setup-plugin-ovirt-engine-common-4.4.10.6-1.el8.noarch                                             35/334 
  Verifying        : ovirt-engine-setup-plugin-vmconsole-proxy-helper-4.4.10.6-1.el8.noarch                                          36/334 
  Verifying        : ovirt-engine-setup-plugin-websocket-proxy-4.4.10.6-1.el8.noarch                                                 37/334 
  Verifying        : ovirt-engine-tools-4.4.10.6-1.el8.noarch                                                                        38/334 
  Verifying        : ovirt-engine-tools-backup-4.4.10.6-1.el8.noarch                                                                 39/334 
  Verifying        : ovirt-engine-ui-extensions-1.2.7-1.el8.noarch                                                                   40/334 
  Verifying        : ovirt-engine-vmconsole-proxy-helper-4.4.10.6-1.el8.noarch                                                       41/334 
  Verifying        : ovirt-engine-webadmin-portal-4.4.10.6-1.el8.noarch                                                              42/334 
  Verifying        : ovirt-engine-websocket-proxy-4.4.10.6-1.el8.noarch                                                              43/334 
  Verifying        : ovirt-engine-wildfly-23.0.2-1.el8.x86_64                                                                        44/334 
  Verifying        : ovirt-engine-wildfly-overlay-23.0.2-1.el8.noarch                                                                45/334 
  Verifying        : ovirt-imageio-common-2.3.0-1.el8.x86_64                                                                         46/334 
  Verifying        : ovirt-imageio-daemon-2.3.0-1.el8.x86_64                                                                         47/334 
  Verifying        : ovirt-provider-ovn-1.2.34-1.el8.noarch                                                                          48/334 
  Verifying        : ovirt-vmconsole-1.0.9-1.el8.noarch                                                                              49/334 
  Verifying        : ovirt-vmconsole-proxy-1.0.9-1.el8.noarch                                                                        50/334 
  Verifying        : ovirt-web-ui-1.7.2-1.el8.noarch                                                                                 51/334 
  Verifying        : python3-otopi-1.9.6-1.el8.noarch                                                                                52/334 
  Verifying        : python3-ovirt-engine-lib-4.4.10.6-1.el8.noarch                                                                  53/334 
  Verifying        : python3-ovirt-engine-sdk4-4.4.15-1.el8.x86_64                                                                   54/334 
  Verifying        : python3-ovirt-setup-lib-1.3.2-1.el8.noarch                                                                      55/334 
  Verifying        : vdsm-jsonrpc-java-1.6.0-1.el8.noarch                                                                            56/334 
  Verifying        : libsodium-1.0.18-2.el8.x86_64                                                                                   57/334 
  Verifying        : python3-pynacl-1.3.0-5.el8.x86_64                                                                               58/334 
  Verifying        : python3-qpid-proton-0.36.0-1.el8.x86_64                                                                         59/334 
  Verifying        : qpid-proton-c-0.36.0-1.el8.x86_64                                                                               60/334 
  Verifying        : sshpass-1.06-9.el8.x86_64                                                                                       61/334 
  Verifying        : python3-monotonic-1.5-5.el8.noarch                                                                              62/334 
  Verifying        : python3-requests_ntlm-1.1.0-8.el8.noarch                                                                        63/334 
  Verifying        : python3-voluptuous-0.11.7-2.el8.noarch                                                                          64/334 
  Verifying        : python3-wcwidth-0.1.7-14.el8.noarch                                                                             65/334 
  Verifying        : python3-winrm-0.3.0-7.el8.noarch                                                                                66/334 
  Verifying        : python3-wrapt-1.11.2-4.el8.x86_64                                                                               67/334 
  Verifying        : python3-xmltodict-0.12.0-4.el8.noarch                                                                           68/334 
  Verifying        : python3-zipp-0.5.1-3.el8.noarch                                                                                 69/334 
  Verifying        : ansible-2.9.27-2.el8.noarch                                                                                     70/334 
  Verifying        : ansible-runner-service-1.0.7-1.el8.noarch                                                                       71/334 
  Verifying        : apache-commons-configuration-1.10-1.el8.noarch                                                                  72/334 
  Verifying        : apache-sshd-2.6.0-2.el8.noarch                                                                                  73/334 
  Verifying        : ebay-cors-filter-1.0.1-4.el8.noarch                                                                             74/334 
  Verifying        : ed25519-java-0.3.0-1.el8.noarch                                                                                 75/334 
  Verifying        : novnc-1.1.0-6.el8.noarch                                                                                        76/334 
  Verifying        : openstack-java-cinder-client-3.2.9-9.el8.noarch                                                                 77/334 
  Verifying        : openstack-java-cinder-model-3.2.9-9.el8.noarch                                                                  78/334 
  Verifying        : openstack-java-client-3.2.9-9.el8.noarch                                                                        79/334 
  Verifying        : openstack-java-glance-client-3.2.9-9.el8.noarch                                                                 80/334 
  Verifying        : openstack-java-glance-model-3.2.9-9.el8.noarch                                                                  81/334 
  Verifying        : openstack-java-keystone-client-3.2.9-9.el8.noarch                                                               82/334 
  Verifying        : openstack-java-keystone-model-3.2.9-9.el8.noarch                                                                83/334 
  Verifying        : openstack-java-quantum-client-3.2.9-9.el8.noarch                                                                84/334 
  Verifying        : openstack-java-quantum-model-3.2.9-9.el8.noarch                                                                 85/334 
  Verifying        : openstack-java-resteasy-connector-3.2.9-9.el8.noarch                                                            86/334 
  Verifying        : ovirt-openvswitch-2.11-1.el8.noarch                                                                             87/334 
  Verifying        : ovirt-openvswitch-ovn-2.11-1.el8.noarch                                                                         88/334 
  Verifying        : ovirt-openvswitch-ovn-central-2.11-1.el8.noarch                                                                 89/334 
  Verifying        : ovirt-openvswitch-ovn-common-2.11-1.el8.noarch                                                                  90/334 
  Verifying        : ovirt-python-openvswitch-2.11-1.el8.noarch                                                                      91/334 
  Verifying        : python3-aniso8601-8.0.0-1.el8.noarch                                                                            92/334 
  Verifying        : python3-ansible-runner-1.4.6-1.el8.noarch                                                                       93/334 
  Verifying        : python3-bcrypt-3.1.7-3.el8.x86_64                                                                               94/334 
  Verifying        : python3-daemon-2.2.3-7.el8.noarch                                                                               95/334 
  Verifying        : python3-debtcollector-1.22.0-2.el8.noarch                                                                       96/334 
  Verifying        : python3-flask-restful-0.3.7-5.el8.noarch                                                                        97/334 
  Verifying        : python3-fluidity-sm-0.2.0-16.el8.noarch                                                                         98/334 
  Verifying        : python3-funcsigs-1.0.2-17.el8.noarch                                                                            99/334 
  Verifying        : python3-invoke-1.4.0-1.el8.noarch                                                                              100/334 
  Verifying        : python3-lexicon-1.0.0-9.el8.noarch                                                                             101/334 
  Verifying        : python3-lockfile-1:0.11.0-16.el8.noarch                                                                        102/334 
  Verifying        : python3-notario-0.0.16-4.el8.noarch                                                                            103/334 
  Verifying        : python3-ovsdbapp-0.17.5-1.el8.noarch                                                                           104/334 
  Verifying        : python3-paramiko-2.7.2-2.el8.noarch                                                                            105/334 
  Verifying        : python3-passlib-1.7.1-6.el8.noarch                                                                             106/334 
  Verifying        : python3-pbr-5.4.3-2.el8.noarch                                                                                 107/334 
  Verifying        : python3-websocket-client-0.56.0-5.el8.noarch                                                                   108/334 
  Verifying        : python3-websockify-0.8.0-15.el8.noarch                                                                         109/334 
  Verifying        : snmp4j-3.6.4-0.1.el8.noarch                                                                                    110/334 
  Verifying        : ws-commons-util-1.0.2-1.el8.noarch                                                                             111/334 
  Verifying        : xmlrpc-client-3.1.3-1.el8.noarch                                                                               112/334 
  Verifying        : xmlrpc-common-3.1.3-1.el8.noarch                                                                               113/334 
  Verifying        : collectd-5.12.0-7.el8s.x86_64                                                                                  114/334 
  Verifying        : collectd-disk-5.12.0-7.el8s.x86_64                                                                             115/334 
  Verifying        : collectd-postgresql-5.12.0-7.el8s.x86_64                                                                       116/334 
  Verifying        : collectd-write_http-5.12.0-7.el8s.x86_64                                                                       117/334 
  Verifying        : collectd-write_syslog-5.12.0-7.el8s.x86_64                                                                     118/334 
  Verifying        : python3-pyasn1-0.4.6-3.el8.noarch                                                                              119/334 
  Verifying        : openvswitch-selinux-extra-policy-1.0-28.el8.noarch                                                             120/334 
  Verifying        : openvswitch2.11-2.11.3-90.el8s.x86_64                                                                          121/334 
  Verifying        : ovn2.11-2.11.1-57.el8s.x86_64                                                                                  122/334 
  Verifying        : ovn2.11-central-2.11.1-57.el8s.x86_64                                                                          123/334 
  Verifying        : python3-openvswitch2.11-2.11.3-90.el8s.x86_64                                                                  124/334 
  Verifying        : blosc-1.17.0-1.el8.x86_64                                                                                      125/334 
  Verifying        : hdf5-1.10.5-5.el8.x86_64                                                                                       126/334 
  Verifying        : python-oslo-concurrency-lang-4.3.1-1.el8.noarch                                                                127/334 
  Verifying        : python-oslo-db-lang-8.4.1-1.el8.noarch                                                                         128/334 
  Verifying        : python-oslo-i18n-lang-5.0.1-2.el8.noarch                                                                       129/334 
  Verifying        : python-oslo-log-lang-4.4.0-2.el8.noarch                                                                        130/334 
  Verifying        : python-oslo-middleware-lang-4.1.1-2.el8.noarch                                                                 131/334 
  Verifying        : python-oslo-privsep-lang-2.4.0-2.el8.noarch                                                                    132/334 
  Verifying        : python-oslo-utils-lang-4.6.0-2.el8.noarch                                                                      133/334 
  Verifying        : python-oslo-versionedobjects-lang-2.3.0-2.el8.noarch                                                           134/334 
  Verifying        : python3-Bottleneck-1.2.1-13.el8.x86_64                                                                         135/334 
  Verifying        : python3-alembic-1.4.2-5.el8.noarch                                                                             136/334 
  Verifying        : python3-amqp-2.6.1-1.el8.noarch                                                                                137/334 
  Verifying        : python3-automaton-2.2.0-1.el8.noarch                                                                           138/334 
  Verifying        : python3-cachetools-4.1.1-2.el8.noarch                                                                          139/334 
  Verifying        : python3-cinder-common-1:17.2.0-1.el8.noarch                                                                    140/334 
  Verifying        : python3-cinderlib-1:3.0.0-1.el8.noarch                                                                         141/334 
  Verifying        : python3-cycler-0.10.0-13.el8.noarch                                                                            142/334 
  Verifying        : python3-editor-1.0.4-4.el8.noarch                                                                              143/334 
  Verifying        : python3-eventlet-0.25.2-3.1.el8.noarch                                                                         144/334 
  Verifying        : python3-fasteners-0.14.1-20.el8.noarch                                                                         145/334 
  Verifying        : python3-futurist-2.3.0-2.el8.noarch                                                                            146/334 
  Verifying        : python3-importlib-metadata-1.7.0-1.el8.noarch                                                                  147/334 
  Verifying        : python3-iso8601-0.1.12-3.el8.noarch                                                                            148/334 
  Verifying        : python3-kazoo-2.8.0-1.el8.noarch                                                                               149/334 
  Verifying        : python3-kiwisolver-1.1.0-4.el8.x86_64                                                                          150/334 
  Verifying        : python3-kombu-1:4.6.11-2.el8.noarch                                                                            151/334 
  Verifying        : python3-matplotlib-3.1.1-2.el8.x86_64                                                                          152/334 
  Verifying        : python3-matplotlib-data-3.1.1-2.el8.noarch                                                                     153/334 
  Verifying        : python3-matplotlib-data-fonts-3.1.1-2.el8.noarch                                                               154/334 
  Verifying        : python3-matplotlib-tk-3.1.1-2.el8.x86_64                                                                       155/334 
  Verifying        : python3-migrate-0.13.0-1.el8.noarch                                                                            156/334 
  Verifying        : python3-msgpack-1.0.0-2.el8.x86_64                                                                             157/334 
  Verifying        : python3-networkx-2.5-1.el8.noarch                                                                              158/334 
  Verifying        : python3-numexpr-2.7.1-1.el8.x86_64                                                                             159/334 
  Verifying        : python3-os-brick-4.0.4-1.el8.noarch                                                                            160/334 
  Verifying        : python3-os-win-5.2.0-1.el8.noarch                                                                              161/334 
  Verifying        : python3-oslo-concurrency-4.3.1-1.el8.noarch                                                                    162/334 
  Verifying        : python3-oslo-config-2:8.3.4-1.el8.noarch                                                                       163/334 
  Verifying        : python3-oslo-context-3.1.2-1.el8.noarch                                                                        164/334 
  Verifying        : python3-oslo-db-8.4.1-1.el8.noarch                                                                             165/334 
  Verifying        : python3-oslo-i18n-5.0.1-2.el8.noarch                                                                           166/334 
  Verifying        : python3-oslo-log-4.4.0-2.el8.noarch                                                                            167/334 
  Verifying        : python3-oslo-messaging-12.5.2-1.el8.noarch                                                                     168/334 
  Verifying        : python3-oslo-middleware-4.1.1-2.el8.noarch                                                                     169/334 
  Verifying        : python3-oslo-privsep-2.4.0-2.el8.noarch                                                                        170/334 
  Verifying        : python3-oslo-rootwrap-6.2.0-2.el8.noarch                                                                       171/334 
  Verifying        : python3-oslo-serialization-4.0.1-2.el8.noarch                                                                  172/334 
  Verifying        : python3-oslo-service-2.4.0-2.el8.noarch                                                                        173/334 
  Verifying        : python3-oslo-utils-4.6.0-2.el8.noarch                                                                          174/334 
  Verifying        : python3-oslo-versionedobjects-2.3.0-2.el8.noarch                                                               175/334 
  Verifying        : python3-packaging-20.4-1.el8.noarch                                                                            176/334 
  Verifying        : python3-pandas-0.25.3-1.el8.x86_64                                                                             177/334 
  Verifying        : python3-paste-3.2.4-1.el8.noarch                                                                               178/334 
  Verifying        : python3-paste-deploy-2.1.0-3.el8.noarch                                                                        179/334 
  Verifying        : python3-pydot-1.4.1-1.el8.noarch                                                                               180/334 
  Verifying        : python3-pygraphviz-1.5-9.el8.x86_64                                                                            181/334 
  Verifying        : python3-pyngus-2.3.0-4.el8.noarch                                                                              182/334 
  Verifying        : python3-redis-3.3.8-1.el8.noarch                                                                               183/334 
  Verifying        : python3-repoze-lru-0.7-6.el8.noarch                                                                            184/334 
  Verifying        : python3-rfc3986-1.4.0-3.el8.noarch                                                                             185/334 
  Verifying        : python3-routes-2.4.1-12.el8.noarch                                                                             186/334 
  Verifying        : python3-sqlparse-0.3.1-3.el8.noarch                                                                            187/334 
  Verifying        : python3-statsd-3.2.1-16.el8.noarch                                                                             188/334 
  Verifying        : python3-stevedore-3.2.2-2.el8.noarch                                                                           189/334 
  Verifying        : python3-tables-3.5.2-6.el8.x86_64                                                                              190/334 
  Verifying        : python3-tabulate-0.8.7-4.el8.noarch                                                                            191/334 
  Verifying        : python3-taskflow-4.5.0-2.el8.noarch                                                                            192/334 
  Verifying        : python3-tempita-0.5.1-25.el8.noarch                                                                            193/334 
  Verifying        : python3-tenacity-6.2.0-1.el8.noarch                                                                            194/334 
  Verifying        : python3-tooz-2.7.2-1.el8.noarch                                                                                195/334 
  Verifying        : python3-vine-1.3.0-4.el8.noarch                                                                                196/334 
  Verifying        : python3-webob-1.8.6-3.el8.noarch                                                                               197/334 
  Verifying        : python3-yappi-1.2.5-1.el8.x86_64                                                                               198/334 
  Verifying        : python3-zake-0.2.2-18.el8.noarch                                                                               199/334 
  Verifying        : python3-zstd-1.4.5.1-1.el8.x86_64                                                                              200/334 
  Verifying        : python3-prettytable-0.7.2-14.el8.noarch                                                                        201/334 
  Verifying        : python3-netaddr-0.7.19-8.el8.noarch                                                                            202/334 
  Verifying        : xorg-x11-fonts-ISO8859-1-100dpi-7.5-19.el8.noarch                                                              203/334 
  Verifying        : glassfish-jaxb-api-2.2.12-8.module+el8+2468+c564cec5.noarch                                                    204/334 
  Verifying        : glassfish-jaxb-core-2.2.11-11.module+el8+2468+c564cec5.noarch                                                  205/334 
  Verifying        : perl-Text-Unidecode-1.30-5.el8.noarch                                                                          206/334 
  Verifying        : xmlstreambuffer-1.5.4-8.module+el8+2468+c564cec5.noarch                                                        207/334 
  Verifying        : python3-itsdangerous-0.24-14.el8.noarch                                                                        208/334 
  Verifying        : python3-mako-1.0.6-13.el8.noarch                                                                               209/334 
  Verifying        : python3-jmespath-0.9.0-11.el8.noarch                                                                           210/334 
  Verifying        : bea-stax-api-1.2.0-16.module+el8+2468+c564cec5.noarch                                                          211/334 
  Verifying        : stax-ex-1.7.7-8.module+el8+2468+c564cec5.noarch                                                                212/334 
  Verifying        : glassfish-fastinfoset-1.2.13-9.module+el8+2468+c564cec5.noarch                                                 213/334 
  Verifying        : perl-XML-XPath-1.42-3.el8.noarch                                                                               214/334 
  Verifying        : jdeparser-2.0.0-5.el8.noarch                                                                                   215/334 
  Verifying        : jboss-logging-tools-2.0.1-6.el8.noarch                                                                         216/334 
  Verifying        : python3-click-6.7-8.el8.noarch                                                                                 217/334 
  Verifying        : glassfish-jaxb-txw2-2.2.11-11.module+el8+2468+c564cec5.noarch                                                  218/334 
  Verifying        : xsom-0-19.20110809svn.module+el8+2468+c564cec5.noarch                                                          219/334 
  Verifying        : python3-jsonschema-2.6.0-4.el8.noarch                                                                          220/334 
  Verifying        : ongres-scram-client-1.0.0~beta.2-5.el8.noarch                                                                  221/334 
  Verifying        : jboss-logging-3.3.0-5.el8.noarch                                                                               222/334 
  Verifying        : relaxngDatatype-2011.1-7.module+el8+2468+c564cec5.noarch                                                       223/334 
  Verifying        : jackson-module-jaxb-annotations-2.7.6-4.module+el8+2468+c564cec5.noarch                                        224/334 
  Verifying        : jboss-jaxrs-2.0-api-1.0.0-6.el8.noarch                                                                         225/334 
  Verifying        : python3-werkzeug-0.12.2-4.el8.noarch                                                                           226/334 
  Verifying        : glassfish-jaxb-runtime-2.2.11-11.module+el8+2468+c564cec5.noarch                                               227/334 
  Verifying        : jboss-annotations-1.2-api-1.0.0-4.el8.noarch                                                                   228/334 
  Verifying        : istack-commons-runtime-2.21-9.el8+7.noarch                                                                     229/334 
  Verifying        : ongres-scram-1.0.0~beta.2-5.el8.noarch                                                                         230/334 
  Verifying        : apr-util-openssl-1.6.1-6.el8.x86_64                                                                            231/334 
  Verifying        : python3-psycopg2-2.7.5-7.el8.x86_64                                                                            232/334 
  Verifying        : apr-util-bdb-1.6.1-6.el8.x86_64                                                                                233/334 
  Verifying        : perl-encoding-4:2.22-3.el8.x86_64                                                                              234/334 
  Verifying        : libXaw-1.0.13-10.el8.x86_64                                                                                    235/334 
  Verifying        : tk-1:8.6.8-1.el8.x86_64                                                                                        236/334 
  Verifying        : perl-XML-Parser-2.44-11.el8.x86_64                                                                             237/334 
  Verifying        : python3-netifaces-0.10.6-4.el8.x86_64                                                                          238/334 
  Verifying        : apr-util-1.6.1-6.el8.x86_64                                                                                    239/334 
  Verifying        : perl-Filter-2:1.58-2.el8.x86_64                                                                                240/334 
  Verifying        : python3-markupsafe-0.23-19.el8.x86_64                                                                          241/334 
  Verifying        : sysfsutils-2.1.0-24.el8.x86_64                                                                                 242/334 
  Verifying        : librdkafka-0.11.4-1.el8.x86_64                                                                                 243/334 
  Verifying        : python3-distro-1.4.0-2.module+el8.1.0+3334+5cb623d7.noarch                                                     244/334 
  Verifying        : jackson-jaxrs-providers-2.9.9-1.module+el8.1.0+3832+9784644d.noarch                                            245/334 
  Verifying        : python3-docutils-0.14-12.module+el8.1.0+3334+5cb623d7.noarch                                                   246/334 
  Verifying        : jackson-jaxrs-json-provider-2.9.9-1.module+el8.1.0+3832+9784644d.noarch                                        247/334 
  Verifying        : jackson-annotations-2.10.0-1.module+el8.2.0+5059+3eb3af25.noarch                                               248/334 
  Verifying        : jackson-databind-2.10.0-1.module+el8.2.0+5059+3eb3af25.noarch                                                  249/334 
  Verifying        : jackson-core-2.10.0-1.module+el8.2.0+5059+3eb3af25.noarch                                                      250/334 
  Verifying        : python3-greenlet-0.4.13-4.el8.x86_64                                                                           251/334 
  Verifying        : postgresql-jdbc-42.2.3-3.el8_2.noarch                                                                          252/334 
  Verifying        : python3-mod_wsgi-4.6.4-4.el8.x86_64                                                                            253/334 
  Verifying        : python3-flask-1:0.12.2-4.el8.noarch                                                                            254/334 
  Verifying        : python3-sqlalchemy-1.3.2-2.module+el8.3.0+6646+6b4b10ec.x86_64                                                 255/334 
  Verifying        : git-core-2.27.0-1.el8.x86_64                                                                                   256/334 
  Verifying        : nodejs-full-i18n-1:10.24.0-1.module+el8.3.0+10166+b07ac28e.x86_64                                              257/334 
  Verifying        : nodejs-1:10.24.0-1.module+el8.3.0+10166+b07ac28e.x86_64                                                        258/334 
  Verifying        : npm-1:6.14.11-1.10.24.0.1.module+el8.3.0+10166+b07ac28e.x86_64                                                 259/334 
  Verifying        : python3-PyMySQL-0.10.1-2.module+el8.4.0+9657+a4b6a102.noarch                                                   260/334 
  Verifying        : mod_http2-1.15.7-3.module+el8.4.0+8625+d397f3da.x86_64                                                         261/334 
  Verifying        : openblas-0.3.12-1.el8.x86_64                                                                                   262/334 
  Verifying        : resteasy-3.0.26-6.module+el8.4.0+8891+bb8828ef.noarch                                                          263/334 
  Verifying        : openblas-threads-0.3.12-1.el8.x86_64                                                                           264/334 
  Verifying        : python-srpm-macros-3-41.el8.noarch                                                                             265/334 
  Verifying        : uuid-1.6.2-43.el8.x86_64                                                                                       266/334 
  Verifying        : python-rpm-macros-3-41.el8.noarch                                                                              267/334 
  Verifying        : python3-rpm-macros-3-41.el8.noarch                                                                             268/334 
  Verifying        : python3-pyOpenSSL-19.0.0-1.el8.noarch                                                                          269/334 
  Verifying        : libpq-13.3-1.el8_4.x86_64                                                                                      270/334 
  Verifying        : python3-scipy-1.0.0-21.module+el8.5.0+10916+41bd434d.x86_64                                                    271/334 
  Verifying        : pki-servlet-4.0-api-1:9.0.30-3.module+el8.5.0+11388+9e95fe00.noarch                                            272/334 
  Verifying        : liblognorm-2.0.5-2.el8.x86_64                                                                                  273/334 
  Verifying        : rsyslog-mmnormalize-8.2102.0-5.el8.x86_64                                                                      274/334 
  Verifying        : texlive-base-7:20180414-23.el8.noarch                                                                          275/334 
  Verifying        : platform-python-devel-3.6.8-41.el8.x86_64                                                                      276/334 
  Verifying        : texlive-tetex-7:20180414-23.el8.noarch                                                                         277/334 
  Verifying        : python3-tkinter-3.6.8-41.el8.x86_64                                                                            278/334 
  Verifying        : python3-numpy-1:1.14.3-10.el8.x86_64                                                                           279/334 
  Verifying        : graphviz-2.40.1-43.el8.x86_64                                                                                  280/334 
  Verifying        : python3-rpm-generators-5-7.el8.noarch                                                                          281/334 
  Verifying        : texlive-dvipng-7:20180414-23.el8.x86_64                                                                        282/334 
  Verifying        : python3-babel-2.5.1-7.el8.noarch                                                                               283/334 
  Verifying        : perl-open-1.11-420.el8.noarch                                                                                  284/334 
  Verifying        : python3-jinja2-2.10.1-3.el8.noarch                                                                             285/334 
  Verifying        : rsyslog-elasticsearch-8.2102.0-5.el8.x86_64                                                                    286/334 
  Verifying        : apr-1.6.3-12.el8.x86_64                                                                                        287/334 
  Verifying        : python3-pillow-5.1.1-16.el8.x86_64                                                                             288/334 
  Verifying        : python3-numpy-f2py-1:1.14.3-10.el8.x86_64                                                                      289/334 
  Verifying        : rhel-system-roles-1.7.3-2.el8.noarch                                                                           290/334 
  Verifying        : texlive-texlive.infra-7:20180414-23.el8.noarch                                                                 291/334 
  Verifying        : texlive-lib-7:20180414-23.el8.x86_64                                                                           292/334 
  Verifying        : rsyslog-mmjsonparse-8.2102.0-5.el8.x86_64                                                                      293/334 
  Verifying        : texlive-kpathsea-7:20180414-23.el8.x86_64                                                                      294/334 
  Verifying        : grafana-pcp-3.1.0-1.el8.x86_64                                                                                 295/334 
  Verifying        : postgresql-contrib-12.9-1.module+el8.5.0+13373+4554acc4.x86_64                                                 296/334 
  Verifying        : postgresql-server-12.9-1.module+el8.5.0+13373+4554acc4.x86_64                                                  297/334 
  Verifying        : postgresql-12.9-1.module+el8.5.0+13373+4554acc4.x86_64                                                         298/334 
  Verifying        : grafana-7.5.9-5.el8_5.x86_64                                                                                   299/334 
  Verifying        : java-11-openjdk-headless-1:11.0.14.0.9-2.el8_5.x86_64                                                          300/334 
  Verifying        : httpd-tools-2.4.37-43.module+el8.5.0+13806+b30d9eec.1.x86_64                                                   301/334 
  Verifying        : httpd-filesystem-2.4.37-43.module+el8.5.0+13806+b30d9eec.1.noarch                                              302/334 
  Verifying        : httpd-2.4.37-43.module+el8.5.0+13806+b30d9eec.1.x86_64                                                         303/334 
  Verifying        : mod_ssl-1:2.4.37-43.module+el8.5.0+13806+b30d9eec.1.x86_64                                                     304/334 
  Verifying        : libaec-1.0.2-3.el8.x86_64                                                                                      305/334 
  Verifying        : libqhull-2015.2-5.el8.x86_64                                                                                   306/334 
  Verifying        : apache-commons-compress-1.18-1.module+el8+2598+06babf2e.noarch                                                 307/334 
  Verifying        : python3-mock-2.0.0-11.el8.noarch                                                                               308/334 
  Verifying        : aopalliance-1.0-17.module+el8+2598+06babf2e.noarch                                                             309/334 
  Verifying        : apache-commons-jxpath-1.3-29.module+el8+2598+06babf2e.noarch                                                   310/334 
  Verifying        : apache-commons-lang-2.6-21.module+el8+2598+06babf2e.noarch                                                     311/334 
  Verifying        : apache-commons-codec-1.11-3.module+el8+2598+06babf2e.noarch                                                    312/334 
  Verifying        : httpcomponents-client-4.5.5-4.module+el8+2598+06babf2e.noarch                                                  313/334 
  Verifying        : httpcomponents-core-4.4.10-3.module+el8+2598+06babf2e.noarch                                                   314/334 
  Verifying        : slf4j-1.7.25-4.module+el8+2598+06babf2e.noarch                                                                 315/334 
  Verifying        : slf4j-jdk14-1.7.25-4.module+el8+2598+06babf2e.noarch                                                           316/334 
  Verifying        : jcl-over-slf4j-1.7.25-4.module+el8+2598+06babf2e.noarch                                                        317/334 
  Verifying        : apache-commons-collections-3.2.2-10.module+el8+2598+06babf2e.noarch                                            318/334 
  Verifying        : apache-commons-io-1:2.6-3.module+el8+2598+06babf2e.noarch                                                      319/334 
  Verifying        : apache-commons-logging-1.2-13.module+el8+2598+06babf2e.noarch                                                  320/334 
  Verifying        : python3-httplib2-0.10.3-4.el8.noarch                                                                           321/334 
  Verifying        : tcl-1:8.6.8-2.el8.x86_64                                                                                       322/334 
  Verifying        : fuse3-libs-3.2.1-12.el8.x86_64                                                                                 323/334 
  Verifying        : python3-cffi-1.11.5-5.el8.x86_64                                                                               324/334 
  Verifying        : python3-pycparser-2.14-14.el8.noarch                                                                           325/334 
  Verifying        : libcgroup-tools-0.41-19.el8.x86_64                                                                             326/334 
  Verifying        : publicsuffix-list-20180723-1.el8.noarch                                                                        327/334 
  Verifying        : python3-dns-1.15.0-10.el8.noarch                                                                               328/334 
  Verifying        : librabbitmq-0.9.0-3.el8.x86_64                                                                                 329/334 
  Verifying        : redhat-logos-httpd-84.5-1.el8.noarch                                                                           330/334 
  Verifying        : python3-cryptography-3.2.1-5.el8.x86_64                                                                        331/334 
  Verifying        : libgfortran-8.5.0-4.el8_5.x86_64                                                                               332/334 
  Verifying        : libquadmath-8.5.0-4.el8_5.x86_64                                                                               333/334 
  Verifying        : python3-dnf-plugin-versionlock-4.0.21-4.el8_5.noarch                                                           334/334 
Installed products updated.

Installed:
  ansible-2.9.27-2.el8.noarch                                                                                                               
  ansible-runner-service-1.0.7-1.el8.noarch                                                                                                 
  aopalliance-1.0-17.module+el8+2598+06babf2e.noarch                                                                                        
  apache-commons-codec-1.11-3.module+el8+2598+06babf2e.noarch                                                                               
  apache-commons-collections-3.2.2-10.module+el8+2598+06babf2e.noarch                                                                       
  apache-commons-compress-1.18-1.module+el8+2598+06babf2e.noarch                                                                            
  apache-commons-configuration-1.10-1.el8.noarch                                                                                            
  apache-commons-io-1:2.6-3.module+el8+2598+06babf2e.noarch                                                                                 
  apache-commons-jxpath-1.3-29.module+el8+2598+06babf2e.noarch                                                                              
  apache-commons-lang-2.6-21.module+el8+2598+06babf2e.noarch                                                                                
  apache-commons-logging-1.2-13.module+el8+2598+06babf2e.noarch                                                                             
  apache-sshd-2.6.0-2.el8.noarch                                                                                                            
  apr-1.6.3-12.el8.x86_64                                                                                                                   
  apr-util-1.6.1-6.el8.x86_64                                                                                                               
  apr-util-bdb-1.6.1-6.el8.x86_64                                                                                                           
  apr-util-openssl-1.6.1-6.el8.x86_64                                                                                                       
  bea-stax-api-1.2.0-16.module+el8+2468+c564cec5.noarch                                                                                     
  blosc-1.17.0-1.el8.x86_64                                                                                                                 
  ceph-common-2:16.2.7-1.el8s.x86_64                                                                                                        
  collectd-5.12.0-7.el8s.x86_64                                                                                                             
  collectd-disk-5.12.0-7.el8s.x86_64                                                                                                        
  collectd-postgresql-5.12.0-7.el8s.x86_64                                                                                                  
  collectd-write_http-5.12.0-7.el8s.x86_64                                                                                                  
  collectd-write_syslog-5.12.0-7.el8s.x86_64                                                                                                
  ebay-cors-filter-1.0.1-4.el8.noarch                                                                                                       
  ed25519-java-0.3.0-1.el8.noarch                                                                                                           
  fuse3-libs-3.2.1-12.el8.x86_64                                                                                                            
  git-core-2.27.0-1.el8.x86_64                                                                                                              
  glassfish-fastinfoset-1.2.13-9.module+el8+2468+c564cec5.noarch                                                                            
  glassfish-jaxb-api-2.2.12-8.module+el8+2468+c564cec5.noarch                                                                               
  glassfish-jaxb-core-2.2.11-11.module+el8+2468+c564cec5.noarch                                                                             
  glassfish-jaxb-runtime-2.2.11-11.module+el8+2468+c564cec5.noarch                                                                          
  glassfish-jaxb-txw2-2.2.11-11.module+el8+2468+c564cec5.noarch                                                                             
  gperftools-libs-2.9.1-1.el8s.x86_64                                                                                                       
  grafana-7.5.9-5.el8_5.x86_64                                                                                                              
  grafana-pcp-3.1.0-1.el8.x86_64                                                                                                            
  graphviz-2.40.1-43.el8.x86_64                                                                                                             
  hdf5-1.10.5-5.el8.x86_64                                                                                                                  
  httpcomponents-client-4.5.5-4.module+el8+2598+06babf2e.noarch                                                                             
  httpcomponents-core-4.4.10-3.module+el8+2598+06babf2e.noarch                                                                              
  httpd-2.4.37-43.module+el8.5.0+13806+b30d9eec.1.x86_64                                                                                    
  httpd-filesystem-2.4.37-43.module+el8.5.0+13806+b30d9eec.1.noarch                                                                         
  httpd-tools-2.4.37-43.module+el8.5.0+13806+b30d9eec.1.x86_64                                                                              
  istack-commons-runtime-2.21-9.el8+7.noarch                                                                                                
  jackson-annotations-2.10.0-1.module+el8.2.0+5059+3eb3af25.noarch                                                                          
  jackson-core-2.10.0-1.module+el8.2.0+5059+3eb3af25.noarch                                                                                 
  jackson-databind-2.10.0-1.module+el8.2.0+5059+3eb3af25.noarch                                                                             
  jackson-jaxrs-json-provider-2.9.9-1.module+el8.1.0+3832+9784644d.noarch                                                                   
  jackson-jaxrs-providers-2.9.9-1.module+el8.1.0+3832+9784644d.noarch                                                                       
  jackson-module-jaxb-annotations-2.7.6-4.module+el8+2468+c564cec5.noarch                                                                   
  java-11-openjdk-headless-1:11.0.14.0.9-2.el8_5.x86_64                                                                                     
  java-client-kubevirt-0.5.0-1.el8.noarch                                                                                                   
  jboss-annotations-1.2-api-1.0.0-4.el8.noarch                                                                                              
  jboss-jaxrs-2.0-api-1.0.0-6.el8.noarch                                                                                                    
  jboss-logging-3.3.0-5.el8.noarch                                                                                                          
  jboss-logging-tools-2.0.1-6.el8.noarch                                                                                                    
  jcl-over-slf4j-1.7.25-4.module+el8+2598+06babf2e.noarch                                                                                   
  jdeparser-2.0.0-5.el8.noarch                                                                                                              
  leveldb-1.20-1.el8s.x86_64                                                                                                                
  libXaw-1.0.13-10.el8.x86_64                                                                                                               
  libaec-1.0.2-3.el8.x86_64                                                                                                                 
  libcephfs2-2:16.2.7-1.el8s.x86_64                                                                                                         
  libcgroup-tools-0.41-19.el8.x86_64                                                                                                        
  libgfortran-8.5.0-4.el8_5.x86_64                                                                                                          
  liblognorm-2.0.5-2.el8.x86_64                                                                                                             
  liboath-2.6.2-4.el8s.x86_64                                                                                                               
  libpq-13.3-1.el8_4.x86_64                                                                                                                 
  libqhull-2015.2-5.el8.x86_64                                                                                                              
  libquadmath-8.5.0-4.el8_5.x86_64                                                                                                          
  librabbitmq-0.9.0-3.el8.x86_64                                                                                                            
  libradosstriper1-2:16.2.7-1.el8s.x86_64                                                                                                   
  librdkafka-0.11.4-1.el8.x86_64                                                                                                            
  librgw2-2:16.2.7-1.el8s.x86_64                                                                                                            
  libsodium-1.0.18-2.el8.x86_64                                                                                                             
  libunwind-1.4.0-5.el8s.x86_64                                                                                                             
  mod_http2-1.15.7-3.module+el8.4.0+8625+d397f3da.x86_64                                                                                    
  mod_ssl-1:2.4.37-43.module+el8.5.0+13806+b30d9eec.1.x86_64                                                                                
  nodejs-1:10.24.0-1.module+el8.3.0+10166+b07ac28e.x86_64                                                                                   
  nodejs-full-i18n-1:10.24.0-1.module+el8.3.0+10166+b07ac28e.x86_64                                                                         
  novnc-1.1.0-6.el8.noarch                                                                                                                  
  npm-1:6.14.11-1.10.24.0.1.module+el8.3.0+10166+b07ac28e.x86_64                                                                            
  ongres-scram-1.0.0~beta.2-5.el8.noarch                                                                                                    
  ongres-scram-client-1.0.0~beta.2-5.el8.noarch                                                                                             
  openblas-0.3.12-1.el8.x86_64                                                                                                              
  openblas-threads-0.3.12-1.el8.x86_64                                                                                                      
  openstack-java-cinder-client-3.2.9-9.el8.noarch                                                                                           
  openstack-java-cinder-model-3.2.9-9.el8.noarch                                                                                            
  openstack-java-client-3.2.9-9.el8.noarch                                                                                                  
  openstack-java-glance-client-3.2.9-9.el8.noarch                                                                                           
  openstack-java-glance-model-3.2.9-9.el8.noarch                                                                                            
  openstack-java-keystone-client-3.2.9-9.el8.noarch                                                                                         
  openstack-java-keystone-model-3.2.9-9.el8.noarch                                                                                          
  openstack-java-quantum-client-3.2.9-9.el8.noarch                                                                                          
  openstack-java-quantum-model-3.2.9-9.el8.noarch                                                                                           
  openstack-java-resteasy-connector-3.2.9-9.el8.noarch                                                                                      
  openvswitch-selinux-extra-policy-1.0-28.el8.noarch                                                                                        
  openvswitch2.11-2.11.3-90.el8s.x86_64                                                                                                     
  otopi-common-1.9.6-1.el8.noarch                                                                                                           
  ovirt-ansible-collection-1.6.6-1.el8.noarch                                                                                               
  ovirt-cockpit-sso-0.1.4-2.el8.noarch                                                                                                      
  ovirt-dependencies-4.4.2-1.el8.noarch                                                                                                     
  ovirt-engine-4.4.10.6-1.el8.noarch                                                                                                        
  ovirt-engine-backend-4.4.10.6-1.el8.noarch                                                                                                
  ovirt-engine-dbscripts-4.4.10.6-1.el8.noarch                                                                                              
  ovirt-engine-dwh-4.4.10-1.el8.noarch                                                                                                      
  ovirt-engine-dwh-grafana-integration-setup-4.4.10-1.el8.noarch                                                                            
  ovirt-engine-dwh-setup-4.4.10-1.el8.noarch                                                                                                
  ovirt-engine-extension-aaa-jdbc-1.2.0-1.el8.noarch                                                                                        
  ovirt-engine-extensions-api-1.0.1-1.el8.noarch                                                                                            
  ovirt-engine-metrics-1.4.4-1.el8.noarch                                                                                                   
  ovirt-engine-restapi-4.4.10.6-1.el8.noarch                                                                                                
  ovirt-engine-setup-4.4.10.6-1.el8.noarch                                                                                                  
  ovirt-engine-setup-base-4.4.10.6-1.el8.noarch                                                                                             
  ovirt-engine-setup-plugin-cinderlib-4.4.10.6-1.el8.noarch                                                                                 
  ovirt-engine-setup-plugin-imageio-4.4.10.6-1.el8.noarch                                                                                   
  ovirt-engine-setup-plugin-ovirt-engine-4.4.10.6-1.el8.noarch                                                                              
  ovirt-engine-setup-plugin-ovirt-engine-common-4.4.10.6-1.el8.noarch                                                                       
  ovirt-engine-setup-plugin-vmconsole-proxy-helper-4.4.10.6-1.el8.noarch                                                                    
  ovirt-engine-setup-plugin-websocket-proxy-4.4.10.6-1.el8.noarch                                                                           
  ovirt-engine-tools-4.4.10.6-1.el8.noarch                                                                                                  
  ovirt-engine-tools-backup-4.4.10.6-1.el8.noarch                                                                                           
  ovirt-engine-ui-extensions-1.2.7-1.el8.noarch                                                                                             
  ovirt-engine-vmconsole-proxy-helper-4.4.10.6-1.el8.noarch                                                                                 
  ovirt-engine-webadmin-portal-4.4.10.6-1.el8.noarch                                                                                        
  ovirt-engine-websocket-proxy-4.4.10.6-1.el8.noarch                                                                                        
  ovirt-engine-wildfly-23.0.2-1.el8.x86_64                                                                                                  
  ovirt-engine-wildfly-overlay-23.0.2-1.el8.noarch                                                                                          
  ovirt-imageio-common-2.3.0-1.el8.x86_64                                                                                                   
  ovirt-imageio-daemon-2.3.0-1.el8.x86_64                                                                                                   
  ovirt-openvswitch-2.11-1.el8.noarch                                                                                                       
  ovirt-openvswitch-ovn-2.11-1.el8.noarch                                                                                                   
  ovirt-openvswitch-ovn-central-2.11-1.el8.noarch                                                                                           
  ovirt-openvswitch-ovn-common-2.11-1.el8.noarch                                                                                            
  ovirt-provider-ovn-1.2.34-1.el8.noarch                                                                                                    
  ovirt-python-openvswitch-2.11-1.el8.noarch                                                                                                
  ovirt-vmconsole-1.0.9-1.el8.noarch                                                                                                        
  ovirt-vmconsole-proxy-1.0.9-1.el8.noarch                                                                                                  
  ovirt-web-ui-1.7.2-1.el8.noarch                                                                                                           
  ovn2.11-2.11.1-57.el8s.x86_64                                                                                                             
  ovn2.11-central-2.11.1-57.el8s.x86_64                                                                                                     
  perl-Filter-2:1.58-2.el8.x86_64                                                                                                           
  perl-Text-Unidecode-1.30-5.el8.noarch                                                                                                     
  perl-XML-Parser-2.44-11.el8.x86_64                                                                                                        
  perl-XML-XPath-1.42-3.el8.noarch                                                                                                          
  perl-encoding-4:2.22-3.el8.x86_64                                                                                                         
  perl-open-1.11-420.el8.noarch                                                                                                             
  pki-servlet-4.0-api-1:9.0.30-3.module+el8.5.0+11388+9e95fe00.noarch                                                                       
  platform-python-devel-3.6.8-41.el8.x86_64                                                                                                 
  postgresql-12.9-1.module+el8.5.0+13373+4554acc4.x86_64                                                                                    
  postgresql-contrib-12.9-1.module+el8.5.0+13373+4554acc4.x86_64                                                                            
  postgresql-jdbc-42.2.3-3.el8_2.noarch                                                                                                     
  postgresql-server-12.9-1.module+el8.5.0+13373+4554acc4.x86_64                                                                             
  publicsuffix-list-20180723-1.el8.noarch                                                                                                   
  python-oslo-concurrency-lang-4.3.1-1.el8.noarch                                                                                           
  python-oslo-db-lang-8.4.1-1.el8.noarch                                                                                                    
  python-oslo-i18n-lang-5.0.1-2.el8.noarch                                                                                                  
  python-oslo-log-lang-4.4.0-2.el8.noarch                                                                                                   
  python-oslo-middleware-lang-4.1.1-2.el8.noarch                                                                                            
  python-oslo-privsep-lang-2.4.0-2.el8.noarch                                                                                               
  python-oslo-utils-lang-4.6.0-2.el8.noarch                                                                                                 
  python-oslo-versionedobjects-lang-2.3.0-2.el8.noarch                                                                                      
  python-rpm-macros-3-41.el8.noarch                                                                                                         
  python-srpm-macros-3-41.el8.noarch                                                                                                        
  python3-Bottleneck-1.2.1-13.el8.x86_64                                                                                                    
  python3-PyMySQL-0.10.1-2.module+el8.4.0+9657+a4b6a102.noarch                                                                              
  python3-alembic-1.4.2-5.el8.noarch                                                                                                        
  python3-amqp-2.6.1-1.el8.noarch                                                                                                           
  python3-aniso8601-8.0.0-1.el8.noarch                                                                                                      
  python3-ansible-runner-1.4.6-1.el8.noarch                                                                                                 
  python3-automaton-2.2.0-1.el8.noarch                                                                                                      
  python3-babel-2.5.1-7.el8.noarch                                                                                                          
  python3-bcrypt-3.1.7-3.el8.x86_64                                                                                                         
  python3-cachetools-4.1.1-2.el8.noarch                                                                                                     
  python3-ceph-argparse-2:16.2.7-1.el8s.x86_64                                                                                              
  python3-ceph-common-2:16.2.7-1.el8s.x86_64                                                                                                
  python3-cephfs-2:16.2.7-1.el8s.x86_64                                                                                                     
  python3-cffi-1.11.5-5.el8.x86_64                                                                                                          
  python3-cinder-common-1:17.2.0-1.el8.noarch                                                                                               
  python3-cinderlib-1:3.0.0-1.el8.noarch                                                                                                    
  python3-click-6.7-8.el8.noarch                                                                                                            
  python3-cryptography-3.2.1-5.el8.x86_64                                                                                                   
  python3-cycler-0.10.0-13.el8.noarch                                                                                                       
  python3-daemon-2.2.3-7.el8.noarch                                                                                                         
  python3-debtcollector-1.22.0-2.el8.noarch                                                                                                 
  python3-distro-1.4.0-2.module+el8.1.0+3334+5cb623d7.noarch                                                                                
  python3-dnf-plugin-versionlock-4.0.21-4.el8_5.noarch                                                                                      
  python3-dns-1.15.0-10.el8.noarch                                                                                                          
  python3-docutils-0.14-12.module+el8.1.0+3334+5cb623d7.noarch                                                                              
  python3-editor-1.0.4-4.el8.noarch                                                                                                         
  python3-eventlet-0.25.2-3.1.el8.noarch                                                                                                    
  python3-fasteners-0.14.1-20.el8.noarch                                                                                                    
  python3-flask-1:0.12.2-4.el8.noarch                                                                                                       
  python3-flask-restful-0.3.7-5.el8.noarch                                                                                                  
  python3-fluidity-sm-0.2.0-16.el8.noarch                                                                                                   
  python3-funcsigs-1.0.2-17.el8.noarch                                                                                                      
  python3-futurist-2.3.0-2.el8.noarch                                                                                                       
  python3-greenlet-0.4.13-4.el8.x86_64                                                                                                      
  python3-httplib2-0.10.3-4.el8.noarch                                                                                                      
  python3-importlib-metadata-1.7.0-1.el8.noarch                                                                                             
  python3-invoke-1.4.0-1.el8.noarch                                                                                                         
  python3-iso8601-0.1.12-3.el8.noarch                                                                                                       
  python3-itsdangerous-0.24-14.el8.noarch                                                                                                   
  python3-jinja2-2.10.1-3.el8.noarch                                                                                                        
  python3-jmespath-0.9.0-11.el8.noarch                                                                                                      
  python3-jsonschema-2.6.0-4.el8.noarch                                                                                                     
  python3-kazoo-2.8.0-1.el8.noarch                                                                                                          
  python3-kiwisolver-1.1.0-4.el8.x86_64                                                                                                     
  python3-kombu-1:4.6.11-2.el8.noarch                                                                                                       
  python3-lexicon-1.0.0-9.el8.noarch                                                                                                        
  python3-lockfile-1:0.11.0-16.el8.noarch                                                                                                   
  python3-mako-1.0.6-13.el8.noarch                                                                                                          
  python3-markupsafe-0.23-19.el8.x86_64                                                                                                     
  python3-matplotlib-3.1.1-2.el8.x86_64                                                                                                     
  python3-matplotlib-data-3.1.1-2.el8.noarch                                                                                                
  python3-matplotlib-data-fonts-3.1.1-2.el8.noarch                                                                                          
  python3-matplotlib-tk-3.1.1-2.el8.x86_64                                                                                                  
  python3-migrate-0.13.0-1.el8.noarch                                                                                                       
  python3-mock-2.0.0-11.el8.noarch                                                                                                          
  python3-mod_wsgi-4.6.4-4.el8.x86_64                                                                                                       
  python3-monotonic-1.5-5.el8.noarch                                                                                                        
  python3-msgpack-1.0.0-2.el8.x86_64                                                                                                        
  python3-netaddr-0.7.19-8.el8.noarch                                                                                                       
  python3-netifaces-0.10.6-4.el8.x86_64                                                                                                     
  python3-networkx-2.5-1.el8.noarch                                                                                                         
  python3-notario-0.0.16-4.el8.noarch                                                                                                       
  python3-numexpr-2.7.1-1.el8.x86_64                                                                                                        
  python3-numpy-1:1.14.3-10.el8.x86_64                                                                                                      
  python3-numpy-f2py-1:1.14.3-10.el8.x86_64                                                                                                 
  python3-openvswitch2.11-2.11.3-90.el8s.x86_64                                                                                             
  python3-os-brick-4.0.4-1.el8.noarch                                                                                                       
  python3-os-win-5.2.0-1.el8.noarch                                                                                                         
  python3-oslo-concurrency-4.3.1-1.el8.noarch                                                                                               
  python3-oslo-config-2:8.3.4-1.el8.noarch                                                                                                  
  python3-oslo-context-3.1.2-1.el8.noarch                                                                                                   
  python3-oslo-db-8.4.1-1.el8.noarch                                                                                                        
  python3-oslo-i18n-5.0.1-2.el8.noarch                                                                                                      
  python3-oslo-log-4.4.0-2.el8.noarch                                                                                                       
  python3-oslo-messaging-12.5.2-1.el8.noarch                                                                                                
  python3-oslo-middleware-4.1.1-2.el8.noarch                                                                                                
  python3-oslo-privsep-2.4.0-2.el8.noarch                                                                                                   
  python3-oslo-rootwrap-6.2.0-2.el8.noarch                                                                                                  
  python3-oslo-serialization-4.0.1-2.el8.noarch                                                                                             
  python3-oslo-service-2.4.0-2.el8.noarch                                                                                                   
  python3-oslo-utils-4.6.0-2.el8.noarch                                                                                                     
  python3-oslo-versionedobjects-2.3.0-2.el8.noarch                                                                                          
  python3-otopi-1.9.6-1.el8.noarch                                                                                                          
  python3-ovirt-engine-lib-4.4.10.6-1.el8.noarch                                                                                            
  python3-ovirt-engine-sdk4-4.4.15-1.el8.x86_64                                                                                             
  python3-ovirt-setup-lib-1.3.2-1.el8.noarch                                                                                                
  python3-ovsdbapp-0.17.5-1.el8.noarch                                                                                                      
  python3-packaging-20.4-1.el8.noarch                                                                                                       
  python3-pandas-0.25.3-1.el8.x86_64                                                                                                        
  python3-paramiko-2.7.2-2.el8.noarch                                                                                                       
  python3-passlib-1.7.1-6.el8.noarch                                                                                                        
  python3-paste-3.2.4-1.el8.noarch                                                                                                          
  python3-paste-deploy-2.1.0-3.el8.noarch                                                                                                   
  python3-pbr-5.4.3-2.el8.noarch                                                                                                            
  python3-pillow-5.1.1-16.el8.x86_64                                                                                                        
  python3-prettytable-0.7.2-14.el8.noarch                                                                                                   
  python3-psycopg2-2.7.5-7.el8.x86_64                                                                                                       
  python3-pyOpenSSL-19.0.0-1.el8.noarch                                                                                                     
  python3-pyasn1-0.4.6-3.el8.noarch                                                                                                         
  python3-pycparser-2.14-14.el8.noarch                                                                                                      
  python3-pydot-1.4.1-1.el8.noarch                                                                                                          
  python3-pygraphviz-1.5-9.el8.x86_64                                                                                                       
  python3-pynacl-1.3.0-5.el8.x86_64                                                                                                         
  python3-pyngus-2.3.0-4.el8.noarch                                                                                                         
  python3-qpid-proton-0.36.0-1.el8.x86_64                                                                                                   
  python3-rados-2:16.2.7-1.el8s.x86_64                                                                                                      
  python3-rbd-2:16.2.7-1.el8s.x86_64                                                                                                        
  python3-redis-3.3.8-1.el8.noarch                                                                                                          
  python3-repoze-lru-0.7-6.el8.noarch                                                                                                       
  python3-requests_ntlm-1.1.0-8.el8.noarch                                                                                                  
  python3-rfc3986-1.4.0-3.el8.noarch                                                                                                        
  python3-rgw-2:16.2.7-1.el8s.x86_64                                                                                                        
  python3-routes-2.4.1-12.el8.noarch                                                                                                        
  python3-rpm-generators-5-7.el8.noarch                                                                                                     
  python3-rpm-macros-3-41.el8.noarch                                                                                                        
  python3-scipy-1.0.0-21.module+el8.5.0+10916+41bd434d.x86_64                                                                               
  python3-sqlalchemy-1.3.2-2.module+el8.3.0+6646+6b4b10ec.x86_64                                                                            
  python3-sqlparse-0.3.1-3.el8.noarch                                                                                                       
  python3-statsd-3.2.1-16.el8.noarch                                                                                                        
  python3-stevedore-3.2.2-2.el8.noarch                                                                                                      
  python3-tables-3.5.2-6.el8.x86_64                                                                                                         
  python3-tabulate-0.8.7-4.el8.noarch                                                                                                       
  python3-taskflow-4.5.0-2.el8.noarch                                                                                                       
  python3-tempita-0.5.1-25.el8.noarch                                                                                                       
  python3-tenacity-6.2.0-1.el8.noarch                                                                                                       
  python3-tkinter-3.6.8-41.el8.x86_64                                                                                                       
  python3-tooz-2.7.2-1.el8.noarch                                                                                                           
  python3-vine-1.3.0-4.el8.noarch                                                                                                           
  python3-voluptuous-0.11.7-2.el8.noarch                                                                                                    
  python3-wcwidth-0.1.7-14.el8.noarch                                                                                                       
  python3-webob-1.8.6-3.el8.noarch                                                                                                          
  python3-websocket-client-0.56.0-5.el8.noarch                                                                                              
  python3-websockify-0.8.0-15.el8.noarch                                                                                                    
  python3-werkzeug-0.12.2-4.el8.noarch                                                                                                      
  python3-winrm-0.3.0-7.el8.noarch                                                                                                          
  python3-wrapt-1.11.2-4.el8.x86_64                                                                                                         
  python3-xmltodict-0.12.0-4.el8.noarch                                                                                                     
  python3-yappi-1.2.5-1.el8.x86_64                                                                                                          
  python3-zake-0.2.2-18.el8.noarch                                                                                                          
  python3-zipp-0.5.1-3.el8.noarch                                                                                                           
  python3-zstd-1.4.5.1-1.el8.x86_64                                                                                                         
  qpid-proton-c-0.36.0-1.el8.x86_64                                                                                                         
  redhat-logos-httpd-84.5-1.el8.noarch                                                                                                      
  relaxngDatatype-2011.1-7.module+el8+2468+c564cec5.noarch                                                                                  
  resteasy-3.0.26-6.module+el8.4.0+8891+bb8828ef.noarch                                                                                     
  rhel-system-roles-1.7.3-2.el8.noarch                                                                                                      
  rsyslog-elasticsearch-8.2102.0-5.el8.x86_64                                                                                               
  rsyslog-mmjsonparse-8.2102.0-5.el8.x86_64                                                                                                 
  rsyslog-mmnormalize-8.2102.0-5.el8.x86_64                                                                                                 
  slf4j-1.7.25-4.module+el8+2598+06babf2e.noarch                                                                                            
  slf4j-jdk14-1.7.25-4.module+el8+2598+06babf2e.noarch                                                                                      
  snmp4j-3.6.4-0.1.el8.noarch                                                                                                               
  sshpass-1.06-9.el8.x86_64                                                                                                                 
  stax-ex-1.7.7-8.module+el8+2468+c564cec5.noarch                                                                                           
  sysfsutils-2.1.0-24.el8.x86_64                                                                                                            
  tcl-1:8.6.8-2.el8.x86_64                                                                                                                  
  texlive-base-7:20180414-23.el8.noarch                                                                                                     
  texlive-dvipng-7:20180414-23.el8.x86_64                                                                                                   
  texlive-kpathsea-7:20180414-23.el8.x86_64                                                                                                 
  texlive-lib-7:20180414-23.el8.x86_64                                                                                                      
  texlive-tetex-7:20180414-23.el8.noarch                                                                                                    
  texlive-texlive.infra-7:20180414-23.el8.noarch                                                                                            
  tk-1:8.6.8-1.el8.x86_64                                                                                                                   
  uuid-1.6.2-43.el8.x86_64                                                                                                                  
  vdsm-jsonrpc-java-1.6.0-1.el8.noarch                                                                                                      
  ws-commons-util-1.0.2-1.el8.noarch                                                                                                        
  xmlrpc-client-3.1.3-1.el8.noarch                                                                                                          
  xmlrpc-common-3.1.3-1.el8.noarch                                                                                                          
  xmlstreambuffer-1.5.4-8.module+el8+2468+c564cec5.noarch                                                                                   
  xorg-x11-fonts-ISO8859-1-100dpi-7.5-19.el8.noarch                                                                                         
  xsom-0-19.20110809svn.module+el8+2468+c564cec5.noarch                                                                                     

Complete!
[root@tektutor ~]# engine-setup
[ INFO  ] Stage: Initializing
[ INFO  ] Stage: Environment setup
          Configuration files: /etc/ovirt-engine-setup.conf.d/10-packaging-jboss.conf, /etc/ovirt-engine-setup.conf.d/10-packaging.conf
          Log file: /var/log/ovirt-engine/setup/ovirt-engine-setup-20220212163618-gd7egj.log
          Version: otopi-1.9.6 (otopi-1.9.6-1.el8)
[ INFO  ] Stage: Environment packages setup
[ INFO  ] Stage: Programs detection
[ INFO  ] Stage: Environment setup (late)
[ INFO  ] Stage: Environment customization
         
          --== PRODUCT OPTIONS ==--
         
          Configure Cinderlib integration (Currently in tech preview) (Yes, No) [No]: No
          Configure Engine on this host (Yes, No) [Yes]: Yes
         
          Configuring ovirt-provider-ovn also sets the Default cluster's default network provider to ovirt-provider-ovn.
          Non-Default clusters may be configured with an OVN after installation.
          Configure ovirt-provider-ovn (Yes, No) [Yes]: Yes
          Configure WebSocket Proxy on this host (Yes, No) [Yes]: Yes
         
          * Please note * : Data Warehouse is required for the engine.
          If you choose to not configure it on this host, you have to configure
          it on a remote host, and then configure the engine on this host so
          that it can access the database of the remote Data Warehouse host.
          Configure Data Warehouse on this host (Yes, No) [Yes]: Yes
          Configure VM Console Proxy on this host (Yes, No) [Yes]: Yes
          Configure Grafana on this host (Yes, No) [Yes]: Yes
         
          --== PACKAGES ==--
         
[ INFO  ] Checking for product updates...
[ INFO  ] DNF Package grafana-postgres available, but not installed.
[ INFO  ] No product updates found
         
          --== NETWORK CONFIGURATION ==--
         
          Host fully qualified DNS name of this server [tektutor.okd.org]: tektutor.okd.org
[WARNING] Failed to resolve tektutor.okd.org using DNS, it can be resolved only locally
         
          Setup can automatically configure the firewall on this system.
          Note: automatic configuration of the firewall may overwrite current settings.
          Do you want Setup to configure the firewall? (Yes, No) [Yes]: Yes
[ INFO  ] firewalld will be configured as firewall manager.
         
          --== DATABASE CONFIGURATION ==--
         
          Where is the DWH database located? (Local, Remote) [Local]: Local
         
          Setup can configure the local postgresql server automatically for the DWH to run. This may conflict with existing applications.
          Would you like Setup to automatically configure postgresql and create DWH database, or prefer to perform that manually? (Automatic, Manual) [Automatic]: Automatic
          Where is the Engine database located? (Local, Remote) [Local]: Local
         
          Setup can configure the local postgresql server automatically for the engine to run. This may conflict with existing applications.
          Would you like Setup to automatically configure postgresql and create Engine database, or prefer to perform that manually? (Automatic, Manual) [Automatic]: Automatic
         
          --== OVIRT ENGINE CONFIGURATION ==--
         
          Engine admin password: 
          Confirm engine admin password: 
[WARNING] Password is weak: The password is shorter than 8 characters
          Use weak password? (Yes, No) [No]: Yes
          Application mode (Virt, Gluster, Both) [Both]: Both
          Use default credentials (admin@internal) for ovirt-provider-ovn (Yes, No) [Yes]: Yes
         
          --== STORAGE CONFIGURATION ==--
         
          Default SAN wipe after delete (Yes, No) [No]: No
         
          --== PKI CONFIGURATION ==--
         
          Organization name for certificate [okd.org]: tektutor
         
          --== APACHE CONFIGURATION ==--
         
          Setup can configure the default page of the web server to present the application home page. This may conflict with existing applications.
          Do you wish to set the application as the default page of the web server? (Yes, No) [Yes]: Yes
         
          Setup can configure apache to use SSL using a certificate issued from the internal CA.
          Do you wish Setup to configure that, or prefer to perform that manually? (Automatic, Manual) [Automatic]: Automatic
         
          --== SYSTEM CONFIGURATION ==--
         
         
          --== MISC CONFIGURATION ==--
         
          Please choose Data Warehouse sampling scale:
          (1) Basic
          (2) Full
          (1, 2)[1]: 1
          Use Engine admin password as initial Grafana admin password (Yes, No) [Yes]: Yes
         
          --== END OF CONFIGURATION ==--
         
[ INFO  ] Stage: Setup validation
         
          --== CONFIGURATION PREVIEW ==--
         
          Application mode                        : both
          Default SAN wipe after delete           : False
          Host FQDN                               : tektutor.okd.org
          Firewall manager                        : firewalld
          Update Firewall                         : True
          Set up Cinderlib integration            : False
          Configure local Engine database         : True
          Set application as default page         : True
          Configure Apache SSL                    : True
          Engine database host                    : localhost
          Engine database port                    : 5432
          Engine database secured connection      : False
          Engine database host name validation    : False
          Engine database name                    : engine
          Engine database user name               : engine
          Engine installation                     : True
          PKI organization                        : tektutor
          Set up ovirt-provider-ovn               : True
          Grafana integration                     : True
          Grafana database user name              : ovirt_engine_history_grafana
          Configure WebSocket Proxy               : True
          DWH installation                        : True
          DWH database host                       : localhost
          DWH database port                       : 5432
          DWH database secured connection         : False
          DWH database host name validation       : False
          DWH database name                       : ovirt_engine_history
          Configure local DWH database            : True
          Configure VMConsole Proxy               : True
         
          Please confirm installation settings (OK, Cancel) [OK]: OK
[ INFO  ] Stage: Transaction setup
[ INFO  ] Stopping engine service
[ INFO  ] Stopping ovirt-fence-kdump-listener service
[ INFO  ] Stopping dwh service
[ INFO  ] Stopping vmconsole-proxy service
[ INFO  ] Stopping websocket-proxy service
[ INFO  ] Stage: Misc configuration (early)
[ INFO  ] Stage: Package installation
[ INFO  ] Stage: Misc configuration
[ INFO  ] Upgrading CA
[ INFO  ] Initializing PostgreSQL
[ INFO  ] Creating PostgreSQL 'engine' database
[ INFO  ] Configuring PostgreSQL
[ INFO  ] Creating PostgreSQL 'ovirt_engine_history' database
[ INFO  ] Configuring PostgreSQL
[ INFO  ] Creating CA: /etc/pki/ovirt-engine/ca.pem
[ INFO  ] Creating CA: /etc/pki/ovirt-engine/qemu-ca.pem
[ INFO  ] Updating OVN SSL configuration
[ INFO  ] Updating OVN timeout configuration
[ INFO  ] Creating/refreshing DWH database schema
[ INFO  ] Setting up ovirt-vmconsole proxy helper PKI artifacts
[ INFO  ] Setting up ovirt-vmconsole SSH PKI artifacts
[ INFO  ] Configuring WebSocket Proxy
[ INFO  ] Creating/refreshing Engine database schema
[ INFO  ] Creating a user for Grafana
[ INFO  ] Creating/refreshing Engine 'internal' domain database schema
[ INFO  ] Creating default mac pool range
[ INFO  ] Adding default OVN provider to database
[ INFO  ] Adding OVN provider secret to database
[ INFO  ] Setting a password for internal user admin
[ INFO  ] Install selinux module /usr/share/ovirt-engine/selinux/ansible-runner-service.cil
[ INFO  ] Generating post install configuration file '/etc/ovirt-engine-setup.conf.d/20-setup-ovirt-post.conf'
[ INFO  ] Stage: Transaction commit
[ INFO  ] Stage: Closing up
[ INFO  ] Starting engine service
[ INFO  ] Starting dwh service
[ INFO  ] Starting Grafana service
[ INFO  ] Restarting ovirt-vmconsole proxy service
         
          --== SUMMARY ==--
         
[ INFO  ] Restarting httpd
          Please use the user 'admin@internal' and password specified in order to login
          Web access is enabled at:
              http://tektutor.okd.org:80/ovirt-engine
              https://tektutor.okd.org:443/ovirt-engine
          Internal CA B4:FE:1E:3C:2D:50:09:92:F5:BB:49:72:C2:85:B1:78:3A:8A:59:2A
          SSH fingerprint: SHA256:gRExX4WTsSrsG3as0PNQpANEmFMkcYsUCcYx+PFdFng
          Web access for grafana is enabled at:
              https://tektutor.okd.org/ovirt-engine-grafana/
          Please run the following command on the engine machine tektutor.okd.org, for SSO to work:
          systemctl restart ovirt-engine
         
          --== END OF SUMMARY ==--
         
[ INFO  ] Stage: Clean up
          Log file is located at /var/log/ovirt-engine/setup/ovirt-engine-setup-20220212163618-gd7egj.log
[ INFO  ] Generating answer file '/var/lib/ovirt-engine/setup/answers/20220212164609-setup.conf'
[ INFO  ] Stage: Pre-termination
[ INFO  ] Stage: Termination
[ INFO  ] Execution of setup completed successfully
</pre>

### Configuring Ovirt
```
sudo engine-setup
```

The expected output is
<pre>
[root@tektutor ~]# <b>engine-setup</b>
[ INFO  ] Stage: Initializing
[ INFO  ] Stage: Environment setup
          Configuration files: /etc/ovirt-engine-setup.conf.d/10-packaging-jboss.conf, /etc/ovirt-engine-setup.conf.d/10-packaging.conf
          Log file: /var/log/ovirt-engine/setup/ovirt-engine-setup-20220212163618-gd7egj.log
          Version: otopi-1.9.6 (otopi-1.9.6-1.el8)
[ INFO  ] Stage: Environment packages setup
[ INFO  ] Stage: Programs detection
[ INFO  ] Stage: Environment setup (late)
[ INFO  ] Stage: Environment customization
         
          --== PRODUCT OPTIONS ==--
         
          Configure Cinderlib integration (Currently in tech preview) (Yes, No) [No]: No
          Configure Engine on this host (Yes, No) [Yes]: Yes
         
          Configuring ovirt-provider-ovn also sets the Default cluster's default network provider to ovirt-provider-ovn.
          Non-Default clusters may be configured with an OVN after installation.
          Configure ovirt-provider-ovn (Yes, No) [Yes]: Yes
          Configure WebSocket Proxy on this host (Yes, No) [Yes]: Yes
         
          * Please note * : Data Warehouse is required for the engine.
          If you choose to not configure it on this host, you have to configure
          it on a remote host, and then configure the engine on this host so
          that it can access the database of the remote Data Warehouse host.
          Configure Data Warehouse on this host (Yes, No) [Yes]: Yes
          Configure VM Console Proxy on this host (Yes, No) [Yes]: Yes
          Configure Grafana on this host (Yes, No) [Yes]: Yes
         
          --== PACKAGES ==--
         
[ INFO  ] Checking for product updates...
[ INFO  ] DNF Package grafana-postgres available, but not installed.
[ INFO  ] No product updates found
         
          --== NETWORK CONFIGURATION ==--
         
          Host fully qualified DNS name of this server [tektutor.okd.org]: tektutor.okd.org
[WARNING] Failed to resolve tektutor.okd.org using DNS, it can be resolved only locally
         
          Setup can automatically configure the firewall on this system.
          Note: automatic configuration of the firewall may overwrite current settings.
          Do you want Setup to configure the firewall? (Yes, No) [Yes]: Yes
[ INFO  ] firewalld will be configured as firewall manager.
         
          --== DATABASE CONFIGURATION ==--
         
          Where is the DWH database located? (Local, Remote) [Local]: Local
         
          Setup can configure the local postgresql server automatically for the DWH to run. This may conflict with existing applications.
          Would you like Setup to automatically configure postgresql and create DWH database, or prefer to perform that manually? (Automatic, Manual) [Automatic]: Automatic
          Where is the Engine database located? (Local, Remote) [Local]: Local
         
          Setup can configure the local postgresql server automatically for the engine to run. This may conflict with existing applications.
          Would you like Setup to automatically configure postgresql and create Engine database, or prefer to perform that manually? (Automatic, Manual) [Automatic]: Automatic
         
          --== OVIRT ENGINE CONFIGURATION ==--
         
          Engine admin password: 
          Confirm engine admin password: 
[WARNING] Password is weak: The password is shorter than 8 characters
          Use weak password? (Yes, No) [No]: Yes
          Application mode (Virt, Gluster, Both) [Both]: Both
          Use default credentials (admin@internal) for ovirt-provider-ovn (Yes, No) [Yes]: Yes
         
          --== STORAGE CONFIGURATION ==--
         
          Default SAN wipe after delete (Yes, No) [No]: No
         
          --== PKI CONFIGURATION ==--
         
          Organization name for certificate [okd.org]: tektutor
         
          --== APACHE CONFIGURATION ==--
         
          Setup can configure the default page of the web server to present the application home page. This may conflict with existing applications.
          Do you wish to set the application as the default page of the web server? (Yes, No) [Yes]: Yes
         
          Setup can configure apache to use SSL using a certificate issued from the internal CA.
          Do you wish Setup to configure that, or prefer to perform that manually? (Automatic, Manual) [Automatic]: Automatic
         
          --== SYSTEM CONFIGURATION ==--
         
         
          --== MISC CONFIGURATION ==--
         
          Please choose Data Warehouse sampling scale:
          (1) Basic
          (2) Full
          (1, 2)[1]: 1
          Use Engine admin password as initial Grafana admin password (Yes, No) [Yes]: Yes
         
          --== END OF CONFIGURATION ==--
         
[ INFO  ] Stage: Setup validation
         
          --== CONFIGURATION PREVIEW ==--
         
          Application mode                        : both
          Default SAN wipe after delete           : False
          Host FQDN                               : tektutor.okd.org
          Firewall manager                        : firewalld
          Update Firewall                         : True
          Set up Cinderlib integration            : False
          Configure local Engine database         : True
          Set application as default page         : True
          Configure Apache SSL                    : True
          Engine database host                    : localhost
          Engine database port                    : 5432
          Engine database secured connection      : False
          Engine database host name validation    : False
          Engine database name                    : engine
          Engine database user name               : engine
          Engine installation                     : True
          PKI organization                        : tektutor
          Set up ovirt-provider-ovn               : True
          Grafana integration                     : True
          Grafana database user name              : ovirt_engine_history_grafana
          Configure WebSocket Proxy               : True
          DWH installation                        : True
          DWH database host                       : localhost
          DWH database port                       : 5432
          DWH database secured connection         : False
          DWH database host name validation       : False
          DWH database name                       : ovirt_engine_history
          Configure local DWH database            : True
          Configure VMConsole Proxy               : True
         
          Please confirm installation settings (OK, Cancel) [OK]: OK
[ INFO  ] Stage: Transaction setup
[ INFO  ] Stopping engine service
[ INFO  ] Stopping ovirt-fence-kdump-listener service
[ INFO  ] Stopping dwh service
[ INFO  ] Stopping vmconsole-proxy service
[ INFO  ] Stopping websocket-proxy service
[ INFO  ] Stage: Misc configuration (early)
[ INFO  ] Stage: Package installation
[ INFO  ] Stage: Misc configuration
[ INFO  ] Upgrading CA
[ INFO  ] Initializing PostgreSQL
[ INFO  ] Creating PostgreSQL 'engine' database
[ INFO  ] Configuring PostgreSQL
[ INFO  ] Creating PostgreSQL 'ovirt_engine_history' database
[ INFO  ] Configuring PostgreSQL
[ INFO  ] Creating CA: /etc/pki/ovirt-engine/ca.pem
[ INFO  ] Creating CA: /etc/pki/ovirt-engine/qemu-ca.pem
[ INFO  ] Updating OVN SSL configuration
[ INFO  ] Updating OVN timeout configuration
[ INFO  ] Creating/refreshing DWH database schema
[ INFO  ] Setting up ovirt-vmconsole proxy helper PKI artifacts
[ INFO  ] Setting up ovirt-vmconsole SSH PKI artifacts
[ INFO  ] Configuring WebSocket Proxy
[ INFO  ] Creating/refreshing Engine database schema
[ INFO  ] Creating a user for Grafana
[ INFO  ] Creating/refreshing Engine 'internal' domain database schema
[ INFO  ] Creating default mac pool range
[ INFO  ] Adding default OVN provider to database
[ INFO  ] Adding OVN provider secret to database
[ INFO  ] Setting a password for internal user admin
[ INFO  ] Install selinux module /usr/share/ovirt-engine/selinux/ansible-runner-service.cil
[ INFO  ] Generating post install configuration file '/etc/ovirt-engine-setup.conf.d/20-setup-ovirt-post.conf'
[ INFO  ] Stage: Transaction commit
[ INFO  ] Stage: Closing up
[ INFO  ] Starting engine service
[ INFO  ] Starting dwh service
[ INFO  ] Starting Grafana service
[ INFO  ] Restarting ovirt-vmconsole proxy service
         
          --== SUMMARY ==--
         
[ INFO  ] Restarting httpd
          Please use the user 'admin@internal' and password specified in order to login
          Web access is enabled at:
              http://tektutor.okd.org:80/ovirt-engine
              https://tektutor.okd.org:443/ovirt-engine
          Internal CA B4:FE:1E:3C:2D:50:09:92:F5:BB:49:72:C2:85:B1:78:3A:8A:59:2A
          SSH fingerprint: SHA256:gRExX4WTsSrsG3as0PNQpANEmFMkcYsUCcYx+PFdFng
          Web access for grafana is enabled at:
              https://tektutor.okd.org/ovirt-engine-grafana/
          Please run the following command on the engine machine tektutor.okd.org, for SSO to work:
          systemctl restart ovirt-engine
         
          --== END OF SUMMARY ==--
         
[ INFO  ] Stage: Clean up
          Log file is located at /var/log/ovirt-engine/setup/ovirt-engine-setup-20220212163618-gd7egj.log
[ INFO  ] Generating answer file '/var/lib/ovirt-engine/setup/answers/20220212164609-setup.conf'
[ INFO  ] Stage: Pre-termination
[ INFO  ] Stage: Termination
[ INFO  ] Execution of setup completed successfully
</pre>
