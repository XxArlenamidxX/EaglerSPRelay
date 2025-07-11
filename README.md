
# EaglerSPRelay

## Creating a LAN Relay
`YOU NEED A VPS AND RECOMMENDED TO HAVE AN EAGLERCRAFT INSTANCE RUNNING ON IT`

### Simply download this in a folder and run:
```bash
java -jar EaglerSPRelay.jar
```

**IF YOU DON'T HAVE AN EAGLERCRAFT INSTANCE ON YOUR VPS, YOU NEED TO REVERSE PROXY localhost:6699 TO A DOMAIN**

To view debug info like incoming IPs:
```bash
java -jar EaglerSPRelay.jar --debug
```

Edit the `relayConfig.ini` file generated on first launch to change the port, configure rate limiting, etc.  
Also edit `relays.txt` to update the list of STUN and TURN relays reported to clients — these are required to establish P2P LAN world connections in browsers.

> **`origin-whitelist` is a semicolon (`;`) separated list of domains allowed to use your relay.**
>
> Examples:
> - `*` allows everything  
> - `*.deev.is` allows all subdomains of `deev.is`  
> - Add `offline` to allow offline clients  
> - Add `null` to allow clients without an `Origin:` header

---

### 📅 2025 Notes

Shared worlds work between **any two Eaglercraft clients** that share a relay.  
Anyone can join — not limited to local devices.

Use **URI format** like:
```
ws://yourdomain.com:port
wss://yourdomain.com:port
```

Rate limiting:
- `ping-ratelimit`: limit pings from world search
- `world-ratelimit`: limit creating/joining new worlds

🔒 The relay is only for **discovery**, not for game packets. Chat, coords, gameplay = private.

---

### 🛡️ Example NGINX Reverse Proxy Config

```nginx
# HTTP to HTTPS redirect
server {
    listen 80;
    server_name <domain>;

    location / {
        return 301 https://$host$request_uri;
    }
}

# HTTPS proxy to relay
server {
    listen 443 ssl;
    server_name <domain>;

    ssl_certificate /etc/letsencrypt/live/<domain>/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/<domain>/privkey.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    location / {
        proxy_pass http://localhost:6699/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

### ⚙️ Full Setup (Debian/Ubuntu)

```bash
apt install nginx certbot python3-certbot-nginx
rm /etc/nginx/sites-enabled/default
nano /etc/nginx/sites-available/eaglercraft-relay
```

Then paste:

```nginx
server {
    listen 443 ssl;
    server_name <domain>;

    ssl_certificate /etc/letsencrypt/live/<domain>/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/<domain>/privkey.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    location / {
        proxy_pass http://localhost:6699/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

Now enable the site and reload NGINX:

```bash
ln -s /etc/nginx/sites-available/eaglercraft-relay /etc/nginx/sites-enabled
nginx -t
systemctl restart nginx
```

✅ Your reverse proxy should now work.  
📩 Questions? DM `arlenrivalxs` on Discord.
