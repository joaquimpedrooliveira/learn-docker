# learn-docker
Aplicação de exemplo do Docker do tutorial: https://docs.docker.com/get-started/part2/

## Pré-requisitos
* Docker instalado e configurado corretamente

## Comandos básicos
* Cria a imagem com o nome amigável de "friendlyhello":

`docker build -t friendlyhello .`

* Lista as imagens criadas:

`docker images`

* Executa a imagem, mapeando a porta 80 do container para a porta 4000 do host:

`docker run -p 4000:80 friendlyhello`

* Executa a imagem **em background** e retorna o id do container:

`docker run -d -p 4000:80 friendlyhello`

* Exibe os containers em execução:

`docker ps`

* Para a execução do container:

`docker stop <CONTAINER ID>`

## Comandos úteis
```
docker build -t friendlyname .  # Create image using this directory's Dockerfile
docker run -p 4000:80 friendlyname  # Run "friendlyname" mapping port 4000 to 80
docker run -d -p 4000:80 friendlyname         # Same thing, but in detached mode
docker ps                                 # See a list of all running containers
docker stop <hash>                     # Gracefully stop the specified container
docker ps -a           # See a list of all containers, even the ones not running
docker kill <hash>                   # Force shutdown of the specified container
docker rm <hash>              # Remove the specified container from this machine
docker rm $(docker ps -a -q)           # Remove all containers from this machine
docker images -a                               # Show all images on this machine
docker rmi <imagename>            # Remove the specified image from this machine
docker rmi $(docker images -q)             # Remove all images from this machine
docker login             # Log in this CLI session using your Docker credentials
docker tag <image> username/repository:tag  # Tag <image> for upload to registry
docker push username/repository:tag            # Upload tagged image to registry
docker run username/repository:tag                   # Run image from a registry
```
