# Primeiros Passos

## Execução de um Container

### Container básico

```bash
# Baixar imagem
podman pull docker.io/nginx:latest

# Executar container em background
podman run -d --name meu-nginx -p 8080:80 nginx:latest

# Verificar container em execução
podman ps

# Acessar o site no navegador ou curl
curl http://localhost:8080
```

Explicação de parâmetros:

- `-d`: Executa em modo "detached" (background)
- `--name meu-nginx`: Atribui um nome amigável ao container
- `-p 8080:80`: Mapeia a porta 8080 do host para a porta 80 do container

Saída esperada do comando `podman ps`:

```txt
CONTAINER ID  IMAGE                           COMMAND               CREATED         STATUS             PORTS                 NAMES
3f4a8d0e15c3  docker.io/library/nginx:latest  nginx -g daemon o...  10 seconds ago  Up 9 seconds ago   0.0.0.0:8080->80/tcp  meu-nginx
```

### Container interativo

```bash
# Executar um container interativo
podman run -it --rm ubuntu:latest bash

# Dentro do container, você pode executar comandos como:
apt update
echo "Estou dentro do container!"
exit
```

Explicação de parâmetros:

- `-it`: Combinação de `-i` (interativo) e `-t` (aloca TTY)
- `--rm`: Remove automaticamente o container após a saída
- `bash`: Comando a executar no container

### Operações com containers

```bash
# Parar o container
podman stop meu-nginx

# Iniciar o container novamente
podman start meu-nginx

# Executar comando em container em execução
podman exec -it meu-nginx /bin/bash

# Ver logs do container
podman logs meu-nginx

# Remover container
podman rm -f meu-nginx
```

## Recursos Avançados do Podman

### Criando e gerenciando pods

```bash
# Criar um pod com publicação de portas
podman pod create --name webapp-pod -p 8080:80

# Adicionar containers ao pod
podman run -d --pod webapp-pod --name web-front nginx:latest
podman run -d --pod webapp-pod --name api-backend httpd:latest

# Listar pods
podman pod ps

# Inspecionar um pod
podman pod inspect webapp-pod

# Parar o pod (para todos os containers)
podman pod stop webapp-pod

# Iniciar o pod (inicia todos os containers)
podman pod start webapp-pod

# Remover o pod e seus containers
podman pod rm -f webapp-pod
```

### Container rootless

```bash
# Verificar seu usuário atual
id

# Executar container sem privilégios root
podman run --rm -it --name rootless-test alpine:latest sh

# Dentro do container, verificar o usuário
id

# Tentar operações privilegiadas (devem falhar)
apk add --no-cache tcpdump
tcpdump -i eth0
```

### Integração com systemd

```bash
# Gerar arquivo de unidade systemd para o container
mkdir -p ~/.config/systemd/user/
podman run -d --name web-service -p 8080:80 nginx:latest
podman generate systemd --name web-service --files --new

# Habilitar o serviço
systemctl --user daemon-reload
systemctl --user enable container-web-service.service
systemctl --user start container-web-service.service

# Verificar status
systemctl --user status container-web-service.service

# Parar o serviço
systemctl --user stop container-web-service.service
```

### Construindo imagens com Podman

```bash
# Criar arquivo Dockerfile

mkdir podman-build && cd podman-build
```

```Dockerfile
# Dockerfile

FROM ubuntu:latest
RUN apt-get update && apt-get install -y nginx
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

```bash
# Construir imagem
podman build -t minha-imagem:latest .

# Listar imagens
podman images

# Executar container com a imagem criada
podman run -d --name meu-app -p 8080:80 minha-imagem:latest
```

## Comparação com Docker (prática)

### Comparação de comandos

| Operação           | Docker            | Podman            | Observações                |
| ------------------ | ----------------- | ----------------- | -------------------------- |
| Executar container | `docker run`     | `podman run`     | Sintaxe idêntica           |
| Listar containers  | `docker ps`      | `podman ps`      | Saída similar              |
| Listar imagens     | `docker images`  | `podman images`  | Sintaxe e saída similares  |
| Construir imagem   | `docker build`   | `podman build`   | Suporta mesmo Dockerfile   |
| Remover containers | `docker rm`      | `podman rm`      | Sintaxe idêntica           |
| Usar compose       | `docker-compose` | `podman-compose` | Requer instalação separada |

### Exemplo de alias para migração de Docker para Podman

```bash
# Adicionar ao ~/.bashrc ou ~/.zshrc
alias docker=podman

# Testar alias
source ~/.bashrc
docker ps  # Na realidade executa podman ps
```

!!!example annotate "Diferenças importantes observadas"

    1. **Container sem privilégios root**
        
        Com Docker:
        
        ```bash
        docker run -it --rm alpine id
        # uid=0(root) gid=0(root) groups=0(root),...
        ```
        
        Com Podman:
        
        ```bash
        podman unshare cat /proc/self/uid_map
        # Verificar mapeamento de UIDs
        podman run -it --rm alpine id
        # uid=0(root) gid=0(root) groups=0(root),...
        # Porém, este "root" é mapeado para seu usuário real no host
        ```
        
    2. **Comunicação direta entre containeres no mesmo pod**
        
        Com Docker:
        
        ```bash
        # Requer rede customizada ou link
        docker network create mynet
        docker run -d --name db --network mynet postgres
        docker run -d --name app --network mynet -p 8080:80 myapp
        # App se conecta usando nome do container como hostname
        ```
        
        Com Podman:
        
        ```bash
        # Usando pods nativos
        podman pod create --name myapp-pod -p 8080:80
        podman run -d --pod myapp-pod --name db postgres
        podman run -d --pod myapp-pod --name app myapp
        # App pode se conectar ao banco usando localhost
        ```
        
    3. **Limitação: Podman-compose ainda em desenvolvimento**
        
        O Docker Compose possui mais funcionalidades que o Podman Compose, que está em estágio inicial de desenvolvimento. Para aplicações complexas que dependem de Docker Compose, pode ser necessário ajustar os arquivos ou usar pods.    

## Laboratório Aplicado: Aplicação Web Multi-container

Neste exemplo prático, implementaremos uma aplicação web com frontend, backend e banco de dados usando pods do Podman.

### 1. Criar pod para a aplicação

```bash
podman pod create --name webapp \
--publish 8080:80 \
--publish 5000:5000 \
--publish 5432:5432
```

### 2. Adicionar banco de dados PostgreSQL

```bash
podman run -d --pod webapp \
--name postgres \
-e POSTGRES_PASSWORD=senhasegura \
-e POSTGRES_USER=appuser \
-e POSTGRES_DB=webapp \
-v postgres-data:/var/lib/postgresql/data \
postgres:16.3-alpine3.20
```

### 3. Adicionar API backend

```bash
# Criar Dockerfile para backend

mkdir -p backend && cd backend
```

```Dockerfile
# Dockerfile

FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

```txt
# requirements.txt

flask
psycopg2-binary
flask-cors
```

```python
# app.py

from flask import Flask, jsonify
from flask_cors import CORS
import os

app = Flask(__name__)
CORS(app)

@app.route('/api/info')
def info():
    return jsonify({"status": "ok", "message": "API running with Podman!"})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

```bash
# Construir e executar backend

podman build -t webapp-backend .
podman run -d --pod webapp --name backend webapp-backend
cd ..
```

### 4. Adicionar frontend

```bash
# Criar Dockerfile para frontend

mkdir -p frontend && cd frontend
```

```Dockerfile
# Dockerfile

FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
COPY style.css /usr/share/nginx/html/style.css
COPY script.js /usr/share/nginx/html/script.js
```

```html
<!-- index.html -->

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Podman App Demo</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <div class="container">
        <h1>Podman Multi-Container App</h1>
        <div id="status">Checking API status...</div>
        <button id="checkButton">Check API Status</button>
    </div>
    <script src="script.js"></script>
</body>
</html>
```

```css
/* style.css */

body {
    font-family: Arial, sans-serif;
    margin: 0;
    padding: 0;
    background-color: #f4f4f4;
}
.container {
    max-width: 800px;
    margin: 50px auto;
    padding: 20px;
    background-color: white;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    text-align: center;
}
button {
    padding: 10px 20px;
    background-color: #4CAF50;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    margin-top: 20px;
}
#status {
    margin: 20px 0;
    padding: 10px;
    background-color: #f8f8f8;
    border-radius: 4px;
}
```

```js
// script.js

document.addEventListener('DOMContentLoaded', function() {
    const statusDiv = document.getElementById('status');
    const checkButton = document.getElementById('checkButton');
    
    function checkAPI() {
        statusDiv.textContent = 'Connecting to API...';
        fetch('http://localhost:5000/api/info')
            .then(response => response.json())
            .then(data => {
                statusDiv.textContent = 'API Status: ' + data.message;
                statusDiv.style.backgroundColor = '#dff0d8';
            })
            .catch(error => {
                statusDiv.textContent = 'Error connecting to API';
                statusDiv.style.backgroundColor = '#f2dede';
                console.error('Error:', error);
            });
    }
    
    checkButton.addEventListener('click', checkAPI);
    
    // Initial check
    checkAPI();
});
```

```bash
# Construir e executar frontend

podman build -t webapp-frontend .
podman run -d --pod webapp --name frontend webapp-frontend
cd ..
```

### 5. Verificar e testar a aplicação

```bash
# Listar componentes em execução
podman pod ps
podman ps --pod webapp

# Verificar logs
podman logs backend
podman logs frontend

# Testar acesso à aplicação
echo "Acesse em seu navegador: http://localhost:8080"
curl http://localhost:5000/api/info
```

## Conclusão

### O que aprendemos com o laboratório

Através deste laboratório prático, exploramos o Podman como uma alternativa viável e segura ao Docker. Aprendemos que:

1. O Podman oferece uma experiência de usuário semelhante ao Docker, com comandos familiares e sintaxe compatível, facilitando a migração.

2. A arquitetura sem daemon do Podman elimina o ponto único de falha presente no Docker e permite operações sem privilégios de root, aumentando significativamente a segurança.

3. O conceito de pods nativos no Podman facilita a organização de aplicações multi-container e a transição para ambientes Kubernetes.

4. A integração com systemd permite gerenciar contêineres como serviços do sistema, aproveitando recursos como reinicialização automática e dependências.

5. Em termos de funcionalidade para desenvolvimento local e ambientes de produção, o Podman oferece todas as capacidades essenciais do Docker com modelo de segurança aprimorado.

!!!example annotate "Sugestões de uso futuro"

    #### Ambientes corporativos com requisitos de segurança elevados

    O Podman é ideal para empresas que precisam aderir a políticas de segurança estritas, onde a execução de serviços com privilégios elevados (como o daemon do Docker) é restrita ou indesejada.

    #### Desenvolvimento para Kubernetes

    Para equipes que desenvolvem aplicações destinadas a serem executadas em Kubernetes, o Podman oferece uma transição mais natural através do conceito de pods nativos, permitindo testar localmente configurações semelhantes às do ambiente de produção.

    #### Sistemas de CI/CD seguros

    Implementar Podman em pipelines de CI/CD permite construir e testar containers sem conceder privilégios elevados aos workers, reduzindo os riscos de ataques ou vazamentos.

    #### Ambientes multi-tenant

    Em servidores compartilhados onde múltiplos usuários precisam criar e gerenciar seus próprios containers, o Podman permite isolamento completo sem necessidade de acesso root.

    #### Integração com ferramentas de automação

    O Podman se integra perfeitamente com Ansible, Terraform e outras ferramentas de IaC, permitindo automação completa do ciclo de vida dos containers em ambientes seguros

    Para futuros experimentos, sugerimos explorar:

    1. Configurações avançadas de rede com CNI plugins
    2. Implementação de políticas de segurança com SELinux e seccomp
    3. Quadra-pod: combinação de Podman com Buildah, Skopeo e CRI-O para um pipeline completo de containers
    4. Configuração de registro local para imagens com autenticação
    5. Implementação de health checks e auto-recuperação de containers com systemd

    O Podman representa uma evolução importante na tecnologia de containers, comprovando que é possível obter toda a conveniência e poder dos containers sem comprometer a segurança e a estabilidade do sistema
