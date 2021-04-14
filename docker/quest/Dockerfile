FROM node:10

# Create app directory
WORKDIR /usr/src/app

# Bundle app source
COPY . .

RUN npm install

EXPOSE 3000
CMD [ "node", "src/000.js" ]