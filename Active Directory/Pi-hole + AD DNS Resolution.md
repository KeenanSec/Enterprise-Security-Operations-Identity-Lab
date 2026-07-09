
## Problem

Endpoints couldn't resolve AD Domain `keenan.sec` through Pi-hole.

**Fix:**

1. I added a conditional forward in dnsmasq.

```
sudo nano /etc/dnsmasq.d/02-keenan-ad.conf

# Add this to the file
server=/keenan.sec/192.168.0.185

```

Pi-hole v6 ignores `/etc/dnsmasq.d/` by default, so I enabled it:

```
sudo pihole-FTL --config misc.etc_dnsmasq_d true
```

2. Pointed the Pi-hole LXC's DNS at itself (`127.0.0.1`).

3. Restarted the LXC.

## Test

```
nslookup keenan.sec 192.168.0.76
nslookup -type=SRV _ldap._tcp.dc._msdcs.keenan.sec 192.168.0.76
cat /etc/resolv.conf
```