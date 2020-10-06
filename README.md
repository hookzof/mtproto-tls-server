# mtproto-tls-server 

These instructions document how to create a Telegram MTProto proxy server with fake TLS name equal to actual hostname. This repository includes a sample web site `html` to announce the proxy details to users. 

You will need a server and a domain name. Create a DNS record for the hostname pointing to the server IP address. 

These instructions are for Debian/Ubuntu logged in as root.

## 1. Install Nginx

Install the Nginx web server from the repositories.

```
apt update && apt upgrade
apt install nginx
```

Edit `/etc/nginx/sites-available/default`.

Insert your actual hostname.

```
        server_name host.example.com;
```

Restart Nginx with your hostname defined in the server configuration.

```
systemctl restart nginx
```

## 2. Add initial web contents

You can adapt the `html` directory from this repository.

## 3. Get certificate from Let's Encrypt

Follow the instructions on https://certbot.eff.org to install a real certificate on your server.

```
apt install certbot python-certbot-nginx
certbot --nginx
```

The Let's Encrypt certificate needs to be updated approximately every 90 days. Set things up to check for the necessity of an update.

```
certbot renew --dry-run
```

## 4. Install Go language

See https://golang.org/doc/install for documentation. At the time of writing, the current version of Go language is 1.15.2.

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

See https://github.com/9seconds/mtg for documentation.

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

Generating the secret needs to be repeated every 90 days, when the Let's Encrypt certificate is renewed.

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

Run the mtg service.

```
systemctl enable mtg
systemctl start mtg
```

## 11. Check mtg

Check that mtg is active and running, and review messages.

```
systemctl status mtg
journalctl -u mtg
```

## 12. Update web content

Now that everything is running, update the hostname, port, secret, start date and expiry date in your web site's contents.

You will need to update your web site every 90 days, when the Let's Encrypt certificate and the secret are renewed.
