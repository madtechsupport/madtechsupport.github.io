---
layout: post
title:  "Gnome Boxes"
date:   2023-02-10 13:00:00 +0000
---
I've been using [Gnome Boxes](https://help.gnome.org/users/gnome-boxes/stable/) to create virtual machines for testing purposes. Mainly comparing the results of running different commands without disrupting my workstation. Along the way I've noticed/learned a couple of things about Gnome Boxes that I've documented on this page.
It started with the warning message "Virtualisation extensions are unavailable on your system. Check your BIOS settings to enable them", I'm not sure if I did something to cause this or not, since on a Fedora 37 Workstation default installation things mostly just work:

![Gnome Boxes Review and Create screen warning message](/assets/posts/2023-01-12-gnome-boxes.png)

The first command I learned about for investigating the further was `gnome-boxes --checks`

``` sh
[warren@madtechsupport ~]$ gnome-boxes --checks
(gnome-boxes:7105): Boxes-WARNING **: 11:31:24.256: util-app.vala:442: Failed to execute child process ?virsh? (No such file or directory)
• The CPU is capable of virtualization: yes
• The KVM module is loaded: yes
• Libvirt KVM guest available: no
• Boxes storage pool available: no
    Could not get “gnome-boxes” storage pool information from libvirt. Make sure “virsh -c qemu:///session pool-dumpxml gnome-boxes” is working.
• The SELinux context is default: yes

Report bugs to <http://gitlab.gnome.org/gnome/gnome-boxes/issues>.
Boxes home page: <https://wiki.gnome.org/Apps/Boxes>.
```
The fix for the warning message is kind of contained in the output of `gnome-boxes --checks` and the fix was to work though the installation steps (`sudo dnf install @virtualization`) described in the [Installing virtualisation software](https://docs.fedoraproject.org/en-US/quick-docs/getting-started-with-virtualization/#f37@fedora:install-guide:install/Installing_Using_Anaconda.adoc) documentation. It turns out I was missing some of the required virtualisation packages.

I did also check for hardware virtualisation support using a few commands:
``` sh
[warren@madtechsupport ~]$ egrep '^flags.*(vmx|svm)' /proc/cpuinfo
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc art arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm 3dnowprefetch cpuid_fault invpcid_single pti ssbd ibrs ibpb stibp tpr_shadow vnmi flexpriority ept vpid ept_ad fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid mpx rdseed adx smap clflushopt intel_pt xsaveopt xsavec xgetbv1 xsaves dtherm ida arat pln pts hwp hwp_notify hwp_act_window hwp_epp md_clear flush_l1d arch_capabilities
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc art arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm 3dnowprefetch cpuid_fault invpcid_single pti ssbd ibrs ibpb stibp tpr_shadow vnmi flexpriority ept vpid ept_ad fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid mpx rdseed adx smap clflushopt intel_pt xsaveopt xsavec xgetbv1 xsaves dtherm ida arat pln pts hwp hwp_notify hwp_act_window hwp_epp md_clear flush_l1d arch_capabilities
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc art arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm 3dnowprefetch cpuid_fault invpcid_single pti ssbd ibrs ibpb stibp tpr_shadow vnmi flexpriority ept vpid ept_ad fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid mpx rdseed adx smap clflushopt intel_pt xsaveopt xsavec xgetbv1 xsaves dtherm ida arat pln pts hwp hwp_notify hwp_act_window hwp_epp md_clear flush_l1d arch_capabilities
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc art arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm 3dnowprefetch cpuid_fault invpcid_single pti ssbd ibrs ibpb stibp tpr_shadow vnmi flexpriority ept vpid ept_ad fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid mpx rdseed adx smap clflushopt intel_pt xsaveopt xsavec xgetbv1 xsaves dtherm ida arat pln pts hwp hwp_notify hwp_act_window hwp_epp md_clear flush_l1d arch_capabilities
```

I think I prefer this one (less output!):

``` sh
[warren@madtechsupport ~]$ lscpu|grep Virtual
Virtualization:                  VT-x
```
After installing the virtualisation group packages and checking for CPU virtualisation support `gnome-boxes --checks` now passes all checks:
``` sh
Complete!
[warren@madtechsupport ~]$ gnome-boxes --checks
• The CPU is capable of virtualization: yes
• The KVM module is loaded: yes
• Libvirt KVM guest available: yes
• Boxes storage pool available: yes
• The SELinux context is default: yes

Report bugs to <http://gitlab.gnome.org/gnome/gnome-boxes/issues>.
Boxes home page: <https://wiki.gnome.org/Apps/Boxes>.
```
A side note, `dnf group list --installed` produces short output that doesn't make sense to me. After reading through this [page](https://linuxconfig.org/how-to-work-with-dnf-package-groups) I learned about `dnf group list --hidden` which provided a better picture of what groups are installed:
``` sh
[warren@madtechsupport ~]$ dnf group list --installed
Last metadata expiration check: 0:01:59 ago on Fri 10 Feb 2023 11:24:18 GMT.
Installed Environment Groups:
   Fedora Workstation
Installed Groups:
   Container Management
   LibreOffice
[warren@madtechsupport ~]$ dnf group list --hidden
Last metadata expiration check: 0:02:12 ago on Fri 10 Feb 2023 11:24:18 GMT.
Available Environment Groups:
   Fedora Custom Operating System
   Minimal Install
   Fedora Server Edition
   Fedora Cloud Server
   KDE Plasma Workspaces
   Xfce Desktop
   LXDE Desktop
   LXQt Desktop
   Cinnamon Desktop
   MATE Desktop
   Sugar Desktop Environment
   Deepin Desktop
   Development and Creative Workstation
   Web Server
   Infrastructure Server
   Basic Desktop
   i3 desktop
Installed Environment Groups:
   Fedora Workstation
Installed Groups:
   Anaconda tools
   base-x
   Container Management
   Core
   Firefox Web Browser
   Fonts
   GNOME
   Guest Desktop Agents
   Hardware Support
   LibreOffice
   Multimedia
   Common NetworkManager Submodules
   Printing Support
   Virtualisation
   Fedora Workstation product core
Available Groups:
   3D Printing
   Administration Tools
   Audio Production
   Authoring and Publishing
   Basic Desktop
   Buildsystem building group
   C Development Tools and Libraries
   Cinnamon
   Bootloader tools for Cloud images
   Cloud Infrastructure
   Cloud Management Tools
   Cloud Server Tools
   Compiz
   Critical Path (Applications)
   Critical Path (Base)
   Critical Path (Deepin desktop)
   Critical Path (GNOME)
   Critical Path (KDE)
   Critical Path (LXDE)
   Critical Path (LXQt)
   Critical Path (Server)
   Critical Path (standard)
   Critical Path (Xfce)
   D Development Tools and Libraries
   Deepin Desktop Environment
   Deepin Desktop Applications
   Media packages for Deepin Desktop
   Deepin Desktop Office
   Design Suite
   Development Libraries
   Development Tools
   Dial-up Networking Support
   Directory Server
   DNS Name Server
   Dogtag Certificate System
   Domain Membership
   Editors
   Educational Software
   Electronic Lab
   Engineering and Scientific
   Enlightenment
   Fedora Packager
   Font design and packaging
   FreeIPA Server
   FTP Server
   Games and Entertainment
   Extra games for the GNOME Desktop
   GNOME Software Development
   Graphical Internet
   Graphics
   Guest Agents
   High Availability
   HAProxy
   Haskell
   Headless Management
   i3 window manager
   i3 window manager (supplemental packages)
   Input Methods
   Java
   Java Development
   Java Application Server
   KDE Applications
   KDE
   KDE Educational applications
   KDE Multimedia support
   KDE Office
   KDE Software Development
   KDE Telepathy
   KDE Frameworks 5 Software Development
   Legacy Fonts
   Legacy Network Server
   LibreOffice Development
   Load Balancer
   Applications for the LXDE Desktop
   LXDE
   Multimedia support for LXDE
   LXDE Office
   Applications for the LXQt Desktop
   LXQt
   Translations of LXQt
   Multimedia support for LXQt
   LXQt Office
   Mail Server
   MATE Applications
   MATE
   Milkymist
   MinGW cross-compiler
   MongoDB
   MariaDB (MySQL) Database
   Network Servers
   Neuron Modelling Simulators
   News Server
   OCaml
   Office/Productivity
   Perl Development
   Perl for Web
   A phone/tablet UX environment
   PHP
   VMware Platform Support
   Python Classroom
   Python Science
   Robotics
   RPM Development Tools
   Ruby
   Ruby on Rails
   Security Lab
   Server Configuration Tools
   Hardware Support for Server Systems
   Fedora Server product core
   Windows File Server
   Sound and Video
   PostgreSQL Database
   Standard
   Additional Sugar Activities
   Sugar Desktop Environment
   System Tools
   Text-based Internet
   Tomcat
   Vagrant with libvirt support
   Headless Virtualization
   Basic Web Server
   Window Managers
   Fedora Workstation ostree support
   X Software Development
   Applications for the Xfce Desktop
   Xfce
   Extra plug-ins for the Xfce panel
   Multimedia support for Xfce
   Xfce Office
   Xfce Software Development
   XMonad
   XMonad for MATE
```
# Boxes doesn't need root privileges to run
Boxes pretty much "just works" on Fedora irrespective of the user account type. A non-privileged user (rootless) can start gnome-boxes create a virtual machine, take snapshots, drag and drop files from their desktop to the vm desktop and when they are finished delete the vm. All without once using the `sudo` command. When compared to other virtualisation experinces e.g `minikube` on Fedora that does require root privileges and the use of the `sudo` command I was interested in learning how Boxes can seemingly do the same without needing `sudo`. In the end I asked about this in the [Fedora forum](https://ask.fedoraproject.org/t/on-fedora-i-can-use-gnome-boxes-to-create-vms-as-a-non-root-user-what-makes-this-possible/30716/3) and learned that it's related to architectural choices made by each application.
* [Boxes uses qemu:///session](https://blog.wikichoon.com/2016/01/qemusystem-vs-qemusession.html) where each user has their own VM, at the expense of system wide networking capabilities.
* [Minikube uses qemu:///system](https://minikube.sigs.k8s.io/docs/drivers/qemu/#overview) which provides [system level networking support](https://github.com/kubernetes/minikube/issues/9009#issuecomment-678055949) but the VM will run as the user `qemu`.

# Monolithic (libvirtd) vs Modular (virtqemud)
On my Fedora 37 workstation I went looking for the listening `libvirtd` daemon and wasn't able to find it. This is because the [modular driver daemons](https://libvirt.org/daemons.html#modular-driver-daemons) are now the default in libvirt, replacing the monolithic libvirtd.
