DNS servers configuration examples
==================================

This is a demo project to show most common configurations of various DNS server implementations.

**NOTE**: This is WIP and unstable project.

Overview
--------

DNS servers implementations:

- BIND9
- dnsmasq
- unbound (TBD)
- NSD (TBD)

Server configurations/roles:
- caching and resolving server
- forwarding server
- authoritative only server (master or slave), no caching and forwarding
- master and slave for a given zone
- delegate a subdomain (like gTLD servers do)
- both authoritative and forwarding server on private LAN
- stealth server, two servers: internal behind firewall and external server for public and private queries
- split horizon, different zone files used to serve single zone depending on client address/location (geographical)
- single name server, which uses BIND "views" feature to serves both internal/private and external/public queries/clients

Combinations of different roles within single name server deployment are also shown.

Various tricks and features covered:
- DNS load balancing (several A records for single name)
- disable caching (server would always go thru query resolve process)
- single name server, which acts as master for zone A and as slave for zone B
- wildcard records
- "glue" A record for delegated subdomain NS record
- virtual subdomains (subdomains declared within single zone, without delegation to another name server)
- reverse lookup zone files
- using BIND "rndc" to manage name server remotely (clear cache, reload zone files, trigger zone transfer)
- using ~/.digrc to make dig output prettier
- using BIND `named-checkconf` and `named-checkzone` to validate conf and zone file before starting server

Install and run
---------------
Every name server deployment runs inside unique Docker container. For BIND9 server, I'm using [resystit/bind9](https://hub.docker.com/r/resystit/bind9/) base Docker image, which runs BIND9 server on top of Linux Alpine.

Use Docker Compose to start all available name servers. Dedicated user-defined network is created: `subnet: 172.20.0.0/16`. Each name server runs inside individual container, joined to that network.

```
git clone <url> <path>
cd <path>
docker-compose up --build
```

There is a dedicated `client` container, which includes BIND server (rndc), bind-tools (dig) and nano. Use it to test various name server configurations:

```
docker-compose run client
/ # dig sun.galaxy @master
/ # dig asia.earth.sun.galaxy @delegated_subzone
/ # dig -x 172.20.0.22 @slave
```

Once you're done, just clean up everything (containers/networks/images):

```
docker-compose down --rmi all -v
```

Domain name hierarchy
---------------------

I use somewhat fictitious domain name from a space realm: `sun.galaxy`. It's good to reason about hierarhy. For example, 3-level domains: `earth.sun.galaxy`, `moon.sun.galaxy`, `mars.sun.galaxy`. 4-level subdomains can be created easy as well: `europe.earth.sun.galaxy`, even 5-level domains possible: `ua.europe.earth.sun.galaxy`.