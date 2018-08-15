# INTRODUCTION
Securing traffic between client application and Kernel is common security case.
AION provides conventional mechanism to enable HTTPS connection for JSON calls.
Please refer to steps below.

## Requirements
```
NGINX
AION Kernel
AION_Web3 API
```

## Deployment
```
sudo apt-get update
sudo apt-get install nginx
ngnix -v
cd /etc/nginx
sudo openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -keyout /etc/nginx/aion8545.key -out /etc/nginx/aion8545.crt
```
## Configuration
```
sudo cp /etc/nginx/sites-enabled/default /etc/nginx/sites-enabled/default.orig.<YYYYMMDD>
sudo vim /etc/nginx/sites-enabled/default
```
Replace the file code with following code 
```
server {

    listen 443;
    server_name localhost;
    ssl_certificate           /etc/nginx/aion8545.crt;
    ssl_certificate_key       /etc/nginx/aion8545.key;

    ssl on;
    ssl_session_cache  builtin:1000  shared:SSL:10m;
    ssl_protocols  TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers HIGH:!aNULL:!eNULL:!EXPORT:!CAMELLIA:!DES:!MD5:!PSK:!RC4;
    ssl_prefer_server_ciphers on;

    access_log            /var/log/nginx/aion8545.access.log;

    location / {
      proxy_set_header        Host $host;
      proxy_set_header        X-Real-IP $remote_addr;
      proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header        X-Forwarded-Proto $scheme;

      proxy_pass          http://localhost:8545;
      proxy_read_timeout  90;

      proxy_redirect      http://localhost:8545 https://localhost;
    }
  }
```
Restart NGINX
```
sudo systemctl restart nginx && sudo systemctl status nginx
```

## Client configuration
Create aionweb3test.js with the code below
```
process.env.NODE_TLS_REJECT_UNAUTHORIZED = "0";
var Web3 = require('web3')
var web3 = new Web3(new Web3.providers.HttpProvider("https://localhost"));
console.log(Web3)
```

## Testing
Execute nodejs script as follow

`node aionweb3test.js`

# SUMMARY
By completing the steps the HTTPS encrypted connection is enabled with end-to-end secure traffic.
