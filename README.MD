# This is the `FORK` of official TeaSpeak docker repository
This repository holds Dockerfile to build the TeaSpeak Server Image

## Table of Content
1. [Server Images](#server-images)
    1. [Build and Run - Debian Flavoured](#debian-427mb-14-layers)
    2. [EnvVars and Build-Args](#envvars-and-build-args-server)
    3. [Run the Server via `docker-compose`](#run-the-server-via-docker-compose)
    4. [View the Logs and get the Token](#view-the-logs-and-token)
    5. [Where to store data](#where-to-store-data)
2. [License](#license)

#### Debian (~427MB, 14 Layers)
<details>
    <summary>Build and Run Debian</summary>
  
```shell script
# Build the Image with the latest server
docker build -t teaspeak/server:latest-debian .

# alternatively with server version <x.y.z>
# docker build --build-arg SERVER_VERSION=<x.y.z> -t teaspeak/server:<x.y.z>-debian .

# run the Debian based server image
docker run -d \
  -v teaspeak_logs:/ts/logs \
  -v teaspeak_db:/ts/database \
  -v teaspeak_files:/ts/files \
  -v teaspeak_certs:/ts/certs \
  -v teaspeak_config:/ts/config \
  -v teaspeak_crash_dumps:/ts/crash_dumps \
  -p 9987:9987/tcp -p 9987:9987/udp -p 10101:10101/tcp -p 30303:30303/tcp \
  --restart=unless-stopped --name teaspeak-server teaspeak/server:latest-debian
```
</details>

### EnvVars and Build-args (server)

| Variable | Default | Description |
|:--------:|:-------:|:----------- |
|  **TZ**  | `Europe/Berlin` | Sets the timezone within the container. |

| Build-Arg | Default | Description |
|:---------:|:-------:|:----------- |
| **SERVER_VERSION** | *blank* | This build argument sets the server version to be installed in the resulting image. If its left *blank* the latest released server verison will be installed. |
|  **uid**  | `4242`  | The default user id of the `teaspeak` user used in the container |
|  **gid**  | `4242`  | The default group id of the `teaspeak` user used in the container |

### Run the server via docker-compose
<details>
    <summary>docker-compose.yml</summary>
    
```yaml
version: '3.7'
services:
  teaspeak-server:
    image: ghcr.io/pikachok/teaserver:latest
    environment:
      - TZ=Europe/Amsterdam
    ports:
      - "9987:9987/udp"
      - "9987:9987/tcp"
      - "10101:10101/tcp"
      - "30303:30303/tcp"
    volumes:
      - type: volume
        source: teaspeak_certs
        target: /ts/certs
      - type: volume
        source: teaspeak_config
        target: /ts/config
      - type: volume
        source: teaspeak_db
        target: /ts/database
      - type: volume
        source: teaspeak_files
        target: /ts/files
      - type: volume
        source: teaspeak_logs
        target: /ts/logs
      - type: volume
        source: teaspeak_crash_dumps
        target: /ts/crash_dumps
    restart: unless-stopped

volumes:
  teaspeak_certs:
  teaspeak_crash_dumps:
  teaspeak_config:
  teaspeak_db:
  teaspeak_files:
  teaspeak_logs:
```
</details>

To run this please ensure you have docker-compose [installed](https://docs.docker.com/compose/install/).
```shell script
docker-compose up -d
```

### View the Logs and Token
To view the docker container logs and to find the server token you have to run the following command:
```shell script
docker logs <container-name|container-id>

# to get the container name or id run the following
docker ps 

# eg. if the server gets started as displayed in the above section
docker logs teaspeak-server

# and if you want to follow the logs (ctrl+c to stop following)
docker logs -f teaspeak-server
```

### Where to Store Data

Important note: There are several ways to store data used by applications that run in Docker containers.
I encourage users of the `teaspeak` image to familiarize themselves with the options available, including:

- Let Docker manage the storage of your database data [by writing the database files to disk on the host system using its own internal volume management](https://docs.docker.com/engine/tutorials/dockervolumes/#adding-a-data-volume).
This is the default and is easy and fairly transparent to the user. 
The downside is that the files may be hard to locate for tools and applications that run directly on the host system, i.e. outside containers.
- Create a data directory on the host system (outside the container) and [mount this to a directory visible from inside the container](https://docs.docker.com/engine/tutorials/dockervolumes/#mount-a-host-directory-as-a-data-volume). 
This places the database files in a known location on the host system, and makes it easy for tools and applications on the host system to access the files. 
The downside is that the user needs to make sure that the directory exists, and that e.g. directory permissions and other security mechanisms on the host system are set up correctly.

The Docker documentation is a good starting point for understanding the different storage options and variations, and there are multiple blogs and forum postings that discuss and give advice in this area.

# License

As with Docker image, these likely also contain other software which may be under other licenses (such as Bash, etc from the base distribution, along with any direct or indirect dependencies of the primary software being contained).

As for any pre-built image usage, it is the image user's responsibility to ensure that any use of this image complies with any relevant licenses for all software contained within.
