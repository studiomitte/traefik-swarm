# traefik-swarm

Use traefik reverse proxy with docker swarm, based on this blog post: https://dockerswarm.rocks/traefik/

* docker service name is *traefik*
* overlay network name is *traefik-public*
* uses docker "host mode" to get real ip addresses and enable ipv6 ingress 
* configured to run on single node in swarm based on constraints
* predefined middlewares for http, https and redirects from non-www to www

## Setup instructions
Add the following environment variables or use a shell script for that.


Create docker overlay network
```
docker network create --driver=overlay --attachable traefik-public
```

  

Export current Node Id to ensure that traefik runs always on same node
```
export NODE_ID=$(docker info -f '{{.Swarm.NodeID}}')
```

  

Create tag on node to add constraint for deployment
```
docker node update --label-add traefik-public.traefik-public-certificates=true $NODE_ID
```

  

Create Email for letsencrypt notifications
```
export EMAIL=my@email.com
```
  

Domain for traefik UI
```
export DOMAIN=my.domain.com
```

  

Basic Auth User for Traefik

```
export USERNAME=MY_USER \
export PASSWORD=MY_SECURE_PASSWORD
```

  
Hash Password
```
export HASHED_PASSWORD=$(openssl passwd -apr1 $PASSWORD)
```

  

Deploy Stack:
```
docker stack deploy -c docker-compose.yml traefik
```

  

Access Logs & Rotation
----------------------

*   Create directory `/var/log/traefik` and mount it in traefik docker-compose file (will be created by init script)

```
    volumes:
      ...
      # Mount log folder
      - /var/log/traefik:/var/log/traefik
```

*   Add log rotation on Host System - example for Ubuntu

```
nano /etc/logrotate.d/traefik

# assuming traefik container contains "traefik" in its name
/var/log/traefik/*.log {
        daily
        missingok
        rotate 30
        compress
        delaycompress
        notifempty
        create 0644 root root
        sharedscripts
        postrotate
           # kill & resstart container which contains "traefik" in name
           docker kill --signal="USR1" $(docker ps | grep traefik | awk '{print $1}')
        endscript
}

# debug logrotate & run it
logrotate /etc/logrotate.conf --debug 
logrotate /etc/logrotate.conf
```
