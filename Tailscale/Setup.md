## Tailscale Setup

1. On the Proxmox node, install and bring up Tailscale.

```
sudo apt install tailscale

# then enable it
tailscale up
```

2. It prints an authentication link. Open it to authenticate the node.

![Auth link](Pasted%20image%2020260623213129.png)

3. Log in with a Tailscale account or one of the listed identity providers.

![Login page](Pasted%20image%2020260623213227.png)
