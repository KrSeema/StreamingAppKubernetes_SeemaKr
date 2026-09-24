FROM node:18 AS base

WORKDIR /app/chatService

COPY chatService/package*.json ./
#RUN npm install --production
RUN npm ci --omit=dev

COPY chatService/. ./
ENV NODE_ENV=production
EXPOSE 3004

CMD ["npm", "run", "start"]
