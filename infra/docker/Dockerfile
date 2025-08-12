### STAGE 1:BUILD ###
# Defining a node image to be used as giving it an alias of "build"
# Which version of Node image to use depends on project dependencies
# This is needed to build and compile our code
# while generating the docker image
FROM node:12.14-alpine AS build
# Create a Virtual directory inside the docker image
WORKDIR /dist/src/app
# ENV NODE_OPTIONS=--openssl-legacy-provider
# Copy files to virtual directory
# COPY package.json package-lock.json ./
# Run command in Virtual directory
RUN npm cache clean --force
# Copy files from local machine to virtual directory in docker image
COPY ../../ .
RUN npm install --global lite-server
RUN npm install
RUN npm rebuild node-sass
# RUN npm run build-locale:test
RUN npm run build-local-locale && cp -r jsonTranslations/en/ dist/AppsRegistry/

EXPOSE 80
CMD ["lite-server", "-c", "lite-server.config.js"]
