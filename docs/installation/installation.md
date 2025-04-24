# Instalando o Podman

## Ubuntu/Debian

```bash
# Ubuntu 20.04+
sudo apt-get update
sudo apt-get -y install podman
```

## Fedora

```bash
sudo dnf -y install podman
```

## RHEL/CentOS

```bash
sudo yum -y install podman
```

## MacOS

```bash
brew install podman
```

## Verificando a Instalação

Após a instalação, verifique se o Podman está funcionando corretamente:

```bash
podman --version
podman info
```