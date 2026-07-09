## Overview

I run Nginx Proxy Manager so I do not have to remember internal IPs and ports, and so every service is reached through one ingress point.

NPM needs a DNS server to point each domain at NPM's IP first. Pi-hole resolves all the service domains to NPM's IP. NPM then reads the Host header on the incoming request and forwards it to the correct backend IP and port from its config.
