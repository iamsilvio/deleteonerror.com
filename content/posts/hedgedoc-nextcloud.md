---
title: "How to install HedgeDoc"
subtitle: "with nginx and nextcloud oAuth login"
date: 2021-04-05T13:32:31+02:00
draft: false
tags: ["webserver", "selfhosted", "oss"]
categories: [itstuff]
---

## Preconditions

### not part of this tutorial

- nginx installation
- mariadb installation
- nextcloud installation
- how to exit VIM therefor I use nano in all commands
<!--more-->
### linux packages

#### nodejs 12, yarn and git

Add the nodejs 12 Sources by executing  
`curl -fsSL https://deb.nodesource.com/setup_12.x | sudo -E bash -`  
  
Add the Yarn public GPG key  
`curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -`  
  
Add the Yarn package Source  
`echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list`  
  
Finally update the sources and install  
`sudo apt update && sudo apt install nodejs yarn git`

### database setup

Start a sql session as root by executing `sudo mysql -uroot`

``` sql
-- create the Database
CREATE DATABASE hedgedoc;
-- create the user with password
CREATE USER 'hedgedoc' IDENTIFIED BY 'CHOSE A PASSWORD';
-- grant all privileges on the new database to the new user
GRANT ALL privileges ON `hedgedoc`.* TO 'hedgedoc'@localhost;
-- apply changes without restart
FLUSH PRIVILEGES;

```

### nginx Setup

Now lets create the nginx site configuration  
`sudo nano /etc/nginx/sites-available/YOUR_HEDGEDOC_DOMAIN`
  
My config looks like this:

``` config
map $http_upgrade $connection_upgrade {
        default upgrade;
        ''      close;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name YOUR_HEDGEDOC_DOMAIN;

    ssl_certificate /etc/nginx/tls/YOUR_HEDGEDOC_DOMAIN.crt;
    ssl_certificate_key /etc/nginx/tls/YOUR_HEDGEDOC_DOMAIN.key;
    ssl_session_timeout 1d;
    ssl_session_cache shared:MozSSL:10m;  # about 40000 sessions
    ssl_session_tickets off;

    # modern configuration
    ssl_protocols TLSv1.3;
    ssl_prefer_server_ciphers off;

    # HSTS (ngx_http_headers_module is required) (63072000 seconds)
    add_header Strict-Transport-Security "max-age=63072000" always;

    # OCSP stapling
    ssl_stapling on;
    ssl_stapling_verify on;

    # verify chain of trust of OCSP response using Root CA and Intermediate certs
    ssl_trusted_certificate /etc/nginx/tls/trusted_chain.crt; 

    add_header Referrer-Policy "no-referrer" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Download-Options "noopen" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Permitted-Cross-Domain-Policies "none" always;
    add_header X-Robots-Tag "none" always;
    add_header X-XSS-Protection "1; mode=block" always;

    location / {
                proxy_pass http://127.0.0.1:3009;
                proxy_set_header Host $host; 
                proxy_set_header X-Real-IP $remote_addr; 
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
                proxy_set_header X-Forwarded-Proto $scheme;
        }

        location /socket.io/ {
                proxy_pass http://127.0.0.1:3009;
                proxy_set_header Host $host; 
                proxy_set_header X-Real-IP $remote_addr; 
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
                proxy_set_header X-Forwarded-Proto $scheme;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection $connection_upgrade;
        }
}
```

How you get a certificate will not be part of this tutorial but you can find a simple description [here](https://certbot.eff.org/docs/using.html#nginx)

Now link your site config to the enabled sites `sudo ln -s /etc/nginx/sites-available/YOUR_HEDGEDOC_DOMAIN /etc/nginx/sites-enabled/YOUR_HEDGEDOC_DOMAIN`  
  
And test if your config is fine with `sudo nginx -t`.  
If you get no error you can reload nginx by executing `sudo service nginx reload`.  

### HedgeDoc setup

If you want to use nextcloud as oAuth provider you have to configure a new oAuth 2.0 client in your nextcloud administration Interface.
Go to **Settings > Administration > Security** scroll to the bottom, there you can add a client, name it HedgeDoc and the URL is `https://YOUR_HEDGEDOC_DOMAIN/auth/oauth2/callback`

Create a new user without the capability to log in by  
`sudo adduser --disabled-login hedgedoc` and switch to the new user with `sudo su - hedgedoc`.  

Clone the git repository `git clone -b 1.7.2 https://github.com/hedgedoc/hedgedoc.git`  
  
Jump into the directory and run `bin/setup`  

Edit `config.json` to your preferences, here is mine:

``` json
{
  "production": {
    "domain": "YOUR_HEDGEDOC_DOMAIN",
    "loglevel": "info",
    "port": "3009",
    "protocolUseSSL": true,
    "useSSL": false,
    "sessionSecret": "USE A RANDOM STRING AS YOUR SECRET",
    "allowAnonymous": false,
    "allowAnonymousEdits": true,
    "email": false,
    "allowGravatar": true,
    "defaultPermission": "private",
    "hsts": {
      "enable": true,
      "maxAgeSeconds": 31536000,
      "includeSubdomains": true,
      "preload": true
    },
    "csp": {
      "enable": true,
      "directives": {},
      "upgradeInsecureRequests": "auto",
      "addDefaults": true,
      "addDisqus": false,
      "addGoogleAnalytics": false
    },
    "cookiePolicy": "lax",
    "db": {
      "username": "hedgedoc",
      "password": "YOUR DB PASSWORD",
      "database": "hedgedoc",
      "host": "localhost",
      "port": "3306",
      "dialect": "mariadb"
    },
    "oauth2": {
      "clientID": "YOUR NEXTCLOUD CLIENT ID",
      "clientSecret": "YOUR NEXTCLOUD SECRET",
      "authorizationURL": "https://NEXTCLOUD_URL/apps/oauth2/authorize",
      "tokenURL": "https://NEXTCLOUD_URL/apps/oauth2/api/v1/token",
      "userProfileURL": "https://NEXTCLOUD_URL/ocs/v2.php/cloud/user?format=json",
      "userProfileUsernameAttr": "ocs.data.id",
      "userProfileDisplayNameAttr": "ocs.data.display-name",
      "userProfileEmailAttr": "ocs.data.email"
    },
    "linkifyHeaderStyle": "gfm"
  }
}

```

edit the database configuration in `.sequelizerc`, the important part is the URL segment, this has to be your Database Connection String

``` json
...

'url': 'mariadb://USER:PASSWORD@localhost:3306/DATABASENAME'

...
```

Run `yarn run build` and start hedgeDoc with `NODE_ENV=production yarn start`. Watch the output for any errors, the site should now be accessible under your domain.
If it runs without problems, you can stop the process with `ctrl-c` and type `exit` to switch back to your normal user.

### Running HedgeDoc at system start

The last step is to set up the Yarn process to start even if no one is logged in or the server is restarted.  
  
First we need to create a systemd configuration file `sudo nano /lib/system/system/hedgedoc.service`  
  
My configuration looks like this

``` config
[Unit]
Description=hedgedoc
After=network.target

[Service]
Environment=NODE_ENV=production
Type=simple
User=hedgedoc
ExecStart=/usr/bin/yarn start
WorkingDirectory=/home/hedgedoc/hedgedoc
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Now we enable the service and reload the daemon by executing  
`sudo systemctl enable hedgedoc.service && sudo systemctl daemon-reload` with `sudo systemctl start hedgedoc` you can start the service and with `sudo systemctl status hedgedoc` you can check the current status of the service.
  
That's it, congratulations on your self hosted open-source collaborative markdown editor.
