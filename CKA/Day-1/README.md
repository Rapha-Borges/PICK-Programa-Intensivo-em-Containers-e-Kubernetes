# CKA

## Atalhos

`--dry-run=client`

```bash
$d
```

`-o yaml`

```bah
$o
```


## Editar um Pod

Obter yaml de um pod:

```bash
kgp <nome do pod> -o yaml > pod.yaml
```

Deletar linhas spec dentro do vim:

```bash
1000 dd
```

Substituir pod com novas informações

```bash
k replace pod --force --grace-period 0
```

Deletar pod sem aguardar o grace period:

```bash
k delete pod <nome do pod> --force --grace-period 0
```

## Modificar o comando padrão do pod:

Adicionando 

```bash
--command <comando a ser executado>
```

Ex:

```bash
k run <nome do pod> --image=nginx --command $d $o -- /bin/sh -c "sleep 30"
```
