## Problem

I tried to ping google.com and I was unable to even though i was able to ping 8.8.8.8 so I knew it had to be a DNS issue. Below is the error that I got.


```
root@NPM:~# ping google.com
ping: google.com: Temporary failure in name resolution                               
```

## Resolution

What I did is click on the NPM container > DNS > Clicked on DNS Server and added the DNS Server 1.1.1.1

![Pasted image 20260623201304](../Images/Pasted%20image%2020260623201304.png)

## Test

I restarted the LXC and resolution worked

```
root@NPM:~# ping google.com
PING google.com (172.217.216.101) 56(84) bytes of data.
64 bytes from lcausr-in-f101.1e100.net (172.217.216.101): icmp_seq=1 ttl=101 time=24.7 ms
64 bytes from lcausr-in-f101.1e100.net (172.217.216.101): icmp_seq=2 ttl=101 time=23.5 ms
^C
```