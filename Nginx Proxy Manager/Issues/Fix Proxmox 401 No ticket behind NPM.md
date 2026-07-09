
## Problem

I kept getting the **401 No ticket** error on Proxmox when I tried to log in. After enough research I found that Proxmox's auth ticket is sent as a TLS-only cookie, so the connection from my browser to the Nginx proxy also has to be HTTPS, not just the proxy's connection to Proxmox. My proxy was serving the front end over plain HTTP, so the ticket never came through.

## Resolution

The fix was an SSL cert on the proxy. Since `pve2.local` is an internal name, no public CA can issue for it, so I generated a self-signed cert myself. NPM also warns that key files with a passphrase aren't supported, so I generated the key without one using `-nodes`.

![Pasted image 20260625030756](../Images/Pasted%20image%2020260625030756.png)

![Pasted image 20260625025713](../Images/Pasted%20image%2020260625025713.png)

## Generating & Uploading Certificate and Certificate Key

I then generated the certificate using this command.

```
openssl req -x509 -nodes -days 825 -newkey rsa:2048 \
  -keyout pve2.key -out pve2.crt \
  -subj "/CN=pve2.local" \
  -addext "subjectAltName=DNS:pve2.local"
```

Then I uploaded pve2.crt and pve2.key into NPM, which added the custom certificate.

![Pasted image 20260625030551](../Images/Pasted%20image%2020260625030551.png)

![Pasted image 20260625030547](../Images/Pasted%20image%2020260625030547.png)

## Test

Now I am able to access proxmox without the 401 error and I am able to see all my services.

![Pasted image 20260625030731](../Images/Pasted%20image%2020260625030731.png)