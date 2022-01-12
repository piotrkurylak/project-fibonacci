FROM node:16-alpine3.14 as builder

ENV NODE_ENV=production

WORKDIR /app

COPY --chown=node:node package* ./

RUN npm ci --only=production

COPY --chown=node:node ./ ./

RUN npm run build

## Multi-stage build
FROM nginx:1.20-alpine

EXPOSE 3000

COPY ./nginx/default.conf /etc/nginx/conf.d/default.conf

COPY --from=builder /app/build /usr/share/nginx/html

