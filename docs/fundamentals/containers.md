# Gerenciamento de Containers

## Ciclo de Vida

```bash
# Criar e iniciar
podman run -d --name web nginx

# Parar container
podman stop web

# Iniciar container existente
podman start web

# Remover container
podman rm web
```

## Operações Básicas

```bash
# Executar comando em container
podman exec web ls /app

# Copiar arquivos
podman cp ./local/file.txt web:/app/

# Ver logs
podman logs web

# Inspecionar container
podman inspect web
```

## Recursos e Limites

```bash
# Limitar CPU e memória
podman run -d \
    --cpus=1.5 \
    --memory=512m \
    --name app \
    nginx

# Monitorar uso
podman stats app
```

## Melhores Práticas

1. Use nomes descritivos
2. Defina limites de recursos
3. Monitore o desempenho
4. Mantenha backups
5. Use healthchecks:

```bash
podman run --health-cmd="curl -f http://localhost" nginx
```
