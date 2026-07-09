![Pasted image 20260623031252](Images/Pasted%20image%2020260623031252.png)
![Pasted image 20260623031301](Images/Pasted%20image%2020260623031301.png)
![Pasted image 20260623031309](Images/Pasted%20image%2020260623031309.png)
![Pasted image 20260623031320](Images/Pasted%20image%2020260623031320.png)
![Pasted image 20260623031333](Images/Pasted%20image%2020260623031333.png)
![Pasted image 20260623031518](Images/Pasted%20image%2020260623031518.png)


Click on the NPM LXC Container > `Options` > `Features` & add `keyctl` and `features`.


`Keyctl` is what allows docker to manage security keys and encryption tokens in an unprivileged container.

![Pasted image 20260623031917](Images/Pasted%20image%2020260623031917.png)

Install necessary packages and docker.

```
# Update system packages
apt update && apt upgrade -y

# Install prerequisites
apt install -y curl sudo nano

# Download and run the official Docker installation script
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh

# Check If Docker is running

systemctl status docker

# Create and enter the npm directory

mkdir -p /opt/nginx-proxy-manager
cd /opt/nginx-proxy-manager

#Then create and configure the config file for docker

nano docker-compose.yml
```

Then add this to configuration file to configure what ports are going to be used.

```
version: '3.8'
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      # Public HTTP Port
      - '80:80'
      # Public HTTPS Port
      - '443:443'
      # Admin Web Port
      - '81:81'
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
```

Then turn on the container.

```
docker compose up -d
```

![Pasted image 20260623212037](Images/Pasted%20image%2020260623212037.png)

First I added the domain name to Pihole pointing to NPMs IP so it can forward the user to the correlating ip and port.

![Pasted image 20260626074820](Images/Pasted%20image%2020260626074820.png)

Then I added a proxy host

![Pasted image 20260625014106](Images/Pasted%20image%2020260625014106.png)

# Adding extra login form to services



Click "Access Lists" > Add Access List > Then go to the "Authorizations" tab and setup a username and password of your choosing.


![Pasted image 20260625015738](Images/Pasted%20image%2020260625015738.png)

Then go to Hosts > Proxy Hosts > click the 3 dots by the host you setup and edit it to add that access list.

![Pasted image 20260625015049](Images/Pasted%20image%2020260625015049.png)

It routed me to the correct side but I ran into a 401 error which I fixed in [Fix Proxmox 401 No ticket behind NPM](Issues/Fix%20Proxmox%20401%20No%20ticket%20behind%20NPM.md).