version: "3.8"

services:
  crafty:
    image: registry.gitlab.com/crafty-controller/crafty-4:latest
    container_name: crafty
    restart: unless-stopped

    ports:
      - "8443:8443"

      # Minecraft TCP
      - "25565-25585:25565-25585"

      # Minecraft UDP (voice/mods)
      - "25565-25585:25565-25585/udp"

    volumes:
      - /opt/crafty/backups:/crafty/backups
      - /opt/crafty/logs:/crafty/logs
      - /opt/crafty/config:/crafty/app/config
      - /opt/crafty/import:/crafty/import
      - /opt/crafty/servers:/crafty/servers

    environment:
      - TZ=Europe/Stockholm