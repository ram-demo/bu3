FROM node:18.14.2-slim
COPY ./build /app
WORKDIR /app
RUN npm install
CMD [ "npm", "run", "prod" ]