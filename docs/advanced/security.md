# Segurança no Podman

A segurança no Podman se baseia em vários componentes do kernel Linux e em sua arquitetura sem daemon, que reduz a superfície de ataque e proporciona maior isolamento entre os contêineres e o sistema host.

## Arquitetura de Segurança

### Design Sem Daemon

Ao contrário do Docker, o Podman não depende de um daemon centralizado em execução com privilégios de root. Isso significa:

- Não há um único ponto de falha que possa ser explorado para comprometer todo o sistema
- Cada usuário executa seus próprios contêineres isolados
- Redução significativa da superfície de ataque
- Sem elevação de privilégios através de soquetes do sistema

### Modelo de Permissões

O Podman utiliza um modelo de permissões baseado no usuário:

```bash
# Execução como usuário normal (rootless)
podman run -d nginx

# Execução como root (quando necessário)
sudo podman run -d --privileged nginx
```

Essa separação de privilégios fortalece a segurança, permitindo operações regulares sem acesso root.

## Execução Rootless

### O que é Execução Rootless?

A execução rootless permite que contêineres sejam executados por usuários sem privilégios, oferecendo várias vantagens de segurança:

- Mitigação de vulnerabilidades de escalação de privilégios
- Isolamento entre diferentes usuários do sistema
- Menor impacto em caso de comprometimento de um contêiner

### Configurando Podman Rootless

Para habilitar e utilizar o modo rootless:

```bash
# Verificar configuração do sistema
podman system info

# Configurar mapeamento de usuário (se necessário)
sudo usermod --add-subuids 100000-165535 $(whoami)
sudo usermod --add-subgids 100000-165535 $(whoami)

# Reiniciar o serviço de usuário (ou fazer logout/login)
systemctl --user restart podman.socket
```

### Limitações do Modo Rootless

É importante entender algumas limitações:

- Portas abaixo de 1024 não podem ser usadas diretamente (requer configuração adicional)
- Alguns recursos de rede podem exigir configurações específicas
- Algumas operações específicas ainda podem exigir permissões de root

## Namespaces e Isolamento

O Podman utiliza namespaces do kernel Linux para fornecer isolamento entre contêineres:

### Namespaces Principais

- **PID Namespace**: Isola os processos em execução no contêiner
- **Network Namespace**: Isola a pilha de rede (interfaces, rotas, firewall)
- **Mount Namespace**: Isola os pontos de montagem do sistema de arquivos
- **UTS Namespace**: Isola hostname e domínio
- **IPC Namespace**: Isola comunicação entre processos
- **User Namespace**: Mapeia UIDs/GIDs entre contêiner e host

### Configurando Isolamento Adicional

```bash
# Isolar o namespace PID do host
podman run --pid=host nginx

# Utilizar namespace de usuário separado
podman run --userns=keep-id nginx

# Executar contêiner com todos os namespaces isolados
podman run --pid=private --ipc=private --net=private nginx
```

## Controles de Recursos

### Limites de CPU e Memória

O Podman permite restringir recursos disponíveis para contêineres, limitando o impacto potencial de ataques de negação de serviço:

```bash
# Limitar CPU
podman run --cpus=1.5 --cpu-shares=512 nginx

# Limitar memória
podman run --memory=512m --memory-swap=1g nginx

# Limitar E/S de disco
podman run --device-read-bps=/dev/sda:10mb --device-write-bps=/dev/sda:10mb nginx
```

### Configurando cgroups

O Podman utiliza cgroups (control groups) para implementar controles de recursos:

```bash
# Especificar o controlador cgroup v2
podman run --cgroup-manager=systemd nginx

# Definir limite de pids (processos)
podman run --pids-limit=100 nginx
```

## SELinux e AppArmor

### SELinux com Podman

O Podman integra-se bem com o SELinux, proporcionando controle de acesso mandatório:

```bash
# Executar com contexto de SELinux padrão
podman run -v /host/path:/container/path:Z nginx

# Compartilhar volume entre contêineres (shared label)
podman run -v /host/path:/container/path:z nginx

# Verificar contexto SELinux em volumes
podman inspect --format '{{.Mounts}}' container_id
```

O modificador `:Z` aplica o contexto SELinux apropriado ao volume, enquanto `:z` aplica um rótulo compartilhado.

### AppArmor com Podman

Em sistemas com AppArmor, o Podman pode utilizar perfis para restringir o acesso:

```bash
# Executar com perfil AppArmor específico
podman run --security-opt apparmor=custom-profile nginx

# Desabilitar AppArmor para um contêiner (não recomendado)
podman run --security-opt apparmor=unconfined nginx
```

## Capabilities do Linux

### Gerenciando Capabilities

As capabilities do Linux permitem um controle fino sobre privilégios, eliminando a necessidade do modo privilegiado completo:

```bash
# Adicionar capabilities específicas
podman run --cap-add=NET_ADMIN,SYS_TIME nginx

# Remover capabilities específicas
podman run --cap-drop=ALL --cap-add=CHOWN nginx

# Listar capabilities padrão
podman run --rm docker.io/alpine grep Cap /proc/self/status
```

### Capabilities Importantes

- `NET_ADMIN`: Configuração de rede
- `SYS_ADMIN`: Operações administrativas do sistema (evitar quando possível)
- `SYS_PTRACE`: Depuração e rastreamento de processos
- `MKNOD`: Criar dispositivos especiais
- `AUDIT_WRITE`: Escrever registros de auditoria

### Modo Privilegiado

O modo privilegiado deve ser evitado sempre que possível, pois concede acesso quase completo ao sistema host:

```bash
# Modo privilegiado (use com extrema cautela)
podman run --privileged nginx

# Alternativa mais segura: adicionar apenas capabilities necessárias
podman run --cap-add=NET_ADMIN --device=/dev/net/tun nginx
```

## Imagens e Vulnerabilidades

### Práticas Seguras para Imagens

A segurança começa com imagens confiáveis e bem mantidas:

```bash
# Verificar a origem da imagem
podman pull docker.io/library/nginx

# Inspecionar a imagem antes de usá-la
podman inspect nginx:latest

# Usar imagens com tags específicas, evitando "latest"
podman pull nginx:1.21.6-alpine
```

### Escaneamento de Vulnerabilidades

O ecossistema do Podman oferece ferramentas para escanear vulnerabilidades:

```bash
# Usando Trivy para escanear imagens
trivy image nginx:latest

# Verificando imagem com Clair (via skopeo)
skopeo inspect --format "{{.Name}}" docker://nginx:latest | xargs -I {} clair-scanner {}
```

### Construção Segura de Imagens

Construa imagens com foco em segurança usando o Buildah (integrado ao Podman):

```bash
# Construir com usuário não-root
podman build --build-arg USER=appuser -t minha-app:secure .

# Conteúdo do Dockerfile
FROM alpine:latest
RUN adduser -D appuser
USER appuser
COPY --chown=appuser:appuser app /app
WORKDIR /app
ENTRYPOINT ["./app"]
```

## Secrets Management

### Gerenciando Segredos

O Podman oferece várias formas de gerenciar segredos em contêineres:

```bash
# Usando variáveis de ambiente (menos seguro)
podman run -e "API_KEY=segredo" nginx

# Usando arquivos montados (mais seguro)
podman run -v /path/to/secret:/run/secrets:ro,Z nginx

# Usando secret mounts (mais recente)
podman secret create db_password password.txt
podman run --secret db_password nginx
```

### Melhores Práticas para Segredos

- Nunca armazene segredos nas imagens ou Dockerfiles
- Use montagens de somente leitura para arquivos de configuração sensíveis
- Evite passar segredos via variáveis de ambiente quando possível
- Considere ferramentas externas como HashiCorp Vault ou AWS Secrets Manager

## Redes e Segurança

### Isolamento de Rede

O Podman permite isolar redes entre contêineres:

```bash
# Criar rede isolada
podman network create app-network

# Executar contêiner na rede isolada
podman run --network=app-network --name=db postgres

# Conectar contêiner a múltiplas redes
podman network connect another-network db
```

### Regras de Firewall

Controle cuidadosamente a exposição de portas:

```bash
# Expor porta apenas em localhost
podman run -p 127.0.0.1:8080:80 nginx

# Evitar expor portas desnecessárias
podman run --network=app-network --expose 5432 postgres
```

### DNS e Resolução de Nomes

Configure DNS de forma segura:

```bash
# Definir servidores DNS específicos
podman run --dns=1.1.1.1 nginx

# Adicionar entrada de hosts personalizada
podman run --add-host=myservice:10.10.10.10 nginx
```

## Melhores Práticas

### Princípio de Privilégio Mínimo

- Execute sempre como usuário não-root dentro do contêiner
- Evite o modo privilegiado
- Adicione apenas as capabilities estritamente necessárias
- Use modo somente leitura para sistemas de arquivos quando possível

```bash
# Exemplo de contêiner com privilégio mínimo
podman run --read-only \
  --cap-drop=ALL \
  --cap-add=NET_BIND_SERVICE \
  --user 1000:1000 \
  nginx
```

### Hardening de Contêineres

```bash
# Sistema de arquivos somente leitura
podman run --read-only nginx

# Desabilitar escalações de privilégio
podman run --security-opt=no-new-privileges nginx

# Usar seccomp para filtrar syscalls
podman run --security-opt seccomp=/path/to/profile.json nginx
```

### Monitoramento e Auditoria

- Configure logging adequado para todos os contêineres
- Implemente ferramentas de monitoramento para detectar comportamentos anômalos
- Estabeleça políticas de auditoria para revisar logs regularmente

```bash
# Configurar logging personalizado
podman run --log-driver=journald --log-opt=tag=webapp nginx
```

## Segurança em Ambientes de Produção

### Configurações Recomendadas

Para ambientes de produção, implemente estas configurações:

```bash
# Contêiner com configurações de segurança para produção
podman run \
  --cap-drop=ALL \
  --cap-add=NET_BIND_SERVICE \
  --security-opt=no-new-privileges \
  --security-opt seccomp=/path/to/seccomp.json \
  --read-only \
  --tmpfs /tmp:rw,noexec,nosuid,size=50m \
  --user nobody \
  nginx
```

### Atualizações e Patches

- Mantenha imagens atualizadas com patches de segurança
- Implemente uma estratégia de atualização regular
- Configure escaneamento automático de vulnerabilidades

```bash
# Atualizar imagens
podman pull nginx:1.21-alpine
podman image prune -a --filter "until=24h"
```

## Comparação com Docker

### Vantagens de Segurança do Podman

O Podman oferece vantagens distintas em comparação ao Docker:

1. **Arquitetura sem daemon**: Não requer processo privilegiado em execução constante
2. **Suporte nativo a rootless**: Projetado desde o início para operação sem privilégios
3. **Integração com systemd**: Melhor gerenciamento de ciclo de vida de contêineres
4. **Fork/Exec em vez de cliente/servidor**: Modelo de processo mais seguro

### Migração Segura do Docker para Podman

```bash
# Converter docker-compose para podman
podman-compose -f docker-compose.yml up

# Migrar contêiner Docker para Podman
docker inspect container > container.json
podman import container.json
```

## Ferramentas de Segurança

### Ferramentas de Avaliação

- **OpenSCAP**: Framework de conformidade e segurança
- **Clair**: Análise estática de vulnerabilidades para contêineres
- **Trivy**: Scanner de vulnerabilidades para imagens de contêineres
- **Falco**: Detecção de comportamento anômalo em runtime

### Integração com CI/CD

Integre escaneamento de segurança em seu pipeline de CI/CD:

```bash
# Exemplo de passo em um pipeline CI
scan_image() {
  podman pull ${IMAGE}
  trivy image --severity HIGH,CRITICAL ${IMAGE}
  if [ $? -ne 0 ]; then
    echo "Falha: vulnerabilidades de alta severidade detectadas"
    exit 1
  fi
}
```

## Rootless Containers

```bash
# Executar como usuário não-root
podman run --user 1000:1000 nginx

# Verificar contexto de segurança
podman run --security-opt label=level:s0:c100,c200 fedora
```

## SELinux e Apparmor

```bash
# Executar com perfil SELinux específico
podman run --security-opt label=type:container_t nginx

# Usar perfil Apparmor
podman run --security-opt apparmor=custom-profile nginx
```

## Capabilities

```bash
# Remover capabilities específicas
podman run --cap-drop NET_ADMIN nginx

# Adicionar capabilities
podman run --cap-add SYS_PTRACE nginx
```

## Boas Práticas

1. Use rootless containers sempre que possível
2. Limite capabilities ao mínimo necessário
3. Implemente políticas de SELinux/AppArmor
4. Atualize imagens regularmente
5. Escaneie vulnerabilidades:

```bash
podman image scan nginx:latest
```
