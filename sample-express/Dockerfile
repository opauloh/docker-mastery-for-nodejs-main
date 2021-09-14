FROM node:10.15-slim

ENV NODE_ENV=production

WORKDIR /app

COPY package.json package-lock*.json ./

RUN npm install && npm cache clean --force

COPY . .

CMD ["node", "./bin/www"]

# Command to install dependencies using docker-compose
# docker-compose run express npm install
# Then run the app using docker-compose
# docker-compose up