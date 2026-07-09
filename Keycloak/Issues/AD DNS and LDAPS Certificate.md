## Problem

The Keycloak VM could not resolve `keenan.sec` (the AD domain), and once DNS worked the LDAPS handshake failed because the domain controller was not presenting a certificate on port 636.

## Fix DNS

Point the VM at Pi-hole so it can resolve `keenan.sec`.

```
nmcli con show                      # get the connection name
nmcli con mod "Wired connection 1" ipv4.dns "192.168.0.76"
nmcli con mod "Wired connection 1" ipv4.ignore-auto-dns yes
nmcli con up "Wired connection 1"
```

On a fresh box I first had to install NetworkManager.

```
apt install network-manager
nmcli con show
nmcli con mod "lo" ipv4.dns "192.168.0.76"
nmcli con mod "lo" ipv4.ignore-auto-dns yes
nmcli con up "lo"
```

`nslookup` then returned the AD domain in the DNS response.

![nslookup resolving keenan.sec](../images/Pasted%20image%2020260626080301.png)

![DNS confirmed](../images/Pasted%20image%2020260626082027.png)

### Test DNS

```
nslookup -type=A keenan.sec
getent hosts keenan.sec
ping keenan.sec
```

## Fix LDAPS certificate

I tried to pull the DC's cert over TLS and got nothing back.

```
keytool -printcert -sslserver keenan.sec:636
keytool error: java.lang.Exception: No certificate from the SSL server
```

The DC had no LDAPS certificate because there was no CA in the domain to issue one. Fix was to install AD CS on the Windows Server and stand up an Enterprise CA.

![Add AD CS role](../images/Pasted%20image%2020260626084755.png)

![AD CS role services](../images/Pasted%20image%2020260626084952.png)

1. Install the Active Directory Certificate Services role.

![Install](../images/Pasted%20image%2020260626085038.png)

2. Click `Configure Active Directory Certificate Services`.

![Configure AD CS](../images/Pasted%20image%2020260626085233.png)

![Credentials](../images/Pasted%20image%2020260626085429.png)

![Role services page](../images/Pasted%20image%2020260626085601.png)

3. Role Services: check **Certification Authority**.

![Certification Authority](../images/Pasted%20image%2020260626085612.png)

4. Setup Type: **Enterprise CA**.

![Enterprise CA](../images/Pasted%20image%2020260626085709.png)

![CA type](../images/Pasted%20image%2020260626090428.png)

5. Leave the remaining pages at their defaults and click `Next`.

![Defaults](../images/Pasted%20image%2020260626090450.png)

![Validity](../images/Pasted%20image%2020260626090606.png)

![Confirmation](../images/Pasted%20image%2020260626090635.png)

6. Click `Configure`.

![Configure](../images/Pasted%20image%2020260626090736.png)

The DC now presents a certificate on 636.

```
keytool -printcert -sslserver keenan.sec:636
```

![Cert now present](../images/Pasted%20image%2020260626090936.png)

## Wire the cert into Keycloak

Grab the DC cert (PEM), import it into Keycloak's truststore, and point Keycloak at that truststore.

```
# Save the DC cert to dc.crt
keytool -printcert -rfc -sslserver WIN-OLEUHE22T3N.keenan.sec:636 > dc.crt

# Import into Keycloak's PKCS12 truststore as ad-dc
keytool -importcert -alias ad-dc -file dc.crt \
  -keystore /opt/keycloak/conf/truststore.p12 -storetype PKCS12 -noprompt
```

Point Keycloak at the truststore in `keycloak.conf`:

```
truststore-paths=/opt/keycloak/conf/truststore.p12
```

Add the truststore password to the systemd unit:

```
nano /etc/systemd/system/keycloak.service
# add under [Service]
Environment=KC_TRUSTSTORE_PASSWORD=randompass
```

Use the DC's FQDN for the LDAP connection URL in Keycloak.

![Copy the connection domain](../images/firefox_JbPPN8y3DM%202.png)
