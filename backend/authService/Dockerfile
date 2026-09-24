FROM node:18 AS base

WORKDIR /app

COPY authService/package*.json ./
RUN npm ci --omit=dev

COPY . .

ENV NODE_ENV=production
EXPOSE 3001

CMD ["npm", "run", "start"]
