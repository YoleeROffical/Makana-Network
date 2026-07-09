services:
  npm:
    image: jc21/nginx-proxy-manager:latest
    container_name: npm
    ports:
      - 80:80
      - 443:443
      - 81:81
    volumes:
      - /opt/npm/data:/data
      - /opt/npm/letsencrypt:/etc/letsencrypt
    restart: unless-stopped