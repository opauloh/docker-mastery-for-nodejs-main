# Docker Mastery for Node.js Projects From a Docker Captain

This repo is for use in my Udemy Course https://www.bretfisher.com/docker-mastery-for-nodejs

# My Notes

## [Alpine](https://wiki.alpinelinux.org/wiki/Alpine_Linux_package_management)

Because Alpine Linux is designed to run from RAM, package management involves two phases:

- Installing / Upgrading / Deleting packages on a running system.
- Restoring a system to a previously configured state (e.g. after reboot), including all previously installed packages and locally modified configuration files. (RAM-Based Installs Only)
- `apk` is the tool used to install, upgrade, or delete software on a running system.
- `lbu` is the tool used to capture the data necessary to restore a system to a previously configured state.

## Docker Compose

- `docker-compose` CLI is a substitute for docker CLI
- `docker-compose` CLI was not designed for production, instead it was designed around developer workflows
- `docker-compose.yml` is a standard convention for name definition
- many `docker` commands == `docker-compose`
- `docker-compose` CLI and YAML versions differ

### YAML

yml is the extension for YAML, a common configuration file format, it uses:

- `:` for key/value pairs
- `-` for lists
- only spaces, no tabs

i.e:

```yml
version: "2.0"

services: # services is one or more container based on a single image
  web:
    image: sample-01 # the image we want it to make when we build this directory
    build: . # this is to where build to, . is a shorthand for the current directory
    ports: # ports is a list because we can have many ports published
      - "3000:3000"
```

- **Compose YAML v2 vs v3** - v2 focus on single-node, dev/test and v3 focus on multi-node and orchestration, so if not using Swarm, Kubernetes, prefer v2 - [Reference](https://github.com/docker/docker.github.io/pull/7593)
- [Online YAML Validator](https://codebeautify.org/yaml-validator)

### Installing docker-compose on Linux

```bash
pip install docker-compose
```

### docker-compose up

this command will do everything it need to make the docker image live, it includes:

- build/pull image(s) if missing
- create volume / network / container(s)
- starts container(s) in foreground (`docker-compose up -d` to detach to background)
- `--build` to always build
- If you build a container, bring it up, then stop it, edit the Dockerfile, and up again, the changes on the Dockerfile won't reflect. And that's because we didn't build the image, because it's already built! So we force it to build with: `docker-compose up -d --build`

### docker-compose down

this command will do everything to stop:

- stop and delete network / container(s) (it preserves data)
- use `-v` to delete volumes (data), so you can use `docker-compose down -v`, to do a clean uninstall of everything including data
- it won't delete the images by default, you have to use `docker-compose down --rmi` to do so. Check for `docker-compose down --help` for more commands

### docker-compose build

- just build/rebuild image(s)
- you can use `docker-compose build --no-cache` to build a image from scratch (don't need technically to use `docker-compose down --rmi`)

### docker-compose CLI basics

- Commands takes "service" name as option (not the container or image name), it's just for building images and not messing around with `up` or `down`
- `docker-compose build` - just build/rebuild image(s)
- `docker-compose stop` - just stop containers, don't delete
- `docker-compose ps` - list services, different from `docker ps`, it shows stopped and running containers, and in an easier format, will show ports open, and which errors might have happenend if a container has stopped
- `docker-compose push` - push all the images in your compose file up to the registry
- `docker-compose logs` - see all logs for all of the containers running or pass the name of the service to filter, you can use `docker-compose logs -f` to follow the logs, and let the output streamming
- `docker-compose exec` - Execute commands inside running containers (it takes the services name, i.e `docker-compose exec web sh`)

### Docker build workflow

- If we want some package to be available on the image, we can run a command to use it's package manager to install it, we can edit `Dockerfile` and add the `RUN` command:

```yml
# we want curl to be installed on the image
RUN apk add --update curl
```

### Ports

How ports publishing works:

- The left side is what the host, your machine will use.
- The right side is what is used in the container.

So, given a node application who start a express server in port 3000, you can:

```yml
"3000:3000" # You can access https://localhost:3000 from your browser and see the application
"8080:3000" # You can access https://localhost:8080 from your browser and see the application
"3000:8080" # You will get error, because the docker is running anything on port 8080
```
