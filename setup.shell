#!/bin/bash

# --- Input Requirements ---
read -p "Enter your domain name (e.g., example.com): " DOMAIN
read -p "Enter your email for Certbot: " EMAIL

if [[ -z "$DOMAIN" || -z "$EMAIL" ]]; then
    echo "Error: Domain and Email are required."
    exit 1
fi

echo "Starting installation for $DOMAIN..."

# 1. Update and Install Nginx
apt update
apt install -y nginx

# 2. Install Certbot and Python Nginx plugin
apt install -y certbot python3-certbot-nginx

# 3. Create Nginx Configuration
# This setup includes 1GB upload limit and WebSocket support headers.
NGINX_CONF="/etc/nginx/sites-available/$DOMAIN"

cat <<EOF > $NGINX_CONF
server {
    listen 80;
    server_name $DOMAIN;

    # Allow file uploads up to 1GB
    client_max_body_size 1024M;

    location / {
        proxy_pass http://localhost:3000; # Change this to your backend port
        
        # WebSocket Support
        proxy_http_version 1.1;
        proxy_set_header Upgrade \$http_upgrade;
        proxy_set_header Connection "upgrade";
        
        # Standard Proxy Headers
        proxy_set_header Host \$host;
        proxy_set_header X-Real-IP \$remote_addr;
        proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto \$scheme;
    }
}
EOF

# 4. Enable the configuration and test
ln -s $NGINX_CONF /etc/nginx/sites-enabled/
rm -f /etc/nginx/sites-enabled/default
nginx -t

if [ $? -eq 0 ]; then
    systemctl restart nginx
    echo "Nginx configured successfully."
else
    echo "Nginx configuration test failed."
    exit 1
fi

# 5. Final Step: Install SSL via Certbot
echo "Requesting SSL certificate for $DOMAIN..."
certbot --nginx -d $DOMAIN --non-interactive --agree-tos -m $EMAIL --redirect

# Reload Nginx to apply all changes
systemctl reload nginx

echo "----------------------------------------------------"
echo "Setup Complete!"
echo "Domain: https://$DOMAIN"
echo "Max Upload: 1GB"
echo "WebSocket Support: Enabled"
echo "----------------------------------------------------"
