# Guia Podman

## Laboratório Prático: Implementando e Gerenciando Contêineres com Podman

**Integrantes**: Gabriel Gomes Galikosky; Mateus Lopes Albano; Paulo José de Oliveira Rolinski; Ricardo André da Silva

Bem-vindo ao Guia Podman! Este guia foi desenvolvido para ajudar você a entender e utilizar o Podman, uma alternativa moderna e segura para gerenciamento de containers.

## O que é o Podman?

Podman (Pod Manager) é uma ferramenta para desenvolver, gerenciar e executar containers em Linux. É uma alternativa ao Docker que não requer um daemon em execução e opera com privilégios de root reduzidos.

Neste laboratório, vamos explorar:

- Instalação do Podman em diferentes sistemas
- Execução de contêineres básicos e avançados
- Criação e gerenciamento de pods
- Integração com systemd para contêineres persistentes
- Comparações práticas com Docker
- Recursos exclusivos como contêineres rootless

## Por que usar Podman?

- **Daemonless**: Não requer um daemon em execução
- **Segurança**: Executa containers com privilégios reduzidos
- **Compatibilidade**: Compatível com Docker CLI e containers OCI
- **Pods**: Suporte nativo a pods, similar ao Kubernetes

## Como usar este guia

Este guia está organizado em seções progressivas, desde conceitos básicos até tópicos avançados. Recomendamos seguir a ordem:

1. Instalação
2. Fundamentos
3. Redes
4. Volumes
5. Tópicos Avançados

## Pré-requisitos

### Sistema operacional

- Linux (recomendado: Fedora, RHEL, CentOS, Ubuntu 20.04+, Debian 10+)
- Para usuários Windows: WSL2 com distribuição Linux compatível
- Para usuários macOS: Podman Machine (virtualização)

### Dependências

- Kernel Linux 3.8+ (recomendado 4.11+ para funcionalidades completas)
- Suporte a namespaces de usuário ativado no kernel
- Suporte a cgroups v2 para melhores recursos de gerenciamento
- slirp4netns (para redes rootless)
- fuse-overlayfs (para storage rootless)

### Hardware recomendado

- CPU: 2+ cores
- RAM: 2GB+ de memória disponível
- Disco: 10GB+ de espaço livre
- Conexão de internet para downloads e atualizações
- Acesso root ou sudo para instalação e configuração
- Editor de texto - nano, por exemplo - para edição de arquivos
