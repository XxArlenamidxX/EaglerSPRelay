# EaglerSPRelay

## Creating a LAN Relay
`YOU NEED A VPS AND RECOMMENDED TO HAVE AN EAGLERCRAFT INSTANCE RUNNING ON IT`

### Simply download this on a folder and run `java -jar EaglerSPRelay.jar`

**IF YOU DONT HAVE AN EAGLERCRAFT INSTANCE ON YOUR VPS, YOU NEED TO REVERSE PROXY localhost:6699 TO A DOMAIN**

Run `java -jar EaglerSPRelay.jar --debug` to view debug info like all the IPs of incoming connections, as it is not shown by default because logging all that info will reduce performance when the relay is being pinged many times a second depending on it's popularity.

Edit the `relayConfig.ini` file generated on first launch to change the port and configure ratelimiting and such, and `relays.txt` to change the list of STUN and TURN relays reported to clients connecting to the relay, which are required to correctly establish a P2P LAN world connection in browsers

**The `origin-whitelist` config variable is a semicolon (`;`) seperated list of domains used to restrict what sites are to be allowed to use your relay. When left blank it allows all sites. Add `offline` to allow offline download clients to use your relay as well, and `null` to allow connections that do not specify an `Origin:` header. Use `*` as a wildcard, for example: `*.deev.is` allows all domains ending with "deev.is" to use the relay.**



2025 notes:

Shared worlds work between any two eaglercraft clients that share a relay server, anyone can join your world it is not limited to just the other devices on your local network

When adding the relay address to the client you must provide it in URI format like "ws://address:port" or "wss://address:port", and determining the IP address and setting up port forwarding is the same as making a regular minecraft server

Ratelimiting is the same as eaglercraftbungee, "ping-ratelimit" is for limiting pings from clients searching for worlds, "world-ratelimit" is for limiting creating and joining new worlds

The relay is not used for transferring the actual gameplay packets, it is only used for the initial discovery process to allow clients to find each other, stuff such as coordinates and chat messages aren't visible to the relay

An example of a reverse proxy configuration using nginx:

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

A full tutorial on how to setup the reverse proxy

**MAKE SURE YOUR RUNNING THIS ON ROOT**

apt install nginx certbot python3-certbot-nginx
rm /etc/nginx/sites-enabled/default
nano /etc/nginx/sites-enabled/eaglercraft-relay

PASTE THIS INTO THE CONFIGURATION
``
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
``
Save it
ln -s /etc/nginx/sites-available/eaglercraft-relay /etc/nginx/sites-enabled
nginx -t
systemctl restart nginx

Reverse proxy should be working now, if something goes add arlenrivalxs on discord
