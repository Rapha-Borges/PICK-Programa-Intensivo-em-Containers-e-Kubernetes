# Volumes - Bind, tmpfs e volume

### O que é um volume?

Um volume é um diretório ou arquivo que é montado dentro do container. O volume pode ser montado em um diretório ou arquivo do host ou em um diretório dentro do container. O volume pode ser usado para armazenar dados que precisam ser persistidos, como banco de dados, arquivos de configuração, etc.

## Bind

O bind é um tipo de volume que permite montar um diretório do host dentro do container. É possível montar um diretório ou um arquivo. Você pode usar o comando `--mount` ou `-v` para montar o volume:

```bash
docker run -d --name nginx -p 8080:80 --mount type=bind,source=/usr/share/nginx/html,target=/usr/share/nginx/html nginx
```

```bash
docker run -d --name nginx -p 8080:80 -v /usr/share/nginx/html:/usr/share/nginx/html nginx
```

## tmpfs

O tmpfs é um tipo de volume que permite montar um diretório na memória RAM do host. É possível montar um diretório ou um arquivo.

```bash
docker run -d --name nginx -p 8080:80 --tmpfs /usr/share/nginx/html nginx
```

## Volume

O volume é um tipo de volume que permite montar um diretório dentro do container. É possível montar um diretório ou um arquivo.

```bash
docker run -d --name nginx -p 8080:80 -v /usr/share/nginx/html nginx
docker run -d --name nginx -p 8080:80 -v <volume>:/usr/share/nginx/html nginx
```

## Comandos

```bash
# Criar um volume
docker volume create <nome>

# Listar volumes
docker volume ls

# Inspect volume
docker volume inspect <nome>

# Remover volume
docker volume rm <nome>

# Remover todos os volumes não utilizados
docker volume prune

# Localizar o local onde o volume está montado
docker inspect <container> | grep Source

# Localizar o local onde o volume está montado 
docker volume inspect --format '{{ .Mountpoint }}' <container>
```