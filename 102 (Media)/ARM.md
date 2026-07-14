services:
  arm:
    image: arm-igpu:local
    container_name: arm
    hostname: arm

    privileged: true
    restart: unless-stopped

    devices:
      - /dev/sr0:/dev/sr0
      - /dev/sg0:/dev/sg0
      - /dev/dri/renderD128:/dev/dri/renderD128

    group_add:
      - "993"

    ports:
      - "8080:8080"

    environment:
      TZ: Europe/Stockholm
      ARM_UID: "1000"
      ARM_GID: "1000"
      LIBVA_DRIVER_NAME: iHD

    volumes:
      - /opt/docker/arm/config:/etc/arm/config
      - /opt/docker/arm/home:/home/arm
      - /media:/media
