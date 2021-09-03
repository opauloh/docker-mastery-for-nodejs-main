# Assignment, MTA - Migrating Traditional Apps (Traditional app = Pre-Docker App) :
# Add .dockerignore
# Create a Dockerfile
# Change Winston to transport to Console (from logging to a file to logging to screen, so Docker can capture the logs)
# Tips:
# See README.md for app details
# Image shouldn't include in, out, node_modules or logs directories
# Change winston to console: winston.transports.Console (in cases we are not allowed to change code, we can symlink logs to stdout or stderr)
# Bind-mont in and out dirs
# set CHARCOAL_FACTOR to 0.1 (When we run the docker image, we can change the CHARCOAL_FACTOR to another value)
# no .gif files in the image

# last updated node 8 with alpine 
FROM node:8.9.4-alpine

# Install graphicsmagick
RUN apk update && apk add --no-cache graphicsmagick

# We don't want everything at the root of the image, to not mix with the operating system files
WORKDIR /app/

# Copying package.json and package-lock.json if any
COPY package*.json . /app/

# Install dependencies
RUN npm install --only=production && npm cache clean --force

# Set Environmental Variables
ENV CHARCOAL_FACTOR=0.1

# Copying files
COPY . /app/

# Run the app
CMD ["node", "index.js"]

# command to build the image:
# docker build -t assignment-mta .
# command to run the docker image:
# docker run -e CHARCOAL_FACTOR=0.1
# command to run with interactive shell:
# docker run -it assignment-mta sh
# -t : Allocate a pseudo-tty
# -i : Keep STDIN open even if not attached
# To copy images from docker to local machine:
# run docker ps to get the container name
# docker ps
# run docker cp, specifying the name, and the path to the file:
# docker cp awesome_pasteur:/app/out /home/paulo
# Bind mount the directory to the container: $(pwd) is the current directory
# docker run -v $(pwd)/in:/app/in -v $(pwd)/out:/app/out assignment-mta