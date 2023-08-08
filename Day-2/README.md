# Dockerfile, Entrypoint e Healthcheck

### O que é um Dockerfile?

É um arquivo de texto que contém todos os comandos necessários para criar uma imagem Docker.

### O que é um Entrypoint?

É um comando que é executado quando o container é iniciado.

### O que é um Healthcheck?

É um comando que verifica se o container está saudável.

### Criando um Dockerfile

```bash
# Define a imagem base
FROM ubuntu:18.04

# Define o mantenedor da imagem
LABEL maintainer="raps_rnb@hotmail.com"

# Executa um comando
RUN apt-get update && apt-get install -y nginx curl

# Define a porta que o container irá expor
EXPOSE 80

# Define o diretório de trabalho
WORKDIR /var/www/html

# Adiciona arquivos para o container
ADD node_exporter-1.6.0.linux-amd64.tar.gz /root/node-exporter

# Copia um arquivo do host para o container (Não esquecer de criar o arquivo index.html)
COPY index.html .

# Adiciona uma variável de ambiente
ENV APP_VERSION 1.0.0

# Exectando o Entrypoint
ENTRYPOINT ["nginx"]

# Define o comando que será executado quando o container for iniciado
CMD ["-g", "daemon off;"]

# Exectando o Healthcheck
HEALTHCHECK --interval=5s --timeout=3s CMD curl -f http://localhost/ || exit 1
```

### Alguns comandos úteis

```bash
ADD # Copia novos arquivos, diretórios, arquivos TAR ou arquivos remotos e os adiciona ao filesystem do container.

CMD # Executa um comando, fornecendo padrões para um container em execução. Diferentemente do RUN, que executa o comando no momento em que está "buildando" a imagem, o CMD irá fazê-lo somente quando o container é iniciado.

LABEL # Adiciona metadados à imagem, como versão, descrição e fabricante.

COPY # Copia novos arquivos e diretórios e os adiciona ao filesystem do container.

ENTRYPOINT # Permite que você configure um container para rodar um executável. Quando esse executável for finalizado, o container também será.

ENV # Informa variáveis de ambiente ao container.

EXPOSE # Informa qual porta o container estará ouvindo.

FROM # Indica qual imagem será utilizada como base. Ela precisa ser a primeira linha do dockerfile.

MAINTAINER # Autor da imagem.

RUN # Executa qualquer comando em uma nova camada no topo da imagem e "commita" as alterações. Essas alterações você poderá utilizar nas próximas instruções de seu dockerfile.

USER # Determina qual usuário será utilizado na imagem. Por default é o root.

VOLUME # Permite a criação de um ponto de montagem no container.

WORKDIR # Responsável por mudar do diretório "/" (raiz) para o especificado nele
```

Para maiores detalhes sobre como criar imagens, veja essa apresentação criada pelo Jeferson: https://www.slideshare.net/jfnredes/images-deep-dive.

---

## Desafio Day 2

Criar um container com a imagem base do Linux Alpine, rodando Nginx e com healthcheck. O nome do container " giropops-web" e deve rodar na porta 80.

```bash
vim Dockerfile
```

```bash
FROM alpine:3.18
LABEL maintainer="raps_rnb@hotmail.com"
RUN apk add --no-cache nginx curl
EXPOSE 80
ENTRYPOINT ["nginx"]
CMD ["-g", "daemon off;"]
HEALTHCHECK --interval=5s --timeout=3s CMD curl -f http://localhost/ || exit 1
```

```bash
docker build -t giropops-web:1.0 .
```

```bash
docker container run -d -p 80:80 --name giropops-web giropops-web:1.0
```

## MultiStage no Dockerfile

O multisatge é uma funcionalidade do Dockerfile que permite que você construa uma imagem em múltiplas etapas. Isso é útil quando você precisa de uma imagem com várias ferramentas para construir o seu projeto, mas não quer que a imagem final fique "gorda" (com muitos MBs).

#### Criando o Dockerfile:

```bash
FROM golang:1.20 as buildando
WORKDIR /app
COPY teste.go .
RUN go mod init hello
RUN go build -o /app/hello

FROM alpine:3.18
COPY --from=buildando /app/hello /app/hello
CMD ["/app/hello"]
```

#### Criando o arquivo main.go:

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello Giropops!")
}
```

#### Executando o build:

```bash
docker build -t go-teste:1.0 .
```

```bash
docker container run -ti go-teste:1.0
```

## ARG e ENV no Dockerfile

### O que é ARG (Argument)?

É uma variável que pode ser passada ao construir a imagem com o parâmetro --build-arg ou no próprio Dockerfile, mas não fica disponível durante a execução do container. 

### O que é ENV (Environment)?

É uma variável de ambiente que pode ser utilizada durante a construção da imagem e também durante a execução do container.

## Volumes no Dockerfile

### O que é um volume?

É um diretório que é montado dentro do container e que pode ser utilizado para persistir dados.

### Criando um volume no Dockerfile

```bash

```bash
FROM alpine:3.18
LABEL maintainer="raps_rnb@hotmail.com"
RUN apk add --no-cache nginx curl
EXPOSE 80
ENTRYPOINT ["nginx"]
CMD ["-g", "daemon off;"]
HEALTHCHECK --interval=5s --timeout=3s CMD curl -f http://localhost/ || exit 1
VOLUME /var/www/html
ARG APP_VERSION=1.0.0
ENV APP_VERSION=${APP_VERSION}
```

## DockerHub - Login, Pull e Push

### O que é o DockerHub?

É o repositório oficial de imagens Docker.

### Como fazer o login no DockerHub?

```bash
docker login
```

### Como fazer o pull de uma imagem do DockerHub?

```bash
docker pull nginx
```

### Como fazer o push de uma imagem para o DockerHub?

```bash
docker push {nome_de_usuario}/{nome_da_imagem}:{tag}
```

## Docker Distribution

### O que é o Docker Distribution?

É um serviço que permite que você tenha um repositório privado de imagens Docker.

### Como instalar o Docker Distribution?

```bash
docker run -d -p 5000:5000 --restart=always --name registry registry:2
```

### Como fazer o push (upload) de uma imagem para o Docker Distribution?

```bash
docker tag nginx localhost:5000/nginx
docker push localhost:5000/nginx
```

### Como fazer o pull (download) de uma imagem do Docker Distribution?

```bash
docker pull localhost:5000/nginx
```

Para maiores detalhes, visite a página do projeto no GitHub: https://github.com/docker/distribution

## Desafio 2 do Day 2


1. Criar um conta no Docker Hub, caso ainda não possua uma.

2. Criar uma conta no Github, caso ainda não possua uma.

3. Criar um Dockerfile para criar uma imagem de container para a nossa App
    
    - O nome da imagem deve ser SEU_USUARIO_NO_DOCKER_HUB/linuxtips-giropops-senhas:1.0

4. Fazer o push da imagem para o Docker Hub, essa imagem deve ser pública

5. Criar um repo no Github chamado LINUXtips-Giropops-Senhas, esse repo deve ser público

6. Fazer o push do cógido da App e o Dockerfile

7. Criar um container utilizando a imagem criada
    - O nome do container deve ser giropops-senhas
    - Você precisa deixar o container rodando

8. O Redis precisa ser um container

Dica: Preste atenção no uso de variável de ambiente, precisamos ter a variável REDIS_HOST no container. Use sua criatividade!

```bash
git clone https://github.com/badtuxx/giropops-senhas.git

```bash
vim Dockerfile
```

```bash
FROM alpine:3.13 as base
COPY ./giropops-senhas /giropops-senhas
RUN apk add --no-cache py3-pip && pip install --no-cache-dir -r /giropops-senhas/requirements.txt

FROM base
COPY --from=base /giropops-senhas /giropops-senhas
WORKDIR /giropops-senhas
ENV REDIS_HOST=localhost
RUN rm -rf /root/.cache /root/.cargo /usr/local/include /usr/local/share
CMD flask run --host=0.0.0.0
```

```bash
docker build -t raphaelborges/linuxtips-giropops-senhas:1.0 .
```

```bash
docker push raphaelborges/linuxtips-giropops-senhas:1.0
```

```bash
git clone https://github.com/Rapha-Borges/LINUXtips-Giropops-Senhas.git
```

```bash
docker container run -d -p 6379:6379 --name redis redis:7.0.12
```

```bash
docker container run -d -p 5000:5000 --name giropops-senhas --link redis:7.0.12 raphaelborges/linuxtips-giropops-senhas:1.0 
```
