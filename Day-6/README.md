# Docker Compose

## O que é o Docker Compose?

O Docker Compose é uma ferramenta que permite a definição e execução de múltiplos containers. É possível definir as imagens, as redes, os volumes, as variáveis de ambiente, os limites de recursos, entre outros.

## Como instalar o Docker Compose?

O Docker Compose é instalado junto com o Docker Desktop ou pode ser instalado seguindo as instruções do [site oficial](https://docs.docker.com/compose/install/linux/).

## Como utilizar o Docker Compose?

O Docker Compose utiliza um arquivo chamado `docker-compose.yml` para definir os containers. O arquivo `docker-compose.yml` é um arquivo YAML que possui uma estrutura de chave e valor. É possível definir as imagens, as redes, os volumes, as variáveis de ambiente, os limites de recursos, entre outros.

```yaml
version: "3.8"
services:
  nginx:
    image: nginx:latest
    ports:
      - "8080:80"
    networks:
      - <nome>
```

## Como executar o Docker Compose?

Para executar o Docker Compose, basta executar o comando `docker-compose up` na mesma pasta do arquivo `docker-compose.yml`.

```bash
# Executar o Docker Compose
docker-compose up

# Executar o Docker Compose em background
docker-compose up -d

# Parar o Docker Compose
docker-compose down

# Parar o Docker Compose e remover as imagens e os volumes
docker-compose down -v --rmi all

# Pausar o Docker Compose
docker-compose pause

# Retomar o Docker Compose
docker-compose unpause
```

## Criando e conectando containers com o Docker Compose

```yaml
version: "3.8"
services:
  giropops-senhas:
    image: linuxtips/giropops-senhas:1.0
    ports:
      - "5000:5000"
    networks:
      - giropops
    environment:
        REDIS_HOST: redis
  redis:
    image: redis
    ports:
      - "6379:6379"
    networks:
      - giropops

networks:
    giropops:
        driver: bridge
```

## Utilizando volumes com o Docker Compose

```yaml