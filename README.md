# mtproto-tls-server 

## 1. Install Nginx

Install web server from repositories.

```
apt install nginx
```

Edit `/etc/nginx/sites-available/default`.

Insert actual server name.

Restart Nginx.

```
systemctl restart nginx
```

## 2. Add initial web contents

You can adapt the `html` directory from this repository.

## 3. Get certificate from Let's Encrypt

```
apt install certbot python-certbot-nginx
certbot --nginx
certbot renew --dry-run
```

## 4. Install Go language

See https://golang.org/doc/install for documentation.

```
wget https://golang.org/dl/go1.15.2.linux-amd64.tar.gz
tar -C /usr/local -xzf go1.15.2.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
```

## 5. Download 9seconds/mtg 

See https://github.com/9seconds/mtg for documentation.

```
apt install git
git clone https://github.com/9seconds/mtg.git
```

## 6. Build mtg

```
cd mtg
go build
cp mtg /usr/local/bin
```

## 7. Generate secret for mtg

Replace `host.example.com` by your server's actual hostname.

```
mtg generate-secret -c host.example.com tls
```

This returns `<SECRET>`.

## 8. Reconfigure Nginx

Edit `/etc/nginx/sites-available/default` to make Nginx listen on port 993.

```
        listen [::]:993 ssl ipv6only=on; # managed by Certbot
        listen 993 ssl; # managed by Certbot
```

Restart Nginx.

```
apt install nginx
```

## 9. Configure mtg

Create `/usr/lib/systemd/system/mtg.service` and insert your actual `<SECRET>` instead of the placeholder. 

```
[Unit]
Description=Telegram MTProto Proxy Server
Documentation=https://github.com/9seconds/mtg
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
ExecStart=/usr/local/bin/mtg run -v -w 128KB -r 128KB --prefer-ip ipv6 --cloak-port 993 -b 0.0.0.0:443 <SECRET>
AmbientCapabilities=CAP_NET_BIND_SERVICE
Restart=never
RestartSec=1
LimitNOFILE=65536
PrivateDevices=true
ProtectControlGroups=true
ProtectHome=true
ProtectKernelTunables=true
ProtectKernelModules=yes
ProtectControlGroups=yes
DynamicUser=yes
ProtectSystem=full
RestrictSUIDSGID=true
PrivateTmp=yes
NoNewPrivileges=yes
ProtectClock=yes
ProtectKernelLogs=yes
CapabilityBoundingSet=~CAP_SET(UID|GID|PCAP)
CapabilityBoundingSet=~CAP_SYS_ADMIN
CapabilityBoundingSet=~CAP_SYS_PTRACE
RestrictNamespaces=~CLONE_NEWUSER 

[Install]
WantedBy=multi-user.target
```

## 10. Run mtg

```
systemctl enable mtg
systemctl start mtg
```

## 11. Check mtg

```
systemctl status mtg
journalctl -u mtg
```

## 12. Update web content

Update hostname, port, secret, start date and expiry date in web contents.
