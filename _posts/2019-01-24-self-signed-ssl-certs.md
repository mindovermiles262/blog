---
title: Self-Signed SSL
date: 2019-01-24
layout: post
author: andrew
categories: [ linux, "system administration" ] 
---

Sometimes you're working on an internal network and you need to enable SSL. Why? Because you don't want your credentials sniffed by anyone on the network.

Unfortunately, most SSL certificates cost money and cerbot needs public DNS to "verify" your domain.  The solution? Create a self-signed certificate.

This guide will walk you through setting up a self-signed SSL certificate. It won't get you past the "This Site is Insecure" warning on Chrome, but it will enable encrypted connections between client and server.

All of this tutorial will take place on the server.

The first step we need to take is creating a public and private key file. To do this, we can use the `openssl` library:

```
openssl req \
    -x509 \
    -newkey rsa:4096 \
    -sha256 \
    -days 3650 \
    -nodes \
    -keyout mydomain.key \
    -out mydomain.crt \
    -extensions san \
    -config <(echo "[req]"; echo distinguished_name=req; echo "[san]"; echo subjectAltName=DNS:mydomain.com,IP:192.168.50.10) \
    -subj /CN=mydomain.com
```

Be sure to change `keyout` (Your private key), `out` (The public key), `DNS:mydomain.com`, `IP:192.168.50.10`, and `CN=mydomain.com`

This will generate the two necessary files. Be sure to change the owner and group to root.

```
$ sudo chown root:root mydomain.crt mydomain.key

$ ls -l
total 8
-rw-rw-r--. 1 root root 1749 Jan 24 12:29 mydomain.crt
-rw-------. 1 root root 3272 Jan 24 12:29 mydomain.key
```

Great, now that we have the files we can use them, but first let's move them to their proper directories:

```
$ sudo mv mydomain.crt /etc/ssl/certs/mydomain.pem

$ sudo mv mydomain.key /etc/ssl/private/mydomain.key
```

Note how we change `.crt` to `.pem`.

Lastly, we can edit our nginx config files to use SSL:

```
server {
    listen 443 ssl;
    ssl_certificate         /etc/ssl/certs/mydomain.pem;
    ssl_certificate_key     /etc/ssl/private/mydomain.key;
    ssl_ciphers		        EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH;
    ssl_protocols           TLSv1.1 TLSv1.2;

    [ .. Continue Main Server Block .. ]
}

# Redirect all HTTP to HTTPS:
server {
    listen 80;
    server_name www.mydomain.com mydomain.com
    return 301 https:$host$request_uri;
}
```

With this, we add SSL to the main configuration block of our website and tell nginx to use it.  We also add a new server block to redirect any traffic on port 80 (HTTP) to use HTTPS.

With that, we just need to restart the nginx service:

```
$ sudo systemctl restart nginx
```

and that's it! If you go to `http://mydomain.com` you should now be redirected to `https://mydomain.com`
