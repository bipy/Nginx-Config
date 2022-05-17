<h2 align="center">Best Practice of Nginx TCP Domain-based Distribution of Traffic</h2>

<div align="center">

Use **Nginx Stream Module** to control the network traffic in TCP layer (and protect your domain certificate, BTW)

Use **Proxy Protocol** to pass the **Real Remote IP**

Suitable for **Reverse Proxy** and **Static Website**

</div>

### As Reverse Proxy

```nginx
server {
    listen 2000 ssl http2 proxy_protocol reuseport;
    
    absolute_redirect off;

    ssl_certificate /etc/nginx/ssl/my.com/public.cer;
    ssl_certificate_key /etc/nginx/ssl/my.com/private.key;

    location / {
        proxy_pass http://127.0.0.1:3000;
        include    proxy.conf;
    }
}
```

### As Static Website

```nginx
server {
    listen 2000 ssl http2 proxy_protocol reuseport;
    
    absolute_redirect off;

    ssl_certificate /etc/nginx/ssl/my.com/public.cer;
    ssl_certificate_key /etc/nginx/ssl/my.com/private.key;

    root /var/www/web;
    index index.html;

    include general.conf;
}
```

### Transfer Traffic to whatever does not support proxy protocol

```nginx
stream {
    map $ssl_preread_server_name $backend_name {
        some_server.com relay;
    }

    upstream relay {
        server 127.0.0.1:4444;
    }

    server {
        listen 4444 proxy_protocol;
        proxy_pass real_backend;
    }

    server {
        listen 443 reuseport;
        proxy_pass $backend_name;
        proxy_protocol on;
        ssl_preread on;
    }
}
```