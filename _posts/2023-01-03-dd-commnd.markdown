---
layout: post
title:  "dd command"
date:   2023-01-03 16:26:00 +0000
categories: linux command line
---
Notes about the `dd` command in Linux.

{% highlight shell %}
dd if=inputfilename.iso of=/dev/sdX
{% endhighlight %}
This command was slow, no progress was displayed and I had no idea if the command was working succesfully or not. Not possible to kill the process once started so I needed to shutdown to stop the process.

{% highlight shell %}
dd if=https://build.funtoo.org/livecd/funtoo-livecd-20220521-2138.iso of=/dev/sdX bs=4k status=progress oflag=sync
{% endhighlight %}
This was from the [Funtoo Install Guide](https://www.funtoo.org/Install/Download_LiveCD). It worked reliably with progress indicator but again very slow, taking a couple of hours to complete.

{% highlight shell %}
dd if=/path/to/image.iso of=/dev/sdX bs=8M status=progress oflag=direct
{% endhighlight %}
From the [Fedora Docs](https://docs.fedoraproject.org/en-US/quick-docs/creating-and-using-a-live-installation-image/index.html) this command worked reliably with progress indicator and quickly too.

Found a comment from [StackExchange](https://unix.stackexchange.com/questions/508701/dd-command-oflag-direct-and-sync-flags) about combining small block size with `oflag=sync` reducing performance and that not using `oflag=direct` can lead to issues too. I think next time I might try with `oflag=direct,sync` and an `8M` block size, lowering that if the result is unsucessful. 
