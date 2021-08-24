# Docker Mastery for Node.js Projects From a Docker Captain

This repo is for use in my Udemy Course https://www.bretfisher.com/docker-mastery-for-nodejs

# My Notes

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

- **Compose YAML v2 vs v3** - v2 focus on single-node, dev/test and v3 focus on multi-node and orchestration, so if not using Swarm, Kubernetes, prefer v2

### Installing docker-compose on Linux

```bash
pip install docker-compose
```

### docker-compose up

this command will do everything it need to make the docker image live, it includes:

- build/pull image(s) if missing
- create volume / network / container(s)
- starts container(s) in foreground (-d to detach to background)
- `--build` to always build

### docker-compose down

this command will do everything it to stop:

- stop and delete network / container(s) (it preserves data)
- use `-v` to delete volumes (data), so you can use `docker-compose down -v`, to do a clean uninstall of everything including data

### docker-compose CLI basics

- Commands takes "service" name as option (not the container or image name), it's just for building images and not messing around with `up` or `down`
- `docker-compose build` - just build/rebuild image(s)
- `docker-compose stop` - just stop containers, don't delete
- `docker-compose ps` - list services, different from `docker ps`, it shows stopped and running containers, and in an easier format, will show ports open, and which errors might have happenend if a container has stopped
- `docker-compose push` - push all the images in your compose file up to the registry
- `docker-compose logs` - see all logs for all of the containers running or pass the name of the service to filter
- `docker-compose exec` - Execute commands inside running containers
