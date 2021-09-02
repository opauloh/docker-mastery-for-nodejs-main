# Assignment: 3 stages: prod, dev and test
# Prod has no devDependencies and runs node
# Dev includes devDependencies and runs nodemon
# Test includes devDependencies and runs npm test
# Build all three stages with unique tags using --target
# Goal: Don't repeat lines, only execute copy once, and npm commands only once
FROM node:10-slim as prod
ENV NODE_ENV=production
WORKDIR /app

COPY package.json package-lock.json* ./
RUN npm install --production && npm cache clean --force
COPY . .
CMD ["node", "./bin/www"]

FROM prod as dev
ENV NODE_ENV=development
RUN npm install --only=dev
COPY . .
CMD ["npm", "run", "dev"]

FROM dev as test
ENV NODE_ENV=development
COPY . .
CMD ["npm", "test"]

# to Build prod:
# docker build -t assignment-multi:prod --target prod .
# Running:
# docker run --init -p 3001:3000 -d assignment-multi:prod

# to Build dev:
# docker build -t assignment-multi:dev --target dev .
# Running:
# docker run --init -p 3002:3000 -d assignment-multi:dev

# to Build test:
# docker build -t assignment-multi:test --target test .
# Running:
# docker run assignment-multi:test