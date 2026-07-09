services:
  wizarr:
    image: ghcr.io/wizarrrr/wizarr:latest
    container_name: wizarr
    environment:
      - TZ=Europe/Stockholm
    volumes:
      - /opt/wizarr:/data/database
    ports:
      - 5690:5690
    restart: unless-stopped