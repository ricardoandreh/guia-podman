# Introdução aos Volumes no Podman

O Podman (POD Manager) é uma ferramenta sem daemon para criar, gerenciar e executar contêineres OCI (Open Container Initiative). Um dos aspectos fundamentais ao trabalhar com contêineres é entender como gerenciar os dados persistentes, e é aí que os volumes entram.

Os volumes no Podman fornecem uma maneira de armazenar e acessar dados persistentes entre execuções de contêineres, permitindo separar o ciclo de vida dos dados do ciclo de vida dos contêineres.

## Conceitos Básicos de Volumes

### O que são Volumes?

Volumes são mecanismos para persistir dados gerados e utilizados por contêineres. Por padrão, os contêineres são efêmeros - quando um contêiner é removido, todos os dados dentro dele são perdidos. Os volumes resolvem este problema fornecendo armazenamento persistente.

### Por que Usar Volumes?

- **Persistência de dados**: Os dados permanecem mesmo após o contêiner ser removido
- **Compartilhamento de dados**: Facilita o compartilhamento entre contêineres
- **Desempenho**: Geralmente oferecem melhor desempenho do que montagens bind
- **Portabilidade**: Funcionam de forma semelhante em diferentes sistemas operacionais
- **Segurança**: Oferecem mecanismos de isolamento mais seguros

## Tipos de Armazenamento no Podman

O Podman suporta diferentes métodos para persistir dados:

### 1. Volumes Nomeados

Volumes gerenciados pelo Podman, armazenados em um local específico do sistema de arquivos do host.

```bash
podman volume create meu-volume
podman run -v meu-volume:/dados alpine sh
```

### 2. Montagens Bind (Bind Mounts)

Mapeiam diretamente um diretório ou arquivo do host para o contêiner.

```bash
podman run -v /caminho/no/host:/caminho/no/container:Z alpine sh
```

> **Nota**: A flag `:Z` é importante em sistemas com SELinux ativado, pois configura o contexto de segurança apropriado.

### 3. Volumes Temporários (tmpfs)

Armazenamento em memória, útil para dados temporários que não precisam ser persistidos.

```bash
podman run --tmpfs /tmp:rw,size=100M alpine sh
```

## Comandos Essenciais para Gerenciar Volumes

### Criar Volumes

```bash
# Criar um volume simples
podman volume create meu-volume

# Criar volume com opções específicas
podman volume create --driver local --opt type=btrfs --opt device=/dev/sda2 meu-volume-btrfs
```

### Listar Volumes

```bash
# Listar todos os volumes
podman volume ls

# Filtrar volumes
podman volume ls --filter name=meu
```

### Inspecionar Volumes

```bash
podman volume inspect meu-volume
```

### Remover Volumes

```bash
# Remover um volume específico
podman volume rm meu-volume

# Remover volumes não utilizados
podman volume prune
```

### Usar Volumes com Contêineres

```bash
# Executar contêiner com volume
podman run -d --name mysql -v mysql-data:/var/lib/mysql mysql:8.0

# Criar e montar volume em um único comando
podman run -d --name postgres -v postgres-data:/var/lib/postgresql/data postgres:14
```

## Casos de Uso Comuns

### Bancos de Dados

```bash
# MongoDB com persistência de dados
podman run -d --name mongodb \
  -v mongodb-data:/data/db \
  -p 27017:27017 \
  mongo:latest
```

### Servidores Web

```bash
# Nginx servindo conteúdo estático
podman run -d --name webserver \
  -v ./website:/usr/share/nginx/html:ro,Z \
  -p 8080:80 \
  nginx:alpine
```

### Compartilhamento entre Contêineres

```bash
# Criar um volume compartilhado
podman volume create dados-compartilhados

# Contêiner 1 grava dados
podman run -d --name produtor \
  -v dados-compartilhados:/shared \
  alpine sh -c "echo 'Dados importantes' > /shared/arquivo.txt && tail -f /dev/null"

# Contêiner 2 lê dados
podman run -it --rm \
  -v dados-compartilhados:/leitura:ro \
  alpine cat /leitura/arquivo.txt
```

## Melhores Práticas

### Nomeação de Volumes

Adote um padrão de nomenclatura claro e consistente para seus volumes:

```bash
# Padrão sugerido: app-tipo-ambiente
podman volume create webapp-mysql-prod
podman volume create webapp-mysql-dev
```

### Backup e Restauração

Para fazer backup de dados em volumes:

```bash
# Backup
podman run --rm -v meu-volume:/origem -v $(pwd):/destino:Z alpine tar -czvf /destino/backup.tar.gz -C /origem .

# Restauração
podman run --rm -v meu-volume:/destino -v $(pwd):/origem:Z alpine sh -c "rm -rf /destino/* && tar -xzvf /origem/backup.tar.gz -C /destino"
```

### Segurança e Permissões

Configurando permissões adequadas:

```bash
# Definir propriedade (como UID 1000)
podman run --rm -v meu-volume:/dados alpine chown -R 1000:1000 /dados

# Definir permissões apenas leitura para o contêiner
podman run -v meu-volume:/dados:ro alpine sh
```

## Solução de Problemas

### Problemas de Permissão

Se encontrar erros de permissão, verifique:

1. **Contexto SELinux**: Use `:Z` ou `:z` em sistemas com SELinux
   ```bash
   podman run -v /host:/container:Z alpine sh
   ```

2. **UID/GID**: Compatibilize os IDs de usuário
   ```bash
   podman run --user $(id -u):$(id -g) -v /host:/container alpine sh
   ```

### Volume não aparece na listagem

Verifique se está usando o mesmo contexto do Podman:

```bash
# Para usuário root
sudo podman volume ls

# Para usuário comum
podman volume ls
```

### Contêineres não conseguem acessar os volumes

Verifique o contexto e namespace corretos:

```bash
# Verificar onde o volume está armazenado
podman volume inspect meu-volume

# Verificar se o namespace do usuário está correto
podman info | grep "User namespace"
```

## Diferenças entre Podman e Docker para Volumes

O Podman foi projetado para ser compatível com o Docker, mas existem algumas diferenças importantes:

1. **Arquitetura sem daemon**: O Podman não depende de um processo daemon central
2. **Localização dos volumes**: 
   - Docker: `/var/lib/docker/volumes/`
   - Podman (root): `/var/lib/containers/storage/volumes/`
   - Podman (rootless): `~/.local/share/containers/storage/volumes/`
3. **Suporte a rootless**: Podman tem melhor suporte para execução sem privilégios

Para migrar volumes do Docker para o Podman:

```bash
# Exportar volume do Docker
docker run --rm -v docker-volume:/from -v $(pwd):/to alpine cp -r /from/. /to/data

# Importar volume para o Podman
podman volume create podman-volume
podman run --rm -v $(pwd)/data:/from -v podman-volume:/to alpine cp -r /from/. /to/
```