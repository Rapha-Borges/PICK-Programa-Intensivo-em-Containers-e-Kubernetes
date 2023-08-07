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

CMD # Executa um comando. Diferentemente do RUN, que executa o comando no momento em que está "buildando" a imagem, o CMD irá fazê-lo somente quando o container é iniciado.

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

### Desafio Day 2

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

---

### MultiStage no Dockerfile

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

