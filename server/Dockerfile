FROM node:20-alpine

WORKDIR /usr/src/app

COPY package*.json ./

RUN npm ci --only=production

COPY --chown=node:node . .
USER node

CMD ["node", "server.js"]