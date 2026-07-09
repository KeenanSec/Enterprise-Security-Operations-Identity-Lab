


## Home VLAN 60

| Action      | Destination    | Port   | Purpose                 |
| ----------- | -------------- | ------ | ----------------------- |
| Pass        | 192.168.30.100 | 53 T/U | DNS                     |
| Pas         | 192.168.10.101 | 443    | Access Services Via NPM |
| Pass        | WAN            | any    | Internet Access         |
| Block       | RFC1918        | any    | Block InterVlan Comm    |
| Block (log) | any            | any    | Implicit Deny           |
![Pasted image 20260708134050](Images/Pasted%20image%2020260708134050.png)

## VLAN 10 INGRESS

| Action | Destination        | Port      | Purpose                                 |
| ------ | ------------------ | --------- | --------------------------------------- |
| Pass   | 192.168.20.101-105 | App Ports | Prox to apps                            |
| Pass   | 192.168.30.100-102 | 443/8443  | proxy Pi-hole UI, Vaultwarden, Keycloak |
| Pass   | 192.168.40.101-103 | 443/9000  | proxy Wazuh, osTicket, theHive          |
| Pass   | 192.168.30.100     | 53        | DNS                                     |
| Pass   | 192.168.40.101     | 1514-1515 | own telemetry                           |
| Pass   | WAN                | 443       | ACME/updates                            |
| Block  | any                | any       | Implicit deny                           |
![Pasted image 20260708140937](Images/Pasted%20image%2020260708140937.png)
## VLAN 20 SERVICES

| Action | Destination    | Port      | Purpose                      |
| ------ | -------------- | --------- | ---------------------------- |
| Pass   | 192.168.30.102 | 8443      | OIDC backchannel to Keycloak |
| Pass   | 192.168.40.101 | 1514-1515 | telemetry                    |
| Pass   | 192.168.30.100 | 53        | DNS                          |
| Pass   | Wan            | 443       | n8n APIs / updates           |
| Block  | any            | any       | default deny                 |
![Pasted image 20260708134157](Images/Pasted%20image%2020260708134157.png)
## VLAN 30 IDENTITY

| Action        | Destination    | Port      | Purpose             |
| ------------- | -------------- | --------- | ------------------- |
| Pass          | 192.168.40.101 | 1514-1515 | telemetry           |
| Pass(Pi-hole) | 192.168.30.103 | 53        | conditional forward |
| Pass(Pi-hole) | WAN            | 53/853    | upstream resolvers  |
| Pass          | WAN            | 443       | updates/NTP         |
| Block         | any            | any       | Implicit Deny       |
![Pasted image 20260708134117](Images/Pasted%20image%2020260708134117.png)
## VLAN 40 SECURITY

| Action | Destination    | Port      | Purpose                                 |
| ------ | -------------- | --------- | --------------------------------------- |
| Pass   | 192.168.30.102 | 8443      | Wazuh/osTicket/theHive OIDC backchannel |
| Pass   | 192.168.30.100 | 53        | DNS                                     |
| Pass   | 192.168.40.101 | 1514-1515 | self telemetry                          |
| Pass   | WAN            | 443       | threat-intel feeds/updates              |
| Block  | any            | any       | implicit deny                           |
![Pasted image 20260708134129](Images/Pasted%20image%2020260708134129.png)

