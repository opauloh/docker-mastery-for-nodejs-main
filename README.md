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

## Dockerfile

- [Best practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

### FROM

- This is the raw machine you will be using, usually it will come with `node` and `npm` installed, or even `yarn`, so prefer machines that comes with your core application installed.
- Tip: Prefer even versions of `node`, because even are stable releases and odd are more experimental, all LTS (Long Term Support) versions are even. So, don't use :latest tag.
- Check the [Releases](https://nodejs.org/en/about/releases/) page for the latest LTS Version
- On [Docker Hub](https://hub.docker.com/_/node) all oficial images use `Debian` as their base image, which means we get `apt get` package manager, core utilities like `openssl`, and `bash` shell. These `Debian` images can be big for convenience, so focus on `Alpine` for most new projects, and just use `Debian` if migrating from a pre-container world, and then consider moving to `Alpine` later. Because `Alpine` is a very small base image, that has only the minimal tools for `node` to get started.
- Caveats: `Debian` distributions will have more base OS packages that might be needed for some `npm` packages depencencies, with `Alpine` we need to make sure its supports those packages, be aware about edge cases where we won't have support in `Alpine` Package Manager

![](screenshots/screenshot-20210827095958.png)

- All this images are what we can specify on the `FROM` line, so we can for example:

```Dockerfile
FROM node:16
# Because there is a single 16 version

FROM node:16-alpine
# Chooose alpine for a much smaller variant, (About 8x less)

# Or
FROM node:16-alpine3.11
# ...and so on
```

- Also you will notice some `slim`, versions, they are smaller versions of debian, it's not recommend to use it, unless you have a high reason to do so, because that's what `Alpine` is for, and it's the industry standard for smaller images. Altough, if you can't use `Alpine`, might worth to check for `slim` versions.
- `onbuild` images are old, don't use it, because are problematic due to the fact your `Dockerfile` will execute commands from the image's `Dockerfile`, before executing from yours.
- `Alpine` is more secure focuses due to the fact there are less out-of-the-box, that means less pottential risk, but be aware that is falls on [CVE (Common Vulnerability) scanning](https://kubedex.com/follow-up-container-scanning-comparison/)
- Worth noting that `Debian`/`Ubuntu` images are not that huge, it's about `~100MB`, so if the convenience worth, go for it and don't worry with space.

### COPY

- most of times you will use `COPY`, it copies from one place to another,

### ADD

- `ADD` does many stuff, like download files from the internet, untar any files it sees in local directory, never use `ADD` for copying files (it can do it, but you shouldn't), for copying use `COPY` instead!

### WORKDIR

- You shoudn't use `cd` or `mkdir` inside `Dockerfile`, instead use `WORKDIR` that does both.
- it will make the directory if it's not there

_One caveat, is that if you need specific permissions on a directory when it's created mkdir might be better_

### CMD

- Use `node` to run your commands, not `npm`:

```Dockerfile
CMD ["node", "./bin/www"]
```

- One of the reason, i's because `npm` requires another application to run, that means, if you run with `npm`, instead of having just `node` running, you will have `npm` running, and `node` as a subprocess, and that adds complexity and an unnecessary layer
- Other reason, is not literal on what is happening, it's better when `Dockerfile` tells exactly what is happening when it launches, and when it's something like `npm start` that means `npm` will arbitrary call another command, so is not literal on it's intention, so it's harder to debug.
- Other reason, `npm` doesn't work well as an init process, that's because in Linux and containers, there's always a main process that everything launches from, so, if we start something that launches something else, we get sort of tree structures, and one main problem would be handle the signal termination. `npm` does not pass signals correctly to `node`, so, it tends to just improperly shut down the container. In this case, keeping `node` as the main process is simpler and allows to use **Direct Singnaling**.
