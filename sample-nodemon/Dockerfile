FROM node:10.15-slim

ENV NODE_ENV=production

WORKDIR /app

COPY package.json package-lock*.json ./

RUN npm install && npm cache clean --force

# Update Path variable, so we are able to run any of the tools we installed as dev-dependencies without having to specify the full path
ENV PATH /app/node_modules/.bin:$PATH

COPY . .

CMD ["node", "./bin/www"]
