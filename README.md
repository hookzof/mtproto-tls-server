# mtproto-tls-server 

## 1. Install Nginx

```
apt install nginx
```

## 2. Add initial web contents

Adapt the `html` template in this repository

## 3. Get certificate from Let's Encrypt

```
apt install certbot python-certbot-nginx
certbot --nginx
certbot renew --dry-run
```

## 4. Install Go language

See https://golang.org/doc/install for documentation

```
wget https://golang.org/dl/go1.15.2.linux-amd64.tar.gz
tar -C /usr/local -xzf go1.15.2.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
```

## 5. Download 9seconds/mtg 

See https://github.com/9seconds/mtg for documentation

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

## 7. Configure mtg

```
mtg generate-secret -c host.example.com tls
```

Returns `<SECRET>`

## 8. Open firewall

We will use port 993, but you can use any port that normally uses TLS. Open the port in your server firewall.

## 9. Run mtg

Create `/usr/lib/systemd/system/mtg.service` and insert your actual `<SECRET>` instead of the placeholder. We will use port 993, but you can use any port that normally uses TLS.

```
[Unit]
Description=MTProxy (Telegram proxy server)
Documentation=https://github.com/9seconds/mtg

[Service]
Restart=always
ExecStart=/usr/local/bin/mtg run <SECRET> -b 0.0.0.0:993

[Install]
WantedBy=multi-user.target
```

```
systemctl enable mtg
systemctl start mtg
systemctl status mtg
journalctl -u mtg
```

## 10. Update web content with details

Update hostname, port, secret, and expiry date in web contents
