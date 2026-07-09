services:
  seerr:
    image: ghcr.io/seerr-team/seerr:latest
    container_name: seerr
    environment:
      - TZ=Europe/Stockholm
    volumes:
      - /opt/seerr:/app/config
    ports:
      - 5055:5055
    restart: unless-stopped