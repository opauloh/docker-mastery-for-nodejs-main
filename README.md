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
- Remember that line order is important, because it builds naturally from top to bottom.
- Also remember, if you have layers that don't change often, let them at the top of the file, otherwise, it won't benefit of caching.
- [Dockerfile reference](https://docs.docker.com/engine/reference/builder/#dockerfile-file)

_Follow this [Assignment](assignment-dockerfile) for a reference on building Dockerfile without docker-compose_

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
- A tip for production is to use patch images, so you guarantee they will work as expected. i.e. `node:10.15.3-alpine3.11-slim`

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

### Dockerfile for a custom image

- We can provide custom node images for OS that are not the officials, like Debian or Alpine, for this we can search docker hub for other images we want, i.e Centos, Ubuntu, Red Hat, etc...

_Follow this [CentOS Dockerfile](centos-node/Dockerfile) for reference_

### User

- Note that on official images, the apps will be running as root in the container (root in container is not the same as host, because containers are restricted from the rest of the system).
- But even they not being the same, we want to reduce security risk inside containers, a way of doing this is run apps in the container as a non root user. So if someone manages to break trough the app, they wouldn't even be root in the container, just a standard user account with very little privileges.
- A good news is that the official node image has a Node user built in, it's just not enabled by default, that's because of the various errors that can occur related to permission, commands like `apt-get` or `yum`, or `npm install -g` , so you need to consider that. Advice is always try to enable least privilege, so you will want to enable the least privileged user after the `apt`/`apk` commands or `npm i -g`, and before the `npm i`.
- Problems that might happen when using node user instead of root, is if your application expect to write or install while your app is running.
- Be aware that only `RUN`, `CMD` and `ENTRYPOINT` commands will behave as the `node` user, everything else will be as `root`, so you might found problems on directory and file specific commands like `WORKDIR`, that's why is important to remember to apply permission to the node user when creating directories.
- Once we elect the node user, any `docker compose exec` will use the node user by default, if we need to change that, we can use the -u root with any exec command. Useful for adding packages without denying access.
- [Diferences between chmod and chwon](https://www.unixtutorial.org/difference-between-chmod-and-chown/)
- [USER reference](https://docs.docker.com/engine/reference/builder/#user)
- [COPY reference](https://docs.docker.com/engine/reference/builder/#copy)

```Dockerfile
FROM node:10-slim

EXPOSE 3000

WORKDIR /node

COPY package*.json ./

# Since Docker has already created the Node user in the upstream from image, we just need to enable it with this command in Dockerfile:
USER node

# Creating directory and setting permission to the node user and group, so we can do things like `run npm install`, write cache files, etc...
RUN mkdir app && chown -R node:node .

# Now we can install as the node user
RUN npm install && npm cache clean --force

WORKDIR /node/app

# We could do:
# RUN chown -R node:node .
# instead, but the chown command adds another layer to the image without deleting the previous layers.
# That means, that the final container image contains both layers. Thus, the size of the container adds the size of both folders: the original working directory for the root user, and the second working directory with permissions for the normal user.
COPY --chown=node:node . .
# when using the --chown flag, it avoids the extra layer, so it will result in smaller size of the image.

CMD ["node", "app.js"]
```

### How cache busting works

- Whenever a line is changed on the Dockerfile, that line, and the line below it, will have to be rebuilt. That's the normal behaviour, but it's good to avoid it whenever is not necessary.
- Like for example if Dockerfile expose this port, and it never change, put them at the top of the file, so it will be cached.

```Dockerfile
EXPOSE 3000
```

### Optimization

- Avoid copying all the source files before npm install, because any time a source file change, it will bust the run line for npm install and have to rerun that line every single time it copies the source files. And we definitely don't want that.

```Dockerfile
FROM node:10-alpine
EXPOSE 3000
WORKDIR /usr/src/app

# Avoid that:
COPY . .
RUN npm install && npm cache clean --force

CMD ["node", "./bin/www"]
```

- To avoid that we usually just have a copy for package.json and package-lock.json, and then we run npm install, and then we copy in . . (that means everything else). This will save a lot of time.
- `npm install --production` will install only the production dependencies, and not the dev dependencies.

### Tips

- We can copy `package-lock.json` with `*` which means it will copy if is there, but won't fail if it's not there.
- This is good in case `package-lock.json` is auto generated, and we don't want to copy it.

```Dockerfile
# without the * it would fail the build in case it's doesn't exist
COPY package.json package-lock.json* ./
```

- Keep only one `apt-get` or `yum` per Dockerfile, and put it higher in the Dockerfile. Also remember to pin versions of packages you're installing.
- Also keep all installs together, or cache busting will be against you.

```Dockerfile
FROM node:10-alpine
EXPOSE 3000

RUN apt-get update && apt-get install curl

WORKDIR /usr/src/app

COPY package.json package-lock.json* ./

RUN npm install && npm cache clean --force

# Don't do this, the correct would be to put at the top with the curl install
# Because if we do that, and then change anything in package.json, it will bust only the cache for httping, but without apt-get being updated, because the lines before were cache busted.
RUN apt-get install httping

COPY . .

CMD ["node", "./bin/www"]
```

### Node process management

- In Docker containers we don't need to use tools for managing processes, like `nodemon`, `forever`, `pm2`, etc... Docker is gonna do it for us.
  -- Altough, we can use nodemon for dev as file watcher.
- Docker can manage the whole process, it can start, stop, pause, restart and health check, to restart the process if is not healthy.
- Docker manages multiple "replicas" of the same process, so if we have a process that is running, and we restart it, it will restart all the replicas.

### Process Identifier - PID

- PID 1 is the first process in a system (or container) (AKA init)
- Init process has two jobs: reap zombie processes and pass signals to sub-processes (like shutdown).
- Zombie process is not a issue with node, usually we're just going to have one node process and subprocesses.
- Docker uses Linux signals to stop app (SIGINT, SIGTERM, SIGKILL) and to restart it (SIGHUP).
  -- We never want to deal with SIGKILL, because it's killing the container, without having a chance to stop it or respond.
  -- SIGINT is used when we ctrl+c
  -- SIGTERM is used when we do a docker container stop
  -- The problem: `npm` does not handle this signals at all, that's why we should use `node` instead of `npm`, it doesn't handle by default but we can add code to handle it.
- Docker provides a init `PID 1` replacement option (it's called [tini](https://github.com/krallin/tini)), but this is a sort of backup. Just pass `--init` flag option to `docker run`

### Proper Node shutdown options

- Temp: use --init to fix ctrl+c issue for now. (it will use tini to shutdown the process)
  -- Use it if you are attempting to stop a container but it's taking about 10 seconds to stop, so it's because Docker is not being able to properly shutdown the process. (It's using a default of waiting for 10 seconds and then killing it)

```bash
docker run --init -d nodeapp
```

- Workaround: add tini to the image (Use only if can't really change the app, but need a proper shutdown solution)

```Dockerfile
# That's in case the app is not handling shutdown correctly, and we can't make changes in the code, we can use tini to shutdown the process.
# So it won't wait 10 seconds and KILL our app.
# Add tini to your Dockerfile, then use it in CMD
RUN apk add --no-cache tini
# Trick here  is to use entrypoint instead of cmd, that's because entrypoint will run first and wrap the cmd
# It's kind of what Docker is doing in the background if we do --init from command line.
# But we use that so people don't have to use --init, and maybe forgot to use that option.
ENTRYPOINT ["/sbin/tini", "--"]
CMD ["node", "./bin/www"]
```

- Production: make the app capture SIGINT for proper exit (Graceful shutdown)

```js
// This only shutdowns the app, is not handling connections nor waiting for jobs to finish.
// quit on ctrl-c when running docker in terminal
process.on("SIGINT", function onSigint() {
  console.info(
    "Got SIGINT (aka ctrl-c in docker). Graceful shutdown ",
    new Date().toISOString()
  );
  shutdown();
});

// quit properly on docker stop
process.on("SIGTERM", function onSigterm() {
  console.info(
    "Got SIGTERM (docker container stop). Graceful shutdown ",
    new Date().toISOString()
  );
  shutdown();
});

// shut down server
function shutdown() {
  // NOTE: server.close is for express based apps
  // If using hapi, use `server.stop`
  server.close(function onServerClosed(err) {
    if (err) {
      console.error(err);
      process.exitCode = 1;
    }
    process.exit();
  });
}
```

- An example of when the machine it's not handling SIGINT properly, we hit ctrl+c, and the app is not handling it.
  ![](screenshots/screenshot-20210901175132.png)

- Example of initializing with tini (PID 1 is tini):
  ![](screenshots/screenshot-20210902105952.png)

- Example of initializing with Docker's default tini (--init):
  ![](screenshots/screenshot-20210902110309.png)

## [MultiStage](https://docs.docker.com/develop/develop-images/multistage-build/) Dockerfile

```Dockerfile
# Create an alias from node to prod, so we can refer to it in the same Dockerfile using prod
FROM node as prod

# Basic node npm stuff
ENV NODE_ENV=production
COPY package.json package-lock.json* ./
RUN npm install && npm cache clean --force
COPY . .
CMD ["node", "./bin/www"]

# Creating another image based on the first one (prod)
# We can have as many as FROM as we want, and they can depend of each other be unrelated.
FROM prod as dev
ENV NODE_ENV=development
# Changing cmd to development environment
CMD ["nodemon", "./bin/www", "--inspect=0.0.0.0:9229"]
```

- To build dev image from dev stage:

```bash
docker build -t my-app .
```

- To build prod image from prod stage:

```bash
docker build -t my-app:prod --target prod .
```

_The end result would be two images, my-app that is the dev image, and a my-app with tag of prod, only with the production part_

### More scenarios for multi-stage Dockerfile

- Add a test stage that runs `npm test`
- Have CI build --target test stage before building prod
- Add `npm install --only=development` to dev stage
- COPY only once, don't COPY code into dev stage
- [Advanced multi stage patterns](https://medium.com/@tonistiigi/advanced-multi-stage-build-patterns-6f741b852fae)
- [Multi Stage Assignment](sample-multi-stage/Dockerfile)

## Buildkit

BuildKit, it's a new way to build your images, and a replacement "build engine" that is now an optional feature with quite a few benefits over traditional docker build commands.

- https://github.com/moby/buildkit
- https://www.youtube.com/watch?v=kkpQ_UZn2uo&ab_channel=Docker
- https://medium.com/@tonistiigi/advanced-multi-stage-build-patterns-6f741b852fae
- https://www.udemy.com/course/docker-mastery-for-nodejs/learn/lecture/13236858#overview

### Using BuildKit to Enable SSH Keys for Private NPM Repositories

If your Node project has private git repos for node modules, it'll need a particular setup so SSH can be used when building the image.

The previous solution before BuildKit was:

- Use multi-stage builds.
- COPY a decrypted-private-key in to an early stage where npm install is run.
- COPY the node_modules from that stage to a new image that doesn't include the key.

That solution worked if you're ok with having the ssh key stored in your local docker engine images, but it wasn't ideal, and didn't work with encrypted ssh keys that required a passphrase.

The new way is to use BuildKit with the ssh-agent feature, and is much more secure:

- Setup ssh-agent and your keys on the host OS like normal.
- Add this as the first line in your Dockerfile: # syntax = docker/dockerfile:experimental
- Start your Dockerfile npm install line with this: RUN --mount=type=ssh
- Run docker build with --ssh default as an additional option to enable the feature for that build.

_Remember you can't yet use this with docker-compose build, so you'd need to build your images manually with docker build and then use that image name in your docker-compose.yml_

- [Sample](sample-buildkit-ssh/Dockerfile)

### Using BuildKit to Reuse NPM Cache

If you ever change a Dockerfile line before the RUN npm install line, or you change your package.json or lock file, Docker will need to re-run npm install on the next build. Docker, by default, won't re-use package manager download caches like the NPM cache.

If you have large `package.json` files with slow dependency installs due to large downloads, you can speed up rebuilds by enabling the BuildKit caching feature on specific directories inside your docker builds. I've seen this speed up re-builds by over 50% with large dependency trees.

Remember you can't yet use this with docker-compose build, so you'd need to build your images manually with docker build and then use that image name in your docker-compose.yml

To set this up for re-using the NPM download cache:

- add this as the first line in your Dockerfile: # syntax = docker/dockerfile:experimental
- Start your Dockerfile npm install line with: RUN --mount=type=cache,target=/root/.npm/\_cacache

- [Sample](sample-buildkit-cache/Dockerfile)
