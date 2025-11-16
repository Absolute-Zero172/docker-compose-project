# Docker Compose Lab
*Calvin Wasilevich*  
*2025.11.16*

## Installing Docker

I opted to use my existing fedora virtual machine to install docker. I used the [instructions provided by docker](https://docs.docker.com/engine/install/fedora/) to install the engine onto fedora systems. These instructions will be summarized here.

1. Remove any previous docker installations that may have come with the distribution. These are often out-of-date, and it is best to fetch the official docker application from the docker website. The possible other commands are removed using the command:
```bash
sudo dnf remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```
![Screenshot removing all other packages](./Screenshots/uninstall%20others.png)

2. The docker rpm repository holds the official docker installation files. It must be added to dnf in order for dnf to look to the docker repository to download and update packages. Add the docker rpm repository with the commands:
```bash
sudo dnf -y install dnf-plugins-core
sudo dnf-3 config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo
```
![Screenshot adding new repository](./Screenshots/add%20docker%20repo.png)

3. Install docker (and supporting packages) from the new repo with:
```bash
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
![Screenshot installing docker](./Screenshots/install%20docker.png)

4. Docker engine must be enabled to constantly run and run on startup. This is completed using systemctl with the command:
```bash
sudo systemctl enable --now docker
```
![Screenshot showing started docker engine](./Screenshots/enable%20docker%20engine.png)

5. Verify the docker installation with:
```bash
sudo docker run hello-world
```
![Screenshot showing valid docker install](./Screenshots/verify%20docker.png)

## Create A Container

For this project, I opted to install [Jellyfin](https://jellyfin.org/), an open-source project for local video and audio streaming. The steps for installation are:

1. Locate the provided docker compose file
    - The docker compose file is found in the [Jellyfin Documentation](https://jellyfin.org/docs/general/installation/container/#installation-instructions)
    - This file is copied into the jellyfin directory

```yaml
services:
  jellyfin:
    image: jellyfin/jellyfin
    container_name: jellyfin
    user: uid:gid
    ports:
      - 8096:8096/tcp
      - 7359:7359/udp
    volumes:
      - /path/to/config:/config
      - /path/to/cache:/cache
      - type: bind
        source: /path/to/media
        target: /media
      - type: bind
        source: /path/to/media2
        target: /media2
        read_only: true
      # Optional - extra fonts to be used during transcoding with subtitle burn-in
      - type: bind
        source: /path/to/fonts
        target: /usr/local/share/fonts/custom
        read_only: true
    restart: 'unless-stopped'
    # Optional - alternative address used for autodiscovery
    environment:
      - JELLYFIN_PublishedServerUrl=http://example.com
    # Optional - may be necessary for docker healthcheck to pass if running in host network mode
    extra_hosts:
      - 'host.docker.internal:host-gateway'
```

2. Modify and update docker compose file
    - All optional lines were removed, as this proof-of-concept does not need them
        - `extra_hosts`, `environment`, and the extra fonts volume
    - Subdirectories of the main jellyfin directory were made for config and media, and their paths were added to the docker compose file. The file structure was:
    ![Jellyfin directory file structure](./Screenshots/file%20structure.png)
    - The user line was set to 1000:1000 indicating the "calvin" user and the "calvin" group (the default user on this virtual machine)

```yaml
services:
  jellyfin:
    image: jellyfin/jellyfin
    container_name: jellyfin
    user: 1000:1000
    ports:
      - 8096:8096/tcp
      - 7359:7359/udp
    volumes:
      - /home/calvin/jellyfin/config:/config
      - /home/calvin/jellyfin/cache:/cache
      - type: bind
        source: /home/calvin/jellyfin/media
        target: /media
      - type: bind
        source: /home/calvin/jellyfin/media2
        target: /media2
        read_only: true
    restart: 'unless-stopped'
```

3. Make the docker container with `sudo docker compose up -d`
    - `-d` allows the container to run in detached mode

![Docker container running](./Screenshots/jellyfin%20running.png)

4. Login to jellyfin at the web address `http://localhost:8096` with any web browser (shown here is Firefox)
    - Complete setup and ensure jellyfin is working properly

![Jellyfin startup](./Screenshots/jellyfin%20startup.png)
![Jellyfin functional](./Screenshots/jellyfin%20functional.png)