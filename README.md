### Installing OpenShift Community Edition (OKD) of OpenShift

As a first step, I installed RHEL 8.5. If you are using it for self-learning purpose, you may request for a Developer license which let's you download RHEL 8.5 and use it free for non-commercial purpose.

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
