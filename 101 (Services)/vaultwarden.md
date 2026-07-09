services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    environment:
      - TZ=Europe/Stockholm
      - WEBSOCKET_ENABLED=true
     # - ADMIN_TOKEN=******
    volumes:
      - /opt/vaultwarden:/data
    ports:
      - 80:80
      - 3012:3012
    restart: unless-stopped