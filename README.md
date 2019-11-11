# Complete Guide for Linking Nodejs MongoDB Without Docker Compose

## Prerequisite
  - Clone Nodejs project form [here](https://github.com/shekhar-raval/node-express-es8) for getting started quickly (You Can user your project if you have already created)
  - Docker Installed on your machine [docker](https://docs.docker.com/install/linux/docker-ce/ubuntu/)
  - Basic Commandline knowledge
  - Basic Docker Knowledge

## Steps
 1. **Create Dockerfile**
 1. **Build Image using Dockerfile**
 1. **Use Docker Networks for multi-container setup**
    1. **Creating Network**
    1. **Launching Mongo Image into Network**
    1. **Launching App into Network**
1. **Launching App into Network**

## Create Dockerfile for Nodejs
```bash
# Pull Nodejs Image from Docker Hub
FROM node:12

# Make working Directory
RUN mkdir -p /home/node/app/node_modules && chown -R node:node /home/node/app

# cd into working directory
WORKDIR /home/node/app

# Copy package.json file in directory
COPY package*.json ./

# Set user
USER node

# Run npm install
RUN npm install

# Copy all Code to Directory
COPY . .

# Exposing PORT 8000 - (Expose port on which your app is Running)
EXPOSE 8000

# Start Node Appliction
CMD ["npm", "start"]
```
## Build Image using Dockerfile
Change directory into the folder containing **Dockerfile** and execute following command to build image tagged with name `node-api`
```bash
# Flag -t is for tagging image (optional)
docker build -t node-api .
```
# Use Docker networks for multi-container setup

## Creating Network
Assuming you want to name your network **node-network**, run this command:
```bash
docker network create --driver=bridge node-network
```
Verify by getting node-network details or list of all networks:

```bash
docker network inspect node-network
docker network ls
```
You would see something like this:
```bash
docker network ls
NETWORK ID          NAME                  DRIVER              SCOPE
e9f653fffa25        node-network          bridge              local
cd768d87acb1        bridge                bridge              local
0cd7db8df819        host                  host                local
8f4db39bd202        none                  null                local
```
## Launching Mongo Image into Network
Next, launch vanilla mongo image in **node-network** (or whatever name you used for your network). The name of the container **node-mongo** will become the host name to access it from our app:

```bash
docker run --rm -it --net=node-network --name node-mongo mongo
```
Note: If you didn’t have mongo image, Docker will download it for you. It’ll happen just once, the first time.

Leave this mongo running. Open a new terminal.

## Launching App into Network
This is my command to launch my Node app in a production mode and connect to my mongo container which is in the same network (node-network):
```bash
docker run --rm -t --net=node-network --name node-api -e NODE_ENV=production -e MONGO_URI="mongodb://node-mongo:27017/node-boilerplate" -p 80:8000 {Your node-api image ID}
```
Content **{}** must be replaced with your app image ID from the previous step when you did docker build .. If you forgot the app image ID, then run **```docker images```** and look up the ID.

## Inspecting Network
```bash
docker network inspect node-network
```
Inspecting your network with **```docker network inspect banking-api-network```** will show you have 2 running containers there:
```bash
 ...
        "Containers": {
            "02ff9bb083484a0fe2abb63ec79e0a78f9cac0d31440374f9bb2ee8995930414": {
                "Name": "node-mongo",
                "EndpointID": "0fa2612ebc14ed7af097f7287e013802e844005fe66a979dfe6cfb1c08336080",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            },
            "3836f4042c5d3b16a565b1f68eb5690e062e5472a09caf563bc9f11efd9ab167": {
                "Name": "node-api",
                "EndpointID": "d6ae871a94553dab1fcd6660185be4029a28c80c893ef1450df8cad20add583e",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            }
        },
    ...
```

## Thank You For Reading
