
# EaglerSPRelay

## Creating a LAN Relay
`YOU NEED A VPS AND RECOMMENDED TO HAVE AN EAGLERCRAFT INSTANCE RUNNING ON IT`

### Simply download this in a folder and run:
```bash
java -jar EaglerSPRelay.jar
```

**IF YOU DON'T HAVE AN EAGLERCRAFT INSTANCE ON YOUR VPS, YOU NEED TO REVERSE PROXY localhost:6699 TO A DOMAIN BECAUSE YOU CANT USE IP WEBSOCKETS IN A HTTPS EAGLERCRAFT INSTANCE**

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

# EaglerSPRelay Setup
```bash
# Install Dependencies
apt install openjdk-17-jdk
apt install unzip

# Install and Start Relay
curl -L https://github.com/XxArlenamidxX/EaglerSPRelay/archive/refs/heads/main.zip -o EaglerSPRelay.zip
unzip EaglerSPRelay.zip
cd EaglerSPRelay-main
java -jar EaglerSPRelay.jar
```

# WARNING - THE SERVICE SETUP I HAVE NOT TESTED SO I CANNOT CONFIRM IF IT WORKS, IF IT FAILS YOU HAVE TO FIX IT BY YOURSELF
**OPTIONAL - YOU CAN SETUP A SERVICE THAT RUNS IT IN THE BACKGROUND**
```bash
sudo nano /etc/systemd/system/eaglersprelay.service
```
PASTE THIS IN:

```ini
[Unit]
Description=EaglerSPRelay
After=network.target

[Service]
WorkingDirectory=/EaglerSPRelay-main
ExecStart=/usr/bin/java -jar EaglerSPRelay.jar
Restart=always

[Install]
WantedBy=multi-user.target
```

Now restart systemctl and enable the service
```bash
systemctl daemon-reload
systemctl enable --now eaglersprelay.service
```

# NGINX Reverse Proxy Setup
```bash
apt install nginx certbot python3-certbot-nginx
# Replace <domain> with your desired domain (You have to have an A record pointing to the public IP of your VPS)
certbot certonly --nginx -d <domain>
rm /etc/nginx/sites-enabled/default
nano /etc/nginx/sites-available/eaglercraft-relay
```

Then paste:

```nginx
# Replace <domain> with your domain
server {
    listen 80;
    server_name <domain>;

    location / {
        return 301 https://$host$request_uri;
    }
}

# Replace <domain> with your domain
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

Now enable the site, reload NGINX and if you setup the relay to run in the background restart it:

```bash
ln -s /etc/nginx/sites-available/eaglercraft-relay /etc/nginx/sites-enabled
nginx -t
systemctl restart nginx

# If you setup the relay as a service
systemctl restart eaglersprelay
```

Now, go to your eaglercraft instance (or just go to https://eaglercraft.com/mc/1.8.8-wasm)
Put in your relay domain with `wss://`
<img width="793" height="611" alt="image" src="https://github.com/user-attachments/assets/2e2406ab-dbd0-49c2-9684-40779d08cdd7" />
And click `add relay`
<img width="760" height="102" alt="image" src="https://github.com/user-attachments/assets/0a6e68e5-ff8d-46dc-965f-fbf8ebe6afc5" />
If you did everything correctly, it should change the name to `Eags. Shared World Relay` and have a green status 

✅ Your relay should now work.  
ℹ️ You can change the name, port or any setting in `relayConfig.ini`
📩 Questions? Problems? DM `arlenrivalxs` on Discord.

**Custom Relay made by Me: wss://minecraft.arlensdash.online/relay**
