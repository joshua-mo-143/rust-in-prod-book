# Setting Up Nginx
Nginx is an open source software for web serving, reverse proxying, caching, load balancing and more. We'll be using Nginx as a reverse proxy server.

Step 1:
```
server {
    #listen 80; # Only if sysctl net.ipv6.bindv6only = 1
    listen 80;
    listen [::]:80;    # The example for yourdomain.tld could be steadylearner.com
    # You just replace it for your domain.    server_name yourdomain.tld www.yourdomain.tld; # 1.    location / {
        # Forward requests to rocket v4.0 production port
        proxy_pass http://0.0.0.0:8000; # 2.
        proxy_buffering off; # Single Page App work faster with it
        proxy_set_header X-Real-IP $remote_addr; 
    }
}
```
