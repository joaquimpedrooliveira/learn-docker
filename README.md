# learn-docker
Aplicação de exemplo do Docker do tutorial: https://docs.docker.com/get-started/part2/

## Pré-requisitos
* Docker instalado e configurado corretamente

## Parte 2 - Containers
* Cria a imagem com o nome amigável de "friendlyhello":
```shell
docker build -t friendlyhello .
```

* Lista as imagens criadas:
```shell
docker images
```

* Executa a imagem, mapeando a porta 80 do container para a porta 4000 do host:
```shell
docker run -p 4000:80 friendlyhello
```

* Executa a imagem **em background** e retorna o id do container:
```shell
docker run -d -p 4000:80 friendlyhello
```

* Exibe os containers em execução:
```shell
docker ps
```

* Para a execução do container:
```shell
docker stop <CONTAINER ID>`

### Comandos úteis
```shell
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
## Parte 3 - Serviços

* Inicia o serviço descrito no docker-compose.yml:
```shell
docker swarm init
docker stack deploy -c docker-compose.yml getstartedlab
```

* Ver os 5 containers rodando:
```shell
docker stack ps getstartedlab
```
* Gerar requisições para http://localhost e observar que o ID do servidor muda, pois está sendo atendido por um container diferente

* Parando o serviço:
```shell
docker stack rm getstartedlab
```

### Comandos úteis
```shell
docker stack ls              # List all running applications on this Docker host
docker stack deploy -c <composefile> <appname>  # Run the specified Compose file
docker stack services <appname>       # List the services associated with an app
docker stack ps <appname>   # List the running containers associated with an app
docker stack rm <appname>                             # Tear down an application
```

## Parte 4 - Swarms

* No Linux, instalar a ferramenta [docker-machine](https://docs.docker.com/machine/), para poder criar máquinas virtuais com docker. Instruções de instalação: https://github.com/docker/machine/releases

```shell
$ curl -L https://github.com/docker/machine/releases/download/v0.11.0/docker-machine-`uname -s`-`uname -m` >/tmp/docker-machine &&
    chmod +x /tmp/docker-machine &&
    sudo cp /tmp/docker-machine /usr/local/bin/docker-machine
```

* Criar duas máquinas virtuais para o cluster:
```shell
docker-machine create --driver virtualbox myvm1
docker-machine create --driver virtualbox myvm2
```

* Obter o IP da myvm1:
```shell
docker-machine ls
```

* Tornar a máquina **myvm1** o manager do cluster:
```shell
docker-machine ssh myvm1 "docker swarm init --advertise-addr <IP DA MYVM1>:2377"`
```

Este comando retorna o comando de join que os workers devem usar para entrar no cluster.

* Executar o comando acima na **myvm2**
```shell
docker-machine ssh myvm2 "docker swarm join \
     --token <TOKEN DO COMANDO ANTERIOR> \
     <IP DA MYVM1>:2377"
```

* Copie o arquivo `docker-compose.yml` para o manager do cluster:

```shell
docker-machine scp docker-compose.yml myvm1:~
```

* Ordene o manager para implantar a aplicação no cluster:

```shell
docker-machine ssh myvm1 "docker stack deploy -c docker-compose.yml getstartedlab"
```

* Verifique que alguns containers estão rodando na **myvm1** e outros na **myvm2**

```shell
docker-machine ssh myvm1 "docker stack ps getstartedlab"
```

* Acesse a aplicação pelo IP das duas VMs (http://192.168.99.100 e http://192.168.99.101) e verifique que a mesma aplicação está disponível.

* Caso queira parar o cluster

```shell
docker-machine ssh myvm1 "docker stack rm getstartedlab"
```

### Comandos úteis

```shell
docker-machine create --driver virtualbox myvm1 # Create a VM (Mac, Win7, Linux)
docker-machine create -d hyperv --hyperv-virtual-switch "myswitch" myvm1 # Win10
docker-machine env myvm1                # View basic information about your node
docker-machine ssh myvm1 "docker node ls"         # List the nodes in your swarm
docker-machine ssh myvm1 "docker node inspect <node ID>"        # Inspect a node
docker-machine ssh myvm1 "docker swarm join-token -q worker"   # View join token
docker-machine ssh myvm1   # Open an SSH session with the VM; type "exit" to end
docker-machine ssh myvm2 "docker swarm leave"  # Make the worker leave the swarm
docker-machine ssh myvm1 "docker swarm leave -f" # Make master leave, kill swarm
docker-machine start myvm1            # Start a VM that is currently not running
docker-machine stop $(docker-machine ls -q)               # Stop all running VMs
docker-machine rm $(docker-machine ls -q) # Delete all VMs and their disk images
docker-machine scp docker-compose.yml myvm1:~     # Copy file to node's home dir
docker-machine ssh myvm1 "docker stack deploy -c <file> <app>"   # Deploy an app
```

## Parte 4 - Stacks

### Pré-requisitos
O cluster da seção anterior deve estar funcionando

* Edite o arquivo `docker-compose.yml` e adicionar o novo serviço após o serviço `web` e antes de `networks:`:
```yaml
visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]
    networks:
      - webnet
```

* Atualize o arquivo `docker-compose.yml` no manager e realize o novo deploy:
```shell
docker-machine scp docker-compose.yml myvm1:~
docker-machine ssh myvm1 "docker stack deploy -c docker-compose.yml getstartedlab"
```

* Verifique o nome serviço, que lista os containers e os nós do cluster, executando:

`http://192.68.99.101:8080`

* Observe que o serviço do visualizador possui somente 1 container, que roda no manager. A aplicação possui vários, distribuídos no cluster:
```shell
docker-machine ssh myvm1 "docker stack ps getstartedlab"
```

* Inclua um novo serviço para o Redis no docker-compose.yml`, adicionando este trecho após a seção do serviço `visualizer`:
```yaml
 redis:
    image: redis
    ports:
      - "6379:6739"
    volumes:
      - ./data:/data
    deploy:
      placement:
        constraints: [node.role == manager]
    networks:
      - webnet
```

* Crie os diretórios de volume do Redis e reinicie o serviço:
```shell
docker-machine ssh myvm1 "mkdir ./data"
docker-machine scp docker-compose.yml myvm1:~
docker-machine ssh myvm1 "docker stack deploy -c docker-compose.yml getstartedlab"
```
* Acesse a aplicação e verifique que o contador de visitantes agora é persistente: 
`http://192.168.99.100/`
