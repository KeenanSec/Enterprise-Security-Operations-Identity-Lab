## Problem

The VM could not ping other nodes or the default gateway, and could not resolve any domain names.

## Fix

Ran `ip a` and found the interface was set with a `/32` CIDR instead of `/24`. A `/32` gives the host a one-address subnet with no route to its neighbors, so nothing on the LAN is reachable.

![ip a showing /32](Images/Pasted%20image%2020260708233926.png)

Changed the prefix from `/32` to `/24`.

```
nmcli con mod "Wired connection 1" ipv4.address 192.168.30.108/24
nmcli con down "Wired connection 1"
nmcli con up "Wired connection 1"
```
