# N8N_installation_aws_ec2_free
This will allow anyone to install n8n on aws unbuntu with ssl for free tier.



Prerequisites

Docker
Nginx
Certbot (For SSL setup)
Step 1: Installing Docker
First, you need to update your system's package list by running:


Copy code
sudo apt update
Then, you can install Docker by running:


Copy code
sudo snap install docker
Step 2: Starting n8n in Docker
To start n8n in Docker, you can use the following command:


Copy code
sudo docker run -d --restart unless-stopped -it \
--name n8n \
-p 5678:5678 \
-e N8N_SECURE_COOKIE=false \
-e N8N_HOST="simply.anagata.in" \
-e VUE_APP_URL_BASE_API="https://simply.anagata.in/" \
-e WEBHOOK_TUNNEL_URL="https://simply.anagata.in/" \
-v ~/.n8n:/root/.n8n \
n8nio/n8n
Please replace simply.anagata.in with your actual domain. After this, you can access n8n on http://simply.anagata.in:5678.

Step 3: Installing Nginx
You can install Nginx by running:


Copy code
sudo apt install nginx
Step 4: Configuring Nginx
After installing Nginx, you need to configure it to reverse proxy the n8n web interface. Here's an example configuration:

First, open a new file using a text editor. Here we'll use nano:


Copy code
sudo nano /etc/nginx/sites-available/n8n
Then, paste the following content:


Copy code
server {


location / {
        proxy_pass http://localhost:5678;
        proxy_set_header Connection '';
        proxy_http_version 1.1;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "upgrade";
        chunked_transfer_encoding off;
        proxy_buffering off;
        proxy_cache off;
       }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/anagataai.anagata.in/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/anagataai.anagata.in/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}
server {
    if ($host = anagataai.anagata.in) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    listen 80;
    server_name anagataai.anagata.in;
    return 404; # managed by Certbot


}
Again, replace simply.anagata.in with your actual domain.

After that, you can create a symbolic link of this file in the sites-enabled directory:


Copy code
sudo ln -s /etc/nginx/sites-available/n8n /etc/nginx/sites-enabled/
Then, test the configuration and restart Nginx:


Copy code
sudo nginx -t
sudo systemctl restart nginx
At this point, you should be able to access n8n on http://simply.anagata.in.

Step 5: Setting up SSL with Certbot
First, install Certbot and the Nginx plugin by running:


Copy code
sudo apt install certbot python3-certbot-nginx
Then, you can run Certbot and follow the on-screen instructions:


Copy code
sudo certbot --nginx -d simply.anagata.in
After finishing these steps, n8n should be set up with HTTPS on your domain simply.anagata.in.

Now, you should be able to access the n8n web interface via https://simply.anagata.in in your browser without adding the port number :5678.

Note: Don't forget to set up DNS A record for simply.anagata.in to point to your server IP address. You would also need to allow ports 80, 443 (for HTTPS), and 5678 (for n8n) in your firewall.





Note: sometimes you can get error like workflow stuck use following process

Try a Different Reverse Proxy Configuration
If none of the above steps work, you can try a different Nginx configuration for the reverse proxy. Here's an example:

nginx


Copy code
server {
    listen 80;
    server_name simply.anagata.in;

    location / {
        proxy_pass http://127.0.0.1:5678;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
After making the configuration changes, don't forget to reload Nginx:


Copy code
sudo nginx -t
sudo systemctl reload nginx
If the issue persists after trying all these steps, you may need to seek further assistance from the n8n community or check the n8n documentation for any additional troubleshooting steps specific to your setup.
