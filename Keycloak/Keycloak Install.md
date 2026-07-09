## Installing Keycloak 26 with PostgreSQL backend

Keycloak 26 needs Java 21 so first I confirmed the version

![Pasted image 20260620062851](images/Pasted%20image%2020260620062851.png)

### 1. PostgreSQL

Create the database and user. Run as the `postgres` user, since it only accepts local commands from its own account.

```bash
sudo -u postgres psql <<'EOF'
CREATE DATABASE keycloak;
CREATE USER keycloak WITH PASSWORD 'RandomPass';
GRANT ALL PRIVILEGES ON DATABASE keycloak TO keycloak;
ALTER DATABASE keycloak OWNER TO keycloak;
EOF
```

### 2. Service user

Give Keycloak its own user so the service runs unprivileged. `/sbin/nologin` blocks anyone logging in as it.

```bash
groupadd keycloak
useradd -r -g keycloak -d /opt/keycloak -s /sbin/nologin keycloak
```

### 3. Install to /opt

Third-party software not from a package manager goes in `/opt`.

```bash
cd /opt
wget https://github.com/keycloak/keycloak/releases/download/26.0.7/keycloak-26.0.7.tar.gz
tar -xzf keycloak-26.0.7.tar.gz
mv keycloak-26.0.7 keycloak
chown -R keycloak:keycloak /opt/keycloak
```

### 4. Config

```bash
cat > /opt/keycloak/conf/keycloak.conf <<'EOF'
db=postgres
db-username=keycloak
db-password=root123
db-url=jdbc:postgresql://localhost:5432/keycloak

hostname=keycloak.keenan.sec
http-enabled=true
proxy-headers=xforwarded
EOF
```

### 5. Build

```bash
runuser -u keycloak -- /opt/keycloak/bin/kc.sh build
```

### 6. Systemd service

```bash
cat > /etc/systemd/system/keycloak.service <<'EOF'
[Unit]
Description=Keycloak
After=network.target postgresql.service

[Service]
User=keycloak
Group=keycloak
ExecStart=/opt/keycloak/bin/kc.sh start --optimized
Environment=KC_BOOTSTRAP_ADMIN_USERNAME=admin
Environment=KC_BOOTSTRAP_ADMIN_PASSWORD=RandomPass
Restart=on-failure
TimeoutStartSec=600

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable --now keycloak
```

![Pasted image 20260620080103](images/Pasted%20image%2020260620080103.png)

### Recover the admin account

`KC_BOOTSTRAP_ADMIN_*` only creates the admin on the very first start. If that boot failed, re-bootstrap a temp admin:

```bash
su -s /bin/bash keycloak -c "KC_BOOTSTRAP_ADMIN_USERNAME=recover KC_BOOTSTRAP_ADMIN_PASSWORD=Recover123! /opt/keycloak/bin/kc.sh start --optimized"
```