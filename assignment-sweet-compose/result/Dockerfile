FROM node:10-alpine3.11

WORKDIR /app

# if nodemon wasn't installed, and we dindn't have access to install it, we could install via compose
RUN npm install -g nodemon
COPY package*.json /app/
RUN npm install \
 && npm ls \
 && npm cache clean --force \
 && mv /app/node_modules /node_modules
COPY . /app

ENV PORT 80
EXPOSE 80

CMD ["node", "server.js"]
