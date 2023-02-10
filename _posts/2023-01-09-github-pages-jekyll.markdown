---
layout: post
title:  "Github Pages and Jekyll"
date:   2023-01-09 09:00:00 +0000
---
As mentioned [already](https://madtechsupport.com/about) this site is a Github Pages site uses Jekyll. Here are some notes about using containers to develop with Jekyll.

## Jekyll Docker container
The instructions for [building your GitHub Pages site locally](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/testing-your-github-pages-site-locally-with-jekyll) describes installing Ruby, installing Bundler and running `bundle install` locally. All straightforward enough except, I don't want to maintain a local development environment. In my time as a technical manager I would rate the no.1 time sink in a project being the time it takes for a developer to get thier local development environment up and running without errors. For development to be productive the local development environment setup should be quick, repeatable, platform independent and "just work" every single time. One way to achive this is by using containers, so I did a quick search and found the [Official Jekll Docker Image](https://hub.docker.com/r/jekyll/jekyll/). I did need to spend a little bit of time getting the command line options right, but with that now sorted it's now working reliably each and every time. Here are the steps I used to get up and running.

### Creating a new Jekyll site
Assuming you've done the repo setup steps already and are working in the repository root where `LICENSE` or `README.md` is present and want to create a new Jekyll site in this location.

{% highlight shell %}
sudo docker run --rm -e JEKYLL_UID="1001" -e JEKYLL_GID="1001" --volume="$PWD:/srv/jekyll" -it jekyll/jekyll sh -c "chown -R jekyll /usr/gem/ && jekyll new . --force"
{% endhighlight %}
Points to note:
* The Jekyll process in the container needs to write to the local filesystem. If the local filesystem permissions do not allow this an error will appear. In the above command, I'm using the [environment variable option](https://github.com/envygeeks/jekyll-docker/blob/master/README.md#configuration) to set the UID and GID of the Jekyll process to match that of the owner of the directory so no ownership or [permission errors will appear](https://ask.fedoraproject.org/t/docker-error-errno-eacces-only-occurs-for-uid-other-than-1000/30524/6).
* Jekyll will error if the target location for the new project is not empty, in my case the location where I was creating the new Jekyll site already contained a file (LICENSE) so I needed to _force_ Jekyll to write to this location by using the `--force` option. 

### Serve the Jekyll site locally
The latest `jekyll/jekyll` Docker container will be using Ruby version 3.1 and Ruby 3.0 broke some things that [previously worked](https://github.com/envygeeks/jekyll-docker/issues/335) because [Jekyll serve fails on Ruby 3.0 (webrick missing)](https://github.com/github/pages-gem/issues/752). The work around is to run the following command _once_ to edit the `Gemfile` to include `gem "webrick"` (or simple add it directly to the `Gemfile`):
{% highlight shell %}
sudo docker run --rm -e JEKYLL_UID="1001" -e JEKYLL_GID="1001" --volume="$PWD:/srv/jekyll:Z" jekyll/jekyll bundle add webrick`
{% endhighlight %}

Next start the webserver:
{% highlight shell %}
sudo docker run --rm -e JEKYLL_UID="1001" -e JEKYLL_GID="1001" --volume="$PWD:/srv/jekyll:Z" --publish [::1]:4000:4000 jekyll/jekyll jekyll serve`
{% endhighlight %}

Test locally by browsing http://localhost:4000

## Running rootless with podman
Always needing to use `sudo` and requiring the correct `UID` is not ideal, it would be simpler if we could run without needing `sudo` and worry not about needing to export the correct UID environment variable.

The commands to repeat the above process with `podman` are:
{% highlight shell %}
podman run -ti --rm -v .:/srv/jekyll:Z -e JEKYLL_ROOTLESS=1 docker.io/jekyll/jekyll jekyll new . --force
podman run --rm --volume="$PWD:/srv/jekyll:Z" -e JEKYLL_ROOTLESS=1 docker.io/jekyll/jekyll bundle add webrick
podman run -ti --rm --volume="$PWD:/srv/jekyll:Z" --publish [::1]:4000:4000 -e JEKYLL_ROOTLESS=1 docker.io/jekyll/jekyll jekyll serve
{% endhighlight %}
Points to note:
* The `--force` option is only required when files are already present in the new Jekyll project location.
* `:Z` is required for hosts that have SELinux enabled. This is explained in detail here [Podman volumes and SELinux](https://blog.christophersmart.com/2021/01/31/podman-volumes-and-selinux/).
* Running the `bundle add webrick` command is requied only once to update `Gemfile`.

## Save a post without publishing
There is a [predefined global variable](https://jekyllrb.com/docs/front-matter/#predefined-global-variables) called `published` that can be set to false to save a post and keep it unpublished when the site is generated.
