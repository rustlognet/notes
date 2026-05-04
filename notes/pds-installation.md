## Summary

PDS installation on Ubuntu server without docker using Nginx.

## Create DNS record 

- create `A` for `pds.example.com` pointing to `server-ip`.

## Install system packages

```bash
sudo apt update
sudo apt install -y git nginx certbot python3-certbot-nginx curl openssl xxd
```

## Create & switch to PDS user

```bash
sudo useradd -r -m -s /bin/bash pds
sudo su -l pds
```

## Install NVM and Node 22

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.4/install.sh | bash
. "$HOME/.nvm/nvm.sh"

nvm install 22
nvm use 22
nvm alias default 22
```

## Enable pnpm

```bash
corepack enable
corepack prepare pnpm@latest --activate
```

## Check (optional)

```bash
node -v
npm -v
pnpm -v
```

## Find exact Node path

```bash
which node
```

- Save that path for the `systemd` service later.

## Clone and install PDS

```bash
git clone https://github.com/bluesky-social/pds repo
cd ~/repo/service
pnpm install --production --frozen-lockfile
```

## Create config and data folder

```bash
cd ~
mkdir -p ~/data/blocks
nano ~/pds.env
```

```text
PDS_HOSTNAME=pds.example.com
PDS_SERVICE_HANDLE_DOMAINS=.example.com

PDS_JWT_SECRET=
PDS_ADMIN_PASSWORD=
PDS_PLC_ROTATION_KEY_K256_PRIVATE_KEY_HEX=

PDS_DATA_DIRECTORY=/home/pds/data
PDS_BLOBSTORE_DISK_LOCATION=/home/pds/data/blocks
PDS_BLOB_UPLOAD_LIMIT=52428800

PDS_DID_PLC_URL=https://plc.directory
PDS_BSKY_APP_VIEW_URL=https://api.bsky.app
PDS_BSKY_APP_VIEW_DID=did:web:api.bsky.app
PDS_REPORT_SERVICE_URL=https://mod.bsky.app
PDS_REPORT_SERVICE_DID=did:plc:ar7c4by46qjdydhdevvrndac
PDS_CRAWLERS=https://bsky.network

LOG_ENABLED=true
PDS_PORT=3000
NODE_ENV=production
```

### Generate secrets

```bash
openssl rand --hex 16
openssl rand --hex 16
openssl ecparam --name secp256k1 --genkey --noout --outform DER \
 | tail --bytes=+8 \
 | head --bytes=32 \
 | xxd --plain --cols 32
```

Put the output into config file:

- `PDS_JWT_SECRET=`
- `PDS_ADMIN_PASSWORD=`
- `PDS_PLC_ROTATION_KEY_K256_PRIVATE_KEY_HEX=`

## Lock permissions

```bash
chmod 600 ~/pds.env
```

## Exit PDS user

```bash
exit
```

## Create systemd service

```bash
sudo nano /etc/systemd/system/pds.service
```

```text
[Unit]
Description=Bluesky Personal Data Server
After=network.target

[Service]
User=pds
Type=simple
EnvironmentFile=/home/pds/pds.env
WorkingDirectory=/home/pds/repo/service
ExecStart=NODE-PATH --enable-source-maps index.js
Restart=on-failure
RestartSec=3

[Install]
WantedBy=multi-user.target
```

- replace `NODE-PATH` with your `which node` output.

### Enable the service

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now pds
```

### Test locally

```bash
curl http://localhost:3000/xrpc/_health
```

## Configure Nginx

```bash
sudo nano /etc/nginx/sites-available/pds.example.com
```

```text
map $http_upgrade $connection_upgrade {
  default upgrade;
  '' close;
}

server {
  listen 80;
  listen [::]:80;
  server_name pds.example.com;

  location / {
    proxy_pass http://127.0.0.1:3000;

    proxy_http_version 1.1;

    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $connection_upgrade;

    client_max_body_size 100M;
    proxy_read_timeout 600s;
    proxy_send_timeout 600s;
  }
}
```

### Enable nginx config

```bash
sudo ln -s /etc/nginx/sites-available/pds.example.com /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

## Add HTTPS

```bash
sudo certbot --nginx -d pds.example.com
```

### Test

```bash
curl https://pds.example.com/xrpc/_health
```

- You should see something like `{"version":"0.x.x"}`

## Create first account

```bash
sudo su -l pds
cd /home/pds/repo
export PDS_ENV_FILE=/home/pds/pds.env

./pdsadmin.sh account create you@example.com admin.example.com
```

## Get your DID

```bash
./pdsadmin.sh account list
```

- Copy the `did:plc:...` value.

## Add DNS record for your handle

```text
Name:  _atproto.example.com
Type:  TXT
Value: did=did:plc:abc123xyz
```

### Check

```bash
dig TXT _atproto.example.com
```

## Change handle to example.com

```bash
./pdsadmin.sh account update-handle example.com
```

- Alternatively, you can do so via bluesky app.

## Request crawl

```bash
./pdsadmin.sh request-crawl
```

```bash
exit
```

## Maintenance

### Logs

```bash
sudo journalctl -fu pds
```

### Restart

```bash
sudo systemctl restart pds
```

### Status

```bash
sudo systemctl status pds
```

### Update

```bash
sudo su -l pds
. "$HOME/.nvm/nvm.sh"
cd /home/pds/repo
git pull
cd service
pnpm install --production --frozen-lockfile
exit

sudo systemctl restart pds
```

### Backup

```bash
sudo tar -czf pds-backup.tar.gz /home/pds/data /home/pds/pds.env
```

The most important things to preserve are:
- `/home/pds/data`
- `/home/pds/pds.env`

Especially:
- `PDS_PLC_ROTATION_KEY_K256_PRIVATE_KEY_HEX`

