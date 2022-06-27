# Testausdeploy
This project is still a concept!

# Development

## Architecture
This project will implement 3 services for the new LUMI server.
1. Docker auto deploy directly from a repository under the Testausserveri Github organization without any repository configuration
2. LXC container deployment for members' personal purposes to allow for strong enough segregation of users
3. REST API to trigger actions and provide statistics

The project will be organized in to 3 components:
1. The Manager, which implements all services of this project
2. The Guardian, which implements a service for the manager to update certain system configurations on the host system.
3. The Worker, which runs runs/restarts/builds containers

All of these services will be implemented in a single manager Docker container that will have the relevant directories mounted from the host system along with an interface on the host service to update the hosts's firewall.

## Services

### Docker auto deploy
The Docker auto deploy service will implement the following deploy pipeline.
1. Pull a repository from Github
    - Whitelist for allowed organizations
    - Image is pulled to a central directory on the host system
2. Accept a path as user input for the Docker-compose file in the pulled repository
3. Run docker-compose for the pulled repository
    - Use the Guardian to configure the host network if required
    - Docker-compose is ran with the Worker

Additionally the Docker auto deploy service will provide information about the running container(s) through the Manager's REST API.

### LXC container deployment
The LXC container deployment service will provide members with an LXC container for their personal purposes.

The following features must be implemented:
- Creation of containers
    - Install certain services in the container (OpenSSH etc.)
- Management of containers
    - Configure SSH credentials within the container
    - Use the Guardian to make the desired firewall configurations on the host system
- Container snapshots & backups

## Communication between services
The Internal Communication Protocol will use a secure WebSocket connection (for flexibility) to allow communication between processes.

Certificates shall be specific for each service, self-signed and stored locally. In addition, public key authentication shall be implemented to identify remote components.

The main reason to use WebSockets, even though they are not the most efficient way to communicate between processes (pipes exist), is to allow possible separation of all components, when more servers become available to us.

This protocol shall use simple JSON objects as their format which shall follow the following structure:

```json
{
    "command": "<The command to execute>",
    "data": any data to transmit (command specific)
}
```

Only one connection at a time shall be accepted by a server.

## Components

### Manager
The Manager communicates with all other components to execute desired actions as a proxy.

Additionally it implements the main entrypoint for the frontend (REST API)

The Manager will be implemented as a Docker container and the service written in Node.Js with the Express web server.

This service will implement the Internal Communications Protocol and be the proactive party of the communication pair in terms of initiating a connection.

Additionally the manager must poll deployed repositories for changes & store all current configuration and information about containers' ownership and status in a database (MongoDB) running in a separate container. This database shall also contain the IPs of the Worker and Guardian components (as in the future many instances of these could exist) and every Worker should have an assigned Guardian.

### Guardian
The Guardian will manage the firewall and other network configurations on the host system. Such as opening ports.

This service will be ran as a SystemD service on the host system with Node.JS.

This service will implement the Internal Communications Protocol.

### Worker
The worker will run all deployment related commands on the host system.

This service will be ran as a SystemD service on the host system with Node.JS.

This service will implement the Internal Communications Protocol.