services:
  nextcloud:
    image: lscr.io/linuxserver/nextcloud:latest
    container_name: nextcloud
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Stockholm
      - PHP_MEMORY_LIMIT=512M
      - PHP_UPLOAD_LIMIT=10G
    volumes:
      - /opt/nextcloud:/config
      - /media/nextcloud:/data
    ports:
      - 443:443
      - 80:80
    depends_on:
      - nextcloud-db
      - nextcloud-redis
    restart: unless-stopped

  nextcloud-db:
    image: mariadb:latest
    container_name: nextcloud-db
    environment:
      - MYSQL_ROOT_PASSWORD=****
      - MYSQL_DATABASE=****
      - MYSQL_USER=****
      - MYSQL_PASSWORD=****
    volumes:
      - /opt/nextcloud-db:/var/lib/mysql
    restart: unless-stopped

  nextcloud-redis:
    image: redis:alpine
    container_name: nextcloud-redis
    restart: unless-stopped