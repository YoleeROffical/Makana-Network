services:
  jellyfin:
    image: lscr.io/linuxserver/jellyfin:10.11.11
    container_name: jellyfin

    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Stockholm
      - LIBVA_DRIVER_NAME=i965

    devices:
      - /dev/dri:/dev/dri

    volumes:
      - /opt/jellyfin/config:/config
      - /opt/jellyfin/cache:/cache
      - /media/content:/media/content
      - /opt/jellyfin/custom-cont-init.d:/custom-cont-init.d:ro

    ports:
      - "8096:8096"

    restart: unless-stopped