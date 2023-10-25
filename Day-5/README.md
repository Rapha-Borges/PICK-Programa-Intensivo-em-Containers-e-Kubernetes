# Redes e Limites de Recursos

## Network

### O que é uma rede no Docker?

Uma rede no Docker é um conjunto de containers que podem se comunicar entre si. O Docker possui três tipos de redes: bridge, host e none.

### Bridge

A bridge é uma rede que permite que os containers se comuniquem entre si e com o host. É a rede padrão do Docker. É possível criar uma bridge para cada aplicação ou utilizar a bridge padrão.

```bash
# Criar uma rede bridge
docker network create <nome>
```

### Host

A host é uma rede que permite que os containers se comuniquem entre si e com o host. É possível criar uma host para cada aplicação ou utilizar a host padrão.

```bash
# Criar uma rede host
docker network create --driver host <nome>
```

### None

A none é uma rede que não permite que os containers se comuniquem entre si e com o host. É possível criar uma none para cada aplicação ou utilizar a none padrão.

```bash
# Criar uma rede none
docker network create --driver none <nome>
```

### Overlay

A overlay é uma rede que permite que os containers se comuniquem entre si e com o host. É possível criar uma overlay para cada aplicação ou utilizar a overlay padrão.


```bash
# Criar uma rede overlay
docker network create --driver overlay <nome>
```




## Limites de Recursos

### O que são limites de recursos?

Limites de recursos são limites que podem ser impostos aos containers. É possível limitar a quantidade de memória, a quantidade de CPU, a quantidade de I/O e a quantidade de processos.

```bash
# Limitar a quantidade de memória
docker run --memory <quantidade>M <imagem>

# Limitar a quantidade de CPU
docker run --cpus <quantidade> <imagem>

# Limitar a quantidade de I/O
docker run --device-write-bps <dispositivo>=<quantidade> <imagem>

# Limitar a quantidade de processos
docker run --pids-limit <quantidade> <imagem>
```

