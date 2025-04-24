# Gerenciando redes de Containers

Este capítulo fornece informações sobre como comunicar-se entre contêineres.

## 1. Listando as redes de um container

No Podman, existem dois comportamentos de rede:

- **Rootless networking**: a rede é configurada automaticamente, o contêiner não possui um endereço IP.
- **Rootful networking**: o contêiner possui um endereço IP.

**Pré-requisitos**: O meta-pacote `container-tools` está instalado.

**Procedimento**: Liste todas as redes como usuário root:

```bash
# podman network ls
NETWORK ID    NAME        VERSION     PLUGINS
2f259bab93aa  podman      0.4.0       bridge,portmap,firewall,tuning
```

Por padrão, o Podman fornece uma rede em ponte. A lista de redes para um usuário sem privilégios (`rootless`) é a mesma que para um usuário com privilégios (`rootful`).

> Additional resources - `podman-network-ls` man page on your system

## 2. Inspecionando uma rede

Exibe o intervalo de IP, plugins habilitados, tipo de rede, etc., para uma rede especificada listada pelo comando `podman network ls`.

**Pré-requisitos**: O meta-pacote `container-tools` está instalado.

**Procedimento**: Inspecione a rede padrão do Podman:

```bash
$ podman network inspect podman
[
    {
        "cniVersion": "0.4.0",
        "name": "podman",
        "plugins": [
            {
                "bridge": "cni-podman0",
                "hairpinMode": true,
                "ipMasq": true,
                "ipam": {
                    "ranges": [
                        [
                            {
                                "gateway": "10.88.0.1",
                                "subnet": "10.88.0.0/16"
                            }
                        ]
                    ],
                    "routes": [
                        {
                            "dst": "0.0.0.0/0"
                        }
                    ],
                    "type": "host-local"
                },
                "isGateway": true,
                "type": "bridge"
            },
            {
                "capabilities": {
                    "portMappings": true
                },
                "type": "portmap"
            },
            {
                "type": "firewall"
            },
            {
                "type": "tuning"
            }
        ]
    }
]
```

Você pode ver o intervalo de IP, plugins habilitados, tipo de rede e outras configurações de rede.

> Additional resources - `podman-network-inspect` man page on your system

## 3. Criando uma rede

Use o comando `podman network create` para criar uma nova rede.

**Nota**: Por padrão, o Podman cria uma rede externa. Você pode criar uma rede interna usando o comando `podman network create --internal`. Contêineres em uma rede interna podem se comunicar com outros contêineres no host, mas não podem se conectar à rede fora do host nem ser alcançados a partir dela.

**Pré-requisitos**: O meta-pacote `container-tools` está instalado.

**Procedimento**: Crie a rede externa chamada `mynet`:

```bash
# podman network create mynet
/etc/cni/net.d/mynet.conflist
```

**Verificação**: Liste todas as redes:

```bash
# podman network ls
NETWORK ID    NAME        VERSION     PLUGINS
2f259bab93aa  podman      0.4.0       bridge,portmap,firewall,tuning
11c844f95e28  mynet       0.4.0       bridge,portmap,firewall,tuning,dnsname
```

Você pode ver a rede `mynet` criada e a rede padrão `podman`.

**Nota**: A partir do Podman 4.0, o plugin DNS é habilitado por padrão se você criar uma nova rede externa usando o comando `podman network create`.

> Additional resources - `podman-network-create` man page on your system

## 4. Conectando um container à rede

Use o comando `podman network connect` para conectar o contêiner à rede.

**Pré-requisitos**:

- O meta-pacote `container-tools` está instalado.
- Uma rede foi criada usando o comando `podman network create`.
- Um contêiner foi criado.

**Procedimento**: Conecte um contêiner chamado `mycontainer` a uma rede chamada `mynet`:

```bash
# podman network connect mynet mycontainer
map[podman:0xc00042ab40 mynet:0xc00042ac60]
```

**Verificação**: Verifique se o `mycontainer` está conectado à rede `mynet`:

```bash
# podman inspect --format='{{.NetworkSettings.Networks}}' mycontainer
```

Você pode ver que `mycontainer` está conectado às redes `mynet` e `podman`.

> Additional resources - `podman-network-connect` man page on your system

## 5. Desconectando um container da rede

Use o comando `podman network disconnect` para desconectar o contêiner da rede.

**Pré-requisitos**:

- O meta-pacote `container-tools` está instalado.
- Uma rede foi criada usando o comando `podman network create`.
- Um contêiner está conectado a uma rede.

**Procedimento**: Desconecte o contêiner chamado `mycontainer` da rede chamada `mynet`:

```bash
# podman network disconnect mynet mycontainer
```

**Verificação**: Verifique se o `mycontainer` está desconectado da rede `mynet`:

```bash
# podman inspect --format='{{.NetworkSettings.Networks}}' mycontainer
map[podman:0xc000537440]
```

Você pode ver que `mycontainer` está desconectado da rede `mynet` e conectado apenas à rede padrão `podman`.

> Additional resources - `podman-network-disconnect` man page on your system

## 6. Removendo uma rede

Use o comando `podman network rm` para remover uma rede especificada.

**Pré-requisitos**: O meta-pacote `container-tools` está instalado.

**Procedimento**
 1. Liste todas as redes não utilizadas:


```bash
# podman network prune
NETWORK ID    NAME        VERSION     PLUGINS
2f259bab93aa  podman      0.4.0       bridge,portmap,firewall,tuning
11c844f95e28  mynet       0.4.0       bridge,portmap,firewall,tuning,dnsname
```

2. Remove the `mynet` network:
```bash
# podman network rm mynet
mynet
```

**Observação**: Se a rede removida tiver contêineres associados a ela, você precisará usar o `podman network rm -f` comando para excluir contêineres e pods.

**Verificação**: Verifique se `mynet` rede foi removida:

```bash
# podman rede ls
ID DA REDE NOME VERSÃO PLUGINS
2f259bab93aa podman 0.4.0 ponte, mapa de portas, firewall, ajuste
```

>Recursos adicionais - `podman-network-rm` página de manual no seu sistema

## 7. Removendo todas as redes não utilizadas

Use o comando `podman network prune` para remover todas as redes não utilizadas. Uma rede não utilizada é uma rede sem contêineres conectados a ela. O `podman network prune` comando não remove a `podman` rede padrão.

**Pré-requisitos**: O meta-pacote `container-tools` está instalado.

**Procedimento**\
Remova todas as redes não utilizadas:

```bash
# podman rede prune
AVISO! Isso removerá todas as redes não utilizadas por pelo menos um contêiner.
Tem certeza de que deseja continuar? [s/N] s
```

**Verificação**\
Verifique se todas as redes foram removidas:
```bash
# podman rede ls
ID DA REDE NOME VERSÃO PLUGINS
2f259bab93aa podman 0.4.0 ponte, mapa de portas, firewall, ajuste
mynet
```

>Recursos adicionais - `podman-network-prune` página de manual no seu sistema