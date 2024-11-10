README

# Creacion de VM con UbuntuServer para virtualizar contenedores con Docker Compose.

Paso 1:

  Descargar la imagen de UbuntuServer 24.04.

  Crear la VM con VirtalBox:

      RAM=4GB
      Storage=200GB
      CPU=2
      Network Ethernet:
          Paravirtualized Network (bridge adapter, eno1)

Paso 2:

  Instalar Docker Compose en la VM:

    sudo apt install docker.io docker-compose -y
    sudo systemctl enable --now docker
    sudo usermod -aG docker $USER # agrega mi usuario al grupo Docker para evitar sudo


  Instalar Openssh-server:

    sudo apt update && sudo apt install openssh-server


Paso 3: (Pi-hole)

  Para organizar creamos una carpeta /home/pi/pihole y dentro el archivo docker-compose.yml.

  version: "3.9"
  services:
    pihole:
      image: pihole/pihole:latest
      container_name: pihole
      environment:
        TZ: 'America/Argentina/Buenos_Aires'
        WEBPASSWORD: 'Lisandro@10'
      volumes:
        - ./etc-pihole:/etc/pihole
        - ./etc-dnsmasq.d:/etc/dnsmasq.d
      ports:
        - "8888:80" # exponemos Pi-hole en el port :8888
        - "53:53/tcp"
        - "53:53/udp"
      restart: unless-stopped

Paso 4:

  Ingresamos a la carpeta creada:
    cd /home/pi/pihole 
  Iniciamos el container:
    docker-compose up -d # 10.0.1.121:8888/admin

Paso 5:

  Configuramos el router para que redirija el tráfico a la ip de la VM:

    [Viking@MK CASA 30] > ip dns set servers=10.0.1.121 allow-remote-requests=yes

    [Viking@MK CASA 30] > ip dhcp-server network set dns-server=10.0.1.121

Paso 6:

  En Pi-hole:

    - settings > DNS # check for dns-server y salvamos los cambios.

Paso 7: 'docker plex y transmission' # 10.0.1.121:32400/web 10.0.1.121:9091

  # sudo mkdir -p /home/pi/plex
  # sudo nano docker-compose.yml

  version: "3.3"
  services:
    plex:
      image: linuxserver/plex:latest
      container_name: plex
      environment:
        - PUID=1000
        - PGID=1000
        - TZ=America/Argentina/Buenos_Aires
      volumes:
        - /home/pi/plex/config:/config
        - /home/pi/transmission/downloads:/media  # Carpeta compartida con Transmission
      network_mode: host  # Necesario para escaneo DLNA
      restart: unless-stopped

  # sudo mkdir -p /home/pi/transmission
  # sudo mkdir -p /home/pi/transmission/config
  # sudo mkdir -p /home/pi/transmission/downloads
  # sudo nano docker-compose.yml

  version: "3.3"
  services:
    transmission:
      image: linuxserver/transmission:latest
      container_name: transmission
      environment:
        - PUID=1000  # ID del usuario (pi)
        - PGID=1000  # ID del grupo (pi)
        - TZ=America/Argentina/Buenos_Aires  # Ajusta tu zona horaria
        - TRANSMISSION_WEB_UI=combustion  # Interfaz web mejorada
      volumes:
        - /home/pi/transmission/config:/config
        - /home/pi/transmission/downloads:/downloads
        - /home/pi/plex:/plex  # Carpeta de Plex como acceso directo
      ports:
        - "9091:9091"  # Interfaz web de Transmission
        - "51413:51413"  # Puerto de conexión Torrent (TCP)
        - "51413:51413/udp"
      restart: unless-stopped

Paso 8: Verificar permisos

  # sudo chown -R blackserver:blackserver /home/pi/transmission/downloads
  # sudo chmod -R 775 /home/pi/transmission/downloads

Paso 9: Verificar descargas

  # Accedemos a PLEX 10.0.1.121:32400/web 

  Agregamos libreria a /media/complete 

NOTA: Para actualizar los contenedores usar, dentro del directorio de cada contenedor.

    docker-compose down 
    docker-compose pull
    docker-compose up -d