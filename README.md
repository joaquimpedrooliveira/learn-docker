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


