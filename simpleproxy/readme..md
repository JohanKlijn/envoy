# Simple example to start with Envoy

This example is used to get started with Envoy. In this example we create a docker container running a .NET Core API on port 8080 (and do not expose it, so only available inside the docker network 'envoymesh') and and Envoy accepting request on port 90 (only available inside the docker network 'envoymesh'). Port 90 of the Envoy container is mapped to port 8080 on the host machine. All the request made on port 80 on the Envoy container are forwarded to the the rest api (http://api1:8080)/. This is hard-coded in the Envoy configuration. So this means the api is only accessable from the outside by going through the Envoy proxy (which is the sidecar in this example).

## Validate if the API1 container is working
This page assumes you are running all the commands from powershell.

Build the docker image

``` powershell
cd simpleproxy\netsolution
docker build -f Dockerfile-api1 -t jklijn/api1 .
```

Run the docker container
``` powershell
docker run -d --rm --name api1 johanklijn/api1
```
-d: run docker container in detached mode, so we can continue using the current powershell shell.
--rm remove the container when the container is stopped, so we keep our disk clean.
--name: name the container api1 so we can easily reference the container in other commands.

Run the following commands to see if the API1 is running in side the container on port 8080 (this port should not be accesable from the maching which host your container, because we didn't expose it in our dockerfile)

``` powershell
docker exec -it api1 bash
```
The command above will start a bash session in the container.

``` bash
curl http://localhost:80/api/values
exit
```
The curl command should return a response from the rest api.

Now navigate on you laptop to http://localhost:80/api/values. It should return an connection refused error, because we did't map the port to a port on the local system.

Stop the container using the following command
``` powershell
docker stop api1
```

## Run the API1 container and use the Envoy container as a sidecar proxy which makes the API1 accesable from the outside.

Run the following command to start-up the containers (API and Envoy) using their own network to communicate with each other.
``` powershell
docker-compose up --build -d
```
--build: build the containers first.
-d: run docker containers in detached mode.

Run the following command to get information from the REST API through the Envoy sidecar proxy:
``` powershell
Invoke-RestMethod http://localhost:8000/service
```