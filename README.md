# Overte Domain Server Docker Images

Contained in this repo are two docker images which are used to build and run the Overte domain server as a docker image.

Also contained is a docker-compose.yml file to run the domain server on your server using docker-compose.

The Domain server image is published to Docker Hub for amd64 and aarch64 architecture. Pull it from here: https://hub.docker.com/r/overte/overte-server

# Caveats

The assignment-client ports need to be published on the same port as they are listening on, since the domain-server doesn't know which ports are published and tells connecting clients the wrong ports otherwise.
This will work in most cases because of hole-punching, but more strict firewall setups will fail.

# Build Instructions

To build the domain server runtime image (Dockerfile.runtime), you must first build the overte server builder image (Dockerfile.build).

- Build the builder image by running the following command (lengthy process): 
```sh 
docker build --no-cache --build-arg "TAG=2024.06.1" -t domain-server-builder -f ./Dockerfile.build .
```
- Upon completion of the above, build the runtime container with the following command:
```sh 
docker build --no-cache -t overte/overte-server:2024.06.1-amd64 -f ./Dockerfile.runtime .
```

- Once the build is completed, you will be able to run the domain server either with docker-compose (see the contained file and change the `image` to `domain-server`), or by running the following:
```sh
docker run -d --name overte-server -p 40100-40102:40100-40102 -p 40100-40102:40100-40102/udp -p 48000-48006:48000-48006/udp -v $(pwd)/logs:/var/log -v $(pwd)/data:/root/.local/share/Overte --restart unless-stopped overte/overte-server:2024.06.1-amd64
```


# Connection issues troubleshooting

On some systems, using **host network** is required. Otherwise, it might lead to issues with getting stuck in the void or lost connection.

- Docker cli example: add `--network=host`
- Docker-compose example: add `network_mode: "host"`

# Pushing a release

When pushing a new release, don't forget to create a manifest that contains aarch64 and amd64:
```bash
docker push overte/overte-server:2024.06.1-amd64
docker push overte/overte-server:2024.06.1-aarch64
docker manifest create overte/overte-server:2024.06.1 overte/overte-server:2024.06.1-amd64 overte/overte-server:2024.06.1-aarch64
docker manifest push overte/overte-server:2024.06.1
docker manifest rm overte/overte-server:latest
docker manifest create overte/overte-server:latest overte/overte-server:2024.06.1-amd64 overte/overte-server:2024.06.1-aarch64
docker manifest push overte/overte-server:latest
```
