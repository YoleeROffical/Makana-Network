services:
  prowlarr:
    image: lscr.io/linuxserver/prowlarr:latest
    container_name: prowlarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Stockholm
    volumes:
      - /opt/prowlarr:/config
    ports:
      - 9696:9696
    restart: unless-stopped

  flaresolverr:
    image: ghcr.io/flaresolverr/flaresolverr:latest
    container_name: flaresolverr
    environment:
      - LOG_LEVEL=info
      - TZ=Europe/Stockholm
    ports:
      - 8191:8191
    restart: unless-stopped

  sonarr:
    image: lscr.io/linuxserver/sonarr:latest
    container_name: sonarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Stockholm
    volumes:
      - /opt/sonarr:/config
      - /media:/media
    ports:
      - 8989:8989
    restart: unless-stopped

  radarr:
    image: lscr.io/linuxserver/radarr:latest
    container_name: radarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Stockholm
    volumes:
      - /opt/radarr:/config
      - /media:/media
    ports:
      - 7878:7878
    restart: unless-stopped

  bazarr:
    image: lscr.io/linuxserver/bazarr:latest
    container_name: bazarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Stockholm
    volumes:
      - /opt/bazarr:/config
      - /media:/media
    ports:
      - 6767:6767
    restart: unless-stopped

  cleanuparr:
    image: ghcr.io/cleanuparr/cleanuparr:latest
    container_name: cleanuparr
    volumes:
      - /opt/cleanuparr:/config
    environment:
      - TZ=Europe/Stockholm
    ports:
      - 11011:11011
    restart: unless-stopped

  profilarr:
    image: ghcr.io/dictionarry-hub/profilarr:latest
    container_name: profilarr
    environment:
      - TZ=Europe/Stockholm
    volumes:
      - /opt/profilarr:/config
    ports:
      - 6868:6868
    restart: unless-stopped