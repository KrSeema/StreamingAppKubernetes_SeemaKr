FROM node:18 AS base

WORKDIR /app/streamingService

COPY streamingService/package*.json ./
#RUN npm install --production
RUN npm ci --omit=dev

COPY streamingService/. ./
ENV NODE_ENV=production
EXPOSE 3002

CMD ["npm", "run", "start"]
