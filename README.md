# freeradius-server-image

Unofficial Docker images of [FreeRADIUS](https://github.com/FreeRADIUS/freeradius-server) v4 (the development branch), built for `linux/amd64` and `linux/arm64` and published as [`cayenneryo/freeradius-server`](https://hub.docker.com/r/cayenneryo/freeradius-server).

This repository holds no FreeRADIUS code. It pins an upstream commit and builds it with the Dockerfile that ships in the upstream tree (`scripts/docker/build/<dist>/Dockerfile.service`), unmodified. The images exist for [Koitohan](https://github.com/jolly-jive/Koitohan); they are not supported by the FreeRADIUS project.

## Tags

```
4.0.0-devel-ubuntu26-14fdfb3-r1
│           │        │       └ build number (rebuilds of the same commit)
│           │        └ upstream commit (first 7 hex digits)
│           └ base distribution
└ version prefix
```

- A published tag is never overwritten. A rebuild of the same commit picks up newer base image and distribution packages, so it gets a new build number.
- Tags without an `-rN` suffix, and `latest`, predate this repository and are `linux/arm64` only.
- The images are developer builds: `freeradius -v` prints `DEVELOPER BUILD`.

The full upstream commit is in the `org.opencontainers.image.revision` label:

```sh
docker inspect --format '{{ index .Config.Labels "org.opencontainers.image.revision" }}' cayenneryo/freeradius-server:<tag>
```

## Using the image

The server binary is `freeradius` and the configuration is in `/etc/freeradius`. The entrypoint passes arguments that start with `-` to `freeradius -f`, and accepts `radiusd` as an alias. The container starts as root; the server then switches to the `freerad` user (uid 101, gid 101).

```sh
docker run --rm cayenneryo/freeradius-server:<tag> -v     # version
docker run --rm cayenneryo/freeradius-server:<tag> -XC    # check the configuration
docker run --rm -p 1812:1812/udp -p 1813:1813/udp cayenneryo/freeradius-server:<tag> -X
```

## Publishing a new image

1. Edit `build.env`: set `UPSTREAM_REF` to the full commit SHA and reset `BUILD` to `1`, or raise `BUILD` to rebuild the same commit.
2. Open a pull request. The workflow builds and tests both architectures without pushing.
3. After merging, run the **Build and push** workflow by hand from `main` with **Push the image to Docker Hub** checked.

The workflow refuses to run if the tag already exists on Docker Hub.

### Repository settings

| Kind     | Name                 | Value                                              |
| -------- | -------------------- | -------------------------------------------------- |
| Variable | `DOCKERHUB_USERNAME` | Docker Hub user name                               |
| Secret   | `DOCKERHUB_TOKEN`    | Docker Hub access token with Read & Write scope    |

## License

The files in this repository are licensed under the Apache License 2.0; see [LICENSE](LICENSE).

The images contain FreeRADIUS, which is licensed under the GNU General Public License, version 2 or later. The source of each image is the upstream commit named in its tag and its `org.opencontainers.image.revision` label.
