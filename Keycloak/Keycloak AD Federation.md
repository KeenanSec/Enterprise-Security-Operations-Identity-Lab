## Federate Keycloak with Active Directory over LDAP

Prerequisite: DNS and the LDAPS certificate are working. See [AD DNS and LDAPS Certificate](Issues/AD%20DNS%20and%20LDAPS%20Certificate.md).

1. In the realm, go to `User Federation`.

![User Federation](images/Pasted%20image%2020260625233259.png)

2. Select `Add Ldap providers`.

![Add LDAP provider](images/Pasted%20image%2020260625233350.png)

3. I created a dedicated service account in AD for Keycloak to bind with.

![Keycloak service account](images/Pasted%20image%2020260626094219.png)

4. Set the connection URL to `ldap://DC_IP`. For the Bind DN, enter `username@domain`.

![Connection URL and Bind DN](images/Pasted%20image%2020260630051558.png)

![Provider settings](images/Pasted%20image%2020260630051725.png)

![Federation working](images/Pasted%20image%2020260701061929.png)
