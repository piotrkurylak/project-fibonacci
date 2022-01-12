FROM node:16-alpine3.14

ENV NODE_ENV=production

WORKDIR /usr/src/app

COPY --chown=node:node package* ./

RUN npm ci --only=production

COPY --chown=node:node ./ ./

USER node

CMD ["npm", "run", "start"]