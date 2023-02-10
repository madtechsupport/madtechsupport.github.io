---
layout: post
title:  "Gnome authentication dialog asks for the 'administrator' password"
date:   2023-02-10 13:00:00 +0000
---

By using the `usermod -G` command I managed to remove my id from the `wheel` group and not notice straight away. The first I noticed that something was wrong was when the Gnome authentication dialog asked for the "administrator" (or could have been "admin") password. However, no such user exists on my Fedora workstation. How `gnome-shell` goes about looking for and deciding what admin user to ask for is discussed [here](https://bugzilla.gnome.org/show_bug.cgi?id=651547). Because `sudo` [caches the right to elevate](https://askubuntu.com/questions/190311/sudo-credential-caching-on-by-default) I was able to use the `usermod -a -G` command to put my id back into the `wheel` group. If I hadn't noticed my mistage and logged out, rebooted or simply waited too long I would have (temporarily) lost root access to my workstation because no other user id on the workstation was a member of the `wheel` group and the `root` user is not enabled by default. The cache mode of `sudo` made for a quick recovery from this error!
