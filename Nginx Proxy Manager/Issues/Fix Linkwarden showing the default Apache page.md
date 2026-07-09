
## Problem

I couldn't reach Linkwarden. Even though I'd set the correct destination IP and port `3000` in NPM, it kept showing the default Apache page, which meant traffic was hitting port 80 instead of Linkwarden on `3000`.

## Resolution

The Linkwarden LXC had Apache running and answering on port 80. I entered the container's terminal, stopped Apache, and disabled it so it won't come back on reboot.

```bash
sudo systemctl stop apache2
sudo systemctl disable apache2
```

## Test

After that, traffic routed to port `3000` and Linkwarden loaded.
