FROM node:18 AS base

WORKDIR /app/adminService

COPY adminService/package*.json ./
#RUN npm install --production
RUN npm ci --omit=dev

COPY adminService/. ./
ENV NODE_ENV=production
EXPOSE 3003

CMD ["npm", "run", "start"]
