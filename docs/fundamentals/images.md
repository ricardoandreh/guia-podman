# Trabalhando com Imagens

## Gerenciamento de Imagens

### Buscar Imagens
```bash
podman search nginx
podman pull nginx:latest
```

### Listar e Inspecionar
```bash
podman images
podman image inspect nginx
```

### Construir Imagens
```bash
podman build -t minha-app:1.0 .
```

## Dockerfile com Podman

O Podman é compatível com Dockerfiles. Exemplo:

```Dockerfile
FROM ubuntu:20.04
WORKDIR /app
COPY . .
RUN apt-get update && apt-get install -y python3
CMD ["python3", "app.py"]
```

## Boas Práticas

1. Use multi-stage builds para imagens menores
2. Minimize o número de camadas
3. Use tags específicas
4. Remova imagens não utilizadas:

```bash
podman image prune
```