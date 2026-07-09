
## Problem

I kept trying to access my Pihole from my test VM and I was unable to access the website despite it resolving the domain name to an Ip via nslookup. The problem was that I kept using domains ending in `.local` which is a reserved TLD for mDNS so instead of routing to my Pihole DNS server it tries to use the mDNS resolver and it never gets to see Pihole.


## Solution


I changed all my domain names from `pihole.local to pihole.keenan.sec , wazuh.local to wazuh.keenan.sec etc.` 

## Test

![Pasted image 20260701065406](../../Keycloak/images/Pasted%20image%2020260701065406.png)