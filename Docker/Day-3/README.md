# Aprofundando em Containers e Docker - Distroless, Trivy e Docker Scout

### O que é o Distroless?

O Distroless é uma imagem Docker que contém apenas o aplicativo e suas dependências, sem qualquer outro software, como shells, utilitários de pacote ou bibliotecas de linguagem. Eles são executados como usuários não privilegiados e não possuem capacidade de execução de código.

### Como usar o Distroless?

Para usar o Distroless, você precisa criar uma imagem Docker multistage. A primeira etapa é criar uma imagem com todas as dependências necessárias para compilar seu aplicativo. A segunda etapa é copiar o binário do aplicativo da primeira etapa e criar uma imagem Distroless com ele.



#### Exemplo de uma imagem de app em go utilizando as imagens da ChainGuard:

Crie os arquivos [main.go](/Day-3/Go-Distroless/main.go), [go.mod](/Day-3/Go-Distroless/go.mod) e [Dockerfile](/Day-3/Go-Distroless/Dockerfile). Após isso, execute os comandos abaixo:

```bash
docker build -t distroless-go .
```

```bash
docker run --rm -it -p 8080:8080 distroless-go
```

#### Desafio proposto em aula

Rodar o app de senhas em uma imagem Distroless.

1. Criar o [Dockerfile](/Day-3/Dockerfile)

2. Subir o Redis

    ```bash	
    sudo docker run -d -p 6379:6379 --name redis cgr.dev/chainguard/redis:latest
    ```

3. Criar a imagem

    ```bash
    sudo docker build -t giropops-senhas:1.0 .
    ```

4. Rodar a imagem

    ```bash
    sudo docker container run -d -p 5000:5000 --env REDIS_HOST=172.17.0.2 --name giropops-senhas giropops-senhas:1.0
    ```

5. Testar a aplicação

    ```bash
    curl http://localhost:5000
    ```

### O que é o Trivy?

O Trivy é um scanner de vulnerabilidade de código aberto para contêineres e sistemas operacionais, detectando pacotes e bibliotecas desatualizados, bem como códigos maliciosos em arquivos de imagem (por exemplo, arquivos binários ELF). O Trivy é executado em contêineres e sistemas operacionais comuns, como Alpine, Debian, RHEL, CentOS, Ubuntu e Amazon Linux AMI.

### Como usar o Trivy?

Para usar o Trivy, você precisa [instalar o binário](https://aquasecurity.github.io/trivy/v0.44/getting-started/installation/)  em seu sistema operacional. Você pode fazer isso usando o gerenciador de pacotes do seu sistema operacional ou baixando o binário do site do Trivy. Após a instalação, você pode executar o Trivy em uma imagem Docker usando o comando abaixo:

```bash
trivy image <image-name>
```

### O que é o Docker Scout?

O Docker Scout é uma ferramenta de linha de comando que permite que você verifique rapidamente se há vulnerabilidades de segurança em suas imagens do Docker. Ele usa o Trivy para verificar as imagens e fornece uma saída fácil de ler.

### Como usar o Docker Scout?

Para usar o Docker Scout, você precisa instalar o binário em seu sistema operacional. Você pode fazer isso usando o gerenciador de pacotes do seu sistema operacional ou baixando o binário do site do Docker Scout. Após a instalação, você pode executar o Docker Scout em uma imagem Docker usando o comando abaixo:

```bash
curl -fsSL https://raw.githubusercontent.com/docker/scout-cli/main/install.sh -o install-scout.sh
sh install-scout.sh
```

```bash
docker scout cves <image-name>
```