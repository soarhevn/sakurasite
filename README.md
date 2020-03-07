# Sakura Site on Docker

Basic set of containers for an HTTPS site with logging to Papertrail

# Running
The downloaded code should run with no problem as is. 
`$ docker stack deploy  -c ~/docker/sakura/docker-compose.yml sakura`

* We are using Docker [stack](https://docs.docker.com/get-started/part3/) instead of compose
* Volumes `$ docker volume inspect [vol name]`

Alpine shell access to running container
`$ docker exec -it [container_name] /bin/sh`

Use [Attach](https://docs.docker.com/engine/reference/commandline/attach/#attach-to-and-detach-from-a-running-container) as a sort of `tail -f` on a container.

Stop: `$ docker stack rm sakura`

Remove stopped containers `$ docker ps -aq --no-trunc | xargs docker rm`

### Note
Firewall on Production: may need to adjust the firewall via [UFW](https://linode.com/docs/security/firewalls/configure-firewall-with-ufw/).

# Adding Other Containers
If other sites are added, those can be surfaced to nginx-proxy automatically by setting environment variables `VIRTUAL_HOST`. Note that if needing to share networks or volumes, they must be `sakura-` in front of the name, i.e. `sakura_webnet`.

# Containers
1. nginx-proxy
2. letsencrypt
3. web
4. log

## nginx-proxy
Automated nginx proxy for Docker containers using docker-gen. docker-gen generates reverse proxy configs for nginx and reloads nginx when containers are started and stopped.

[Source Code Link](https://github.com/jwilder/nginx-proxy)

This container also provides an HTTPS front end to all other sits.

### HTTPS local
[How to get HTTPS working on your local development environment in 5 minutes](https://medium.freecodecamp.org/how-to-get-https-working-on-your-local-development-environment-in-5-minutes-7af615770eec)

Note: The certificate and keys should be named after the virtual host with a .crt and .key extension. For example, a container with VIRTUAL_HOST=foo.bar.com should have a foo.bar.com.crt and foo.bar.com.key file in the certs directory.


## letsencrypt
letsencrypt-nginx-proxy-companion is a lightweight companion container for the nginx-proxy. It allows the creation/renewal of Let's Encrypt certificates automatically. 

[Source Code Link](https://github.com/JrCs/docker-letsencrypt-nginx-proxy-companion)


## web
Using the official nginx:alpine build

[Source on docker.com](https://store.docker.com/images/nginx) 

Putting conf files for each site into `/etc/nginx/conf.d` to keep this as normal as possible. 

Note that any new sites added, that `VIRTUAL_HOST` and `LETSENCRYPT_HOST` must be the same so that Let's Encrypt can create and set the correct certs.


## log
Logspout is a log router for Docker containers that runs inside Docker. It attaches to all containers on a host, then routes our logs to [Papertrail](https://papertrailapp.com).

[Source Code link](https://github.com/gliderlabs/logspout)

Note that we added `SYSLOG_HOSTNAME=moose-docker` so we can see all messages under one ID.

