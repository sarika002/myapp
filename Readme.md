## How to create a multi-stage NestJS Docker Deployment?

Start by creating the following files in the project's root directory:

- `Dockerfile` - This file will be responsible for importing the Docker images, divide them into `development` and `production` environments, copying all of our files and install npm dependencies

- `docker-compose.yml` - This file will be responsible for defining our containers, required images for the app other services, storage volumes, environment variables, etc...


Open the `Dockerfile` and add the following code inside:

```sh
FROM node:12.19.0-alpine3.9 AS development

WORKDIR /usr/src/app

COPY package*.json ./

RUN npm install glob rimraf

RUN npm install --only=development

COPY . .

RUN npm run build

FROM node:12.19.0-alpine3.9 as production

ARG NODE_ENV=production
ENV NODE_ENV=${NODE_ENV}

WORKDIR /usr/src/app

COPY package*.json ./

RUN npm install --only=production

COPY . .

COPY --from=development /usr/src/app/dist ./dist

CMD ["node", "dist/main"]
```

Open the `docker-compose.yml` file and add the following code:


```sh
version: '3.8'

services:
    dev:
        container_name: nestjs_api_dev
        image: nestjs-api-dev:1.0.0
        build:
            context: .
            target: development
            dockerfile: ./Dockerfile
        command: npm run start:debug
        ports:
            - 3000:3000
            - 9229:9229
        networks:
            - nesjs-network
        volumes:
            - .:/usr/src/app
            - /usr/src/app/node_modules
        restart: unless-stopped
    prod:
        container_name: nestjs_api_prod
        image: nestjs-api-prod:1.0.0
        build:
            context: .
            target: production
            dockerfile: ./Dockerfile
        command: npm run start:prod
        ports:
            - 3000:3000
            - 9229:9229
        networks:
            - nesjs-network
        volumes:
            - .:/usr/src/app
            - /usr/src/app/node_modules
        restart: unless-stopped

networks:
    nesjs-network:
```

## Running the Docker containers

Now that we have defined our Docker files, we can run our app solely on Docker.

To start our app, write the following command in your terminal:

```sh
docker-compose up dev
```

This will start it in development mode. 

And to start our app in production mode,run the following command in your terminal:

```sh
docker-compose up prod
```

