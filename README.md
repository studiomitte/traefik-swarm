# traefik-swarm

Use traefik reverse proxy with docker swarm

* docker service name is *traefik*
* overlay network name is *traefik-public*
* uses docker "host mode" to get real ip addresses and enable ipv6 ingress 
* configured to run on single node in swarm based on constraints
* predefined middlewares for http, https and redirects from non-www to www

## Setup instructions
Add the following environment variables or use a shell script for thats

