#dockerfile
FROM node:18-alpine

WORKDIR /app

#copiar dependencias
COPY package*.json ./ 

RUN npm install

#copiar codigo
COPY . .

EXPOSE 4000

CMD ["node", "server.js"]