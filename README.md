### Installing OpenShift Community Edition (OKD) of OpenShift

As a first step, I installed RHEL 8.5. If you are using it for self-learning purpose, you may request for a Developer license which let's you download RHEL 8.5 and use it free for non-commercial purpose.

#### System Configuration
I used my Dell 

##### Enabling Software Repositories on a fresh RHEL 8.x OS
For more detailed instructions, you may read this article
https://access.redhat.com/solutions/253273

```
subscription-manager register
subscription-manager refresh
subscription-manager attach --auto
subscription-manager list --available --all
```
When prompted for username and password, type your redhat developer credentials to register your OS with RedHat to let you install/upgrade softwares.

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
sudo yum -y install vim tmux ovirt-engine
```
The expected output is
<pre>
[root@tektutor ~]# yum -y install vim tmux ovirt-engine --skip-broken
Updating Subscription Management repositories.
Last metadata expiration check: 0:21:09 ago on Fri 11 Feb 2022 07:46:37 PM PST.
Package vim-enhanced-2:8.0.1763-16.el8_5.4.x86_64 is already installed.
Dependencies resolved.
=========================================================================================================================
 Package             Architecture          Version                    Repository                                    Size
=========================================================================================================================
Installing:
 tmux                x86_64                2.7-1.el8                  rhel-8-for-x86_64-baseos-rpms                317 k

Transaction Summary
=========================================================================================================================
Install  1 Package

Total download size: 317 k
Installed size: 770 k
Downloading Packages:
tmux-2.7-1.el8.x86_64.rpm                                                                520 kB/s | 317 kB     00:00    
-------------------------------------------------------------------------------------------------------------------------
Total                                                                                    518 kB/s | 317 kB     00:00     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                 1/1 
  Installing       : tmux-2.7-1.el8.x86_64                                                                           1/1 
  Running scriptlet: tmux-2.7-1.el8.x86_64                                                                           1/1 
  Verifying        : tmux-2.7-1.el8.x86_64                                                                           1/1 
Installed products updated.

Installed:
  tmux-2.7-1.el8.x86_64                                                                                                  
Skipped:
  ovirt-engine-4.4.10.6-1.el8.noarch                                                                                     

Complete!
</pre>
