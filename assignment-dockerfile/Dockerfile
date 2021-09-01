# Assignment: Make a Dockerfile for existing Node app (container image migration)
# First step, leanr about the app (look at package.json, look at the app.js, look ar ports) 
# Second step, see if it's a legacy app that does not handle shutdown properly, so you're going to use tini
# fill in the blanks to dockerize this node app
# https://hub.docker.com/_/node
FROM node:10-alpine

# Por figured out from app.js
# Also if the host is 0.0.0.0 is good, because localhost only works on the container
EXPOSE 3000
# Adding tini to handle shutdown properly https://github.com/krallin/tini
RUN apk add --no-cache tini

# Workdir can be whatever we want, but be consistent across projects on your company
WORKDIR /app

COPY package.json package-lock.json* ./

# npm cache clean will ensure there's no leftover files from previous builds that were downloaded from npm package repositories
# Double ampersand && means the first command have to succeed for the second to run, that's something we definitely want to happen in containers
RUN npm install && npm cache clean --force

# Copying everything to the container
COPY . .

ENTRYPOINT ["/sbin/tini", "--"]

CMD ["node", "app.js"]

# Bulding with tag, from your terminal:
# docker build -t assignment1 .
# Running:
# docker run -p 3000:3000 assignment1
# and check on http://localhost:3000
# or if you want to check on http://localhost:
# docker run -p 80:3000 assignment1