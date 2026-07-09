services:
  homepage:
    image: ghcr.io/gethomepage/homepage:latest
    container_name: homepage
    environment:
      - TZ=Europe/Stockholm
      - HOMEPAGE_ALLOWED_HOSTS=192.168.1.33:3000,yoleer.eu
    volumes:
      - /opt/homepage:/app/config
    ports:
      - 3000:3000
    restart: unless-stopped