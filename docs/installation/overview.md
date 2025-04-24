# Visão Geral da Instalação

O Podman está disponível para várias distribuições Linux e MacOS. A instalação varia dependendo do seu sistema operacional.

## Sistemas Suportados

- Fedora
- Red Hat Enterprise Linux
- CentOS
- Ubuntu
- Debian
- MacOS (via máquina virtual)

## Requisitos do Sistema

- Linux kernel 3.8 ou superior
- systemd (recomendado)
- cgroups v2 (para melhor funcionalidade)
- Armazenamento suficiente para imagens e containers

## Instalação

=== "Fedora/Red Hat/CentOS"

    ```bash
    # Instalar o Podman
    sudo dnf install podman -y

    # Verificar instalação
    podman --version
    ```

=== "Ubuntu/Debian"

    ```bash
    # Atualizar índices de pacotes
    sudo apt update

    # Instalar Podman
    sudo apt install podman -y

    # Verificar instalação
    podman --version
    ```

=== "Arch Linux"

    ```bash
    sudo pacman -S podman

    # Verificar instalação
    podman --version
    ```

=== "macOS (via Podman Machine)"

    ```bash
    # Usando Homebrew
    brew install podman

    # Inicializar máquina virtual do Podman
    podman machine init
    podman machine start

    # Verificar status
    podman machine list
    ```

=== "Windows (via WSL2)"

    1. Instalar WSL2 com distribuição Linux compatível
    2. Na distribuição Linux, instalar Podman conforme instruções acima
    3. Configurar namespaces de usuário se necessário:

    ```bash
    sudo echo 'kernel.unprivileged_userns_clone=1' >> /etc/sysctl.d/00-local-userns.conf
    sudo systemctl restart systemd-sysctl
    ```
