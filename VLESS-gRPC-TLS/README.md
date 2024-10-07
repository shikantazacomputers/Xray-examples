# VLESS-gRPC-TLS

These are expanded instructions for [https://github.com/chika0801/Xray-examples/VLESS-gRPC-TLS](https://github.com/chika0801/Xray-examples/tree/main/VLESS-gRPC-TLS).

They add the use of a CDN.

## 1. Add domain to Cloudflare CDN

Purchase a domain name. 

Example: `shikantaza.xyz`.

Add your domain to the Cloudflare free plan.

## 2. Create VPS

Purchase a VPS with Debian or Ubuntu operating system.

Choose a subdomain and map it to your VPS IP address.

Example: `grpc.shikantaza.xyz`.

Set the Proxy Status for your subdomain to DNS only (gray cloud icon).

SSH into your server.

Example: `ssh root@grpc.shikantaza.xyz`.

Update the server:

```
apt update && apt upgrade -y
```

## 3. Add SSL certificate

Open port `80/tcp` in your firewall.

Obtain an SSL certificate from Let's Encrypt. Replace the example `grpc.shikantaza.xyz` in what follows by your actual subdomain name.

```
apt install -y socat cron

curl https://get.acme.sh | sh

source ~/.bashrc

acme.sh --upgrade --auto-upgrade

acme.sh --set-default-ca --server letsencrypt

acme.sh --issue -d grpc.shikantaza.xyz --standalone --keylength ec-256

acme.sh --install-cert -d grpc.shikantaza.xyz --ecc --fullchain-file /etc/ssl/private/fullchain.cer --key-file /etc/ssl/private/private.key

chown -R nobody:nogroup /etc/ssl/private

acme.sh --renew -d grpc.shikantaza.xyz --force --ecc
```

## 4. Install and configure Nginx

Open port `443/tcp` in your firewall.

Install Nginx.

```
apt install nginx -y

nginx -v
```

Replace the entire original contents of `/etc/nginx/nginx.conf` with `VLESS-gRPC-TLS/nginx.conf`.

Replace the original user with `user www-data;`.

Comment out the `server` block on port `80` to prevent conflicts with SSL certificate renewal.

Example:

```
#    server {
#        listen 80;
#        listen [::]:80;
#        return 301 https://$host$request_uri;
#    }
```

For Nginx version below `1.25.1`, replace the port `443` `listen` and `http2` directives with:

```
#        listen                     443 ssl;
#        listen                     [::]:443 ssl;
#        http2                      on; 
        listen 443 ssl http2;
        listen [::]:443 ssl http2;
```

Replace the `server_name` example with your actual subdomain.

Example: `server_name grpc.shikantaza.xyz;`.

Replace the `location` (service name) example with your own value.

Example: `location /qwerty123`

Replace the `set $website` example with your own value.

Example: `set $website www.speedtest.net;`

Save the file.

Restart Nginx:

```
nginx -t

systemctl restart nginx

systemctl status nginx
```

## 5. Configure Cloudflare CDN

On the **DNS** page, set subdomain proxying on for your subdomain (orange cloud icon).

Example: `grpc.shikantaza.xyz`.

Go to the Cloudflare **SSL/TLS** page.

Configure the setting to `Full (strict)` mode.

Go to the Cloudflare **Network** page.

Toggle the switch to allow gRPC connections to your origin server.

Test your set-up so far by opening a browser and visiting your server.

Example:

```
https://grpc.shikantaza.xyz
```

You should see your proxy page (`https://www.speedtest.net` in our example).

You will need to SSH into your server by IP address from now on.

## 6. Install Xray server

Install Xray-core and the geodata files with `User=nobody`:

```
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install
```

## 7. Configure Xray server

Replace the installed `/usr/local/etc/xray/config.json` with the example from `VLESS-gRPC-REALITY/config_server.json`.

Replace the `id` example with your own UUID from https://www.uuidgenerator.net.

Example:

```
"id": "f4af6131-9996-4cc0-abe9-23bfbe208e2f"
```

Replace the `serviceName` example with your own.

Example:

```
"serviceName": "qwerty123"
```

Save the file.

Restart Xray:

```
systemctl restart xray

systemctl status xray
```

Exit your SSH session:

```
exit
```

## 8. Configure Xray client

### 8.1 Install, configure, and run CLI client

You can use the binary client from https://github.com/XTLS/Xray-core/releases if you prefer a command-line interface. 

Use `VLESS-gRPC-REALITY/config_server.json` as a model for your client configuration `config.json`.

Fill in the VLESS server address:

```
"address": "grpc.shikantaza.xyz",
```

Replace the `id` example with your own UUID.

Example:

```
"id": "f4af6131-9996-4cc0-abe9-23bfbe208e2f",
```

Replace `serviceName` example with your own.

Example:

```
"serviceName": "qwerty123",
```

Here are the suggested `grpcSettings` for use with Cloudflare CDN:

```
                "grpcSettings": {
                    "serviceName": "qwerty123",
                    "multiMode": true,
                    "idle_timeout": 60,
                    "health_check_timeout": 20,
                    "initial_windows_size": 35536,
                    "permit_without_stream": true                  
                },
```

Save the file.

Run the Xray CLI with your `config.json`, which is in the same folder as your binary:

```
./xray -c config.json
```

### 8.2 Install, configure, and run GUI client

You can use a graphical user interface client from https://github.com/XTLS/Xray-core?tab=readme-ov-file#gui-clients if you prefer a GUI.

Configure and run your GUI client.

### 8.3 Configure browser

Configure your browser to use the SOCKS proxy on port `10808`.

Visit https://whatismyipaddress.com.
