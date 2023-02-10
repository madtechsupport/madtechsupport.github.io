---
layout: post
title:  "WordPress Docker Container"
date:   2023-01-20 09:00:00 +0000
published: false
---
# WordPress Docker container
There's an "DOCKER OFFICIAL IMAGE" maintained by the Docker Community here: https://hub.docker.com/_/wordpress/ along with many other images here: https://hub.docker.com/search?q=wordpress. Let's start by learning all about the official image.

Getting working ~~test~~ email is a long standing open issue: https://github.com/docker-library/wordpress/issues/30. What did puphpet use? (cloned the archived project and did a search) Puphpet used [MailHog](https://github.com/mailhog/MailHog) which could still be a thing according to [Kinsta](https://kinsta.com/blog/mailhog/). Hmmm, I wonder what Kista think of Docker? Other than requiring [Docker for DevKinsta](https://kinsta.com/blog/install-wordpress-locally/) not much it seems. Let's check the Kinsta competition, WPEngine/Flywheel. It seems that [WPEngine has given Docker some thought](https://wpengine.com/resources/containers-clusters-wordpress/). Can't find anything Docker specific for Flywheel.

So, from the (`stack.yml` example)[https://hub.docker.com/_/wordpress] and from looking around at the "Dockerfile"](https://github.com/docker-library/wordpress/blob/master/Dockerfile.template) the WordPress container includes PHP, WordPress and Apache all configured to work together. What we need to bring is a database and some secrets.

This page and resources seems pretty important: https://learn.wordpress.org/lesson-plan/how-to-add-demo-content-in-wordpress/

Content importing / synchronisation could be DB restore or WP import. Maybe the https://distributorplugin.com/ ? Maybe this? [https://wordpress.stackexchange.com/questions/123311/synchronizing-two-wordpress-sites-content](https://wordpress.stackexchange.com/questions/123311/synchronizing-two-wordpress-sites-content). And maybe this [https://en-gb.wordpress.org/plugins/wp-data-sync/](https://en-gb.wordpress.org/plugins/wp-data-sync/).

Also what is this? https://support.gatsbyjs.com/hc/en-us/articles/4410372011027-Installing-Content-Sync-for-WordPress - something to look into later. Ah, seems related to WP being the headless API backend to a Gatsby powered frontend?

Could be interesting: https://www.designlabthemes.com/import-demo-content-wordpress/

This is FakerPress: https://wordpress.org/plugins/fakerpress/ reachable from this page https://learn.wordpress.org/lesson-plan/how-to-add-demo-content-in-wordpress/ which includes links to other content generation tools like [Gutenberg Test Data](https://github.com/Automattic/theme-tools/tree/master/gutenberg-test-data)

The we need to manage secrets as well... It seems they can be stored in a file. Does github have a vault? Maybe, it turns out that Github has a cli now called `gh` and a way to set the secret using `gh secret set` but now was to get the secret??? I think this is because "Environment secrets can be set for use in GitHub Actions". https://cli.github.com/manual/gh_secret. Hmmm. Hashicorp Vault? Maybe Hahsicorp vault is overkill.

# Step 1. Start with a local WordPress site. Let's go straight to getting the Docker container working with a Maria container for the DB. That's probably the faster path towards "getting this working".

``` sh
[warren@madtechsupport wordpress-docker]$ sudo docker pull wordpress
[sudo] password for warren: 
Using default tag: latest
latest: Pulling from library/wordpress
Digest: sha256:ab46a62b3d6f3d5b221234c30836d4407b2a4b98bf7031fb9537eac7eadbb188
Status: Image is up to date for wordpress:latest
docker.io/library/wordpress:latest
[warren@madtechsupport wordpress-docker]$ ls
[warren@madtechsupport wordpress-docker]$ vi stack.yml
[warren@madtechsupport wordpress-docker]$ sudo docker-compose -f stack.yml up
sudo: docker-compose: command not found
[warren@madtechsupport wordpress-docker]$ sudo docker stack deploy -c stack.yml wordpress
Ignoring unsupported options: restart

this node is not a swarm manager. Use "docker swarm init" or "docker swarm join" to connect this node to swarm and try again
[warren@madtechsupport wordpress-docker]$ sudo docker compose -f stack.yml up
[+] Running 12/12
 ⠿ db Pulled                                                                                                                                                                                                                            14.2s
   ⠿ d26998a7c52d Pull complete                                                                                                                                                                                                          3.4s
   ⠿ 4a9d8a3567e3 Pull complete                                                                                                                                                                                                          3.9s
   ⠿ bfee1f0f349e Pull complete                                                                                                                                                                                                          4.1s
   ⠿ 71ff8dfb9b12 Pull complete                                                                                                                                                                                                          4.7s
   ⠿ bf56cbebc916 Pull complete                                                                                                                                                                                                          5.0s
   ⠿ 9a7bdfff0f5b Pull complete                                                                                                                                                                                                          5.2s
   ⠿ b7677d20b791 Pull complete                                                                                                                                                                                                          6.7s
   ⠿ b4bcfc1167c2 Pull complete                                                                                                                                                                                                          7.3s
   ⠿ 03f1c0b71b70 Pull complete                                                                                                                                                                                                         11.8s
   ⠿ 61aa2e026ac3 Pull complete                                                                                                                                                                                                         12.2s
   ⠿ 3b784fa66c50 Pull complete                                                                                                                                                                                                         12.3s
[+] Running 5/5
 ⠿ Network wordpress-docker_default        Created                                                                                                                                                                                       0.5s
 ⠿ Volume "wordpress-docker_wordpress"     Created                                                                                                                                                                                       0.0s
 ⠿ Volume "wordpress-docker_db"            Created                                                                                                                                                                                       0.0s
 ⠿ Container wordpress-docker-wordpress-1  Created                                                                                                                                                                                       0.2s
 ⠿ Container wordpress-docker-db-1         Created                                                                                                                                                                                       0.2s
Attaching to wordpress-docker-db-1, wordpress-docker-wordpress-1
wordpress-docker-db-1         | 2023-01-21 17:05:26+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 5.7.41-1.el7 started.
wordpress-docker-wordpress-1  | WordPress not found in /var/www/html - copying now...
wordpress-docker-wordpress-1  | Complete! WordPress has been successfully copied to /var/www/html
wordpress-docker-wordpress-1  | No 'wp-config.php' found in /var/www/html, but 'WORDPRESS_...' variables supplied; copying 'wp-config-docker.php' (WORDPRESS_DB_HOST WORDPRESS_DB_NAME WORDPRESS_DB_PASSWORD WORDPRESS_DB_USER)
wordpress-docker-wordpress-1  | AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 172.18.0.3. Set the 'ServerName' directive globally to suppress this message
wordpress-docker-wordpress-1  | AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 172.18.0.3. Set the 'ServerName' directive globally to suppress this message
wordpress-docker-wordpress-1  | [Sat Jan 21 17:05:28.188129 2023] [mpm_prefork:notice] [pid 1] AH00163: Apache/2.4.54 (Debian) PHP/8.0.27 configured -- resuming normal operations
wordpress-docker-wordpress-1  | [Sat Jan 21 17:05:28.188162 2023] [core:notice] [pid 1] AH00094: Command line: 'apache2 -D FOREGROUND'
```

So the docker compose didn't work and for a while there the mysql container was out of control... We need to pull this thing apart.

Okay, here's part of the answer:
>The WORDPRESS_DB_NAME needs to already exist on the given MySQL server; it will not be created by the wordpress container.

and we're pulling the mysql container, something we don't want to do, we want the mariadb option instead. Don't re-invent the wheel and just use DDEV?
