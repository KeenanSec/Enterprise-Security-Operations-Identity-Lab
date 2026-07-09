## Wazuh Install

1. With the Ubuntu server up, SSH in (I used MobaXterm) so I can paste the installer instead of typing it out.

![SSH session](Images/Pasted%20image%2020260707115529.png)

![Connected](Images/Pasted%20image%2020260707120638.png)

2. Run the Wazuh assisted install.

```
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
```

![Install running](Images/Pasted%20image%2020260707124842.png)
