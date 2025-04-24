# Mapeamento de Portas

## Exposição de Portas

```bash
# Mapear porta 80 do container para 8080 do host
podman run -p 8080:80 nginx

# Mapear múltiplas portas
podman run -p 8080:80 -p 443:443 nginx

# Mapear para interface específica
podman run -p 127.0.0.1:8080:80 nginx
```

## Publicação de Portas

No Dockerfile:
```dockerfile
EXPOSE 80
EXPOSE 443
```

## Verificação de Portas

```bash
# Listar portas mapeadas
podman port meu-container

# Verificar conexões
podman top meu-container
```

## Boas Práticas

1. Evite expor portas desnecessárias
2. Use portas não privilegiadas (> 1024)
3. Considere restrições de firewall
4. Documente todas as portas utilizadas
