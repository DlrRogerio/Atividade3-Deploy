# Projeto DevOps: Deploy Automatizado de API com FastAPI, Jenkins e Kubernetes

## Descrição Geral

Este projeto tem como objetivo ensinar na prática os conceitos de CI/CD (Integração Contínua/Entrega Contínua) utilizando FastAPI, Docker, Jenkins e Kubernetes. A aplicação é desenvolvida em FastAPI, conteinerizada com Docker, automatizada com Jenkins e implantada em um cluster Kubernetes local.

---

## Tecnologias Utilizadas

- **Github**: Versionamento de código.
- **FastAPI**: Framework web Python.
- **Docker**: Conteinerização.
- **Docker Hub**: Registro e distribuição de imagens Docker.
- **Jenkins**: Ferramenta de CI/CD.
- **Kubernetes local**: Minikube, Kind, Docker Desktop ou Rancher Desktop.

---

## Código Base da Aplicação (Backend)

O código base está em `backend/main.py`.  
Exemplo mínimo:

```python name=backend/main.py
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"Hello": "World"}
```

---

## Fases do Projeto

### Fase 1: Preparação do Projeto

1. **Criar repositório no GitHub** e subir `main.py`.
2. **Criar conta no Docker Hub** para publicar imagens.
3. **Verificar acesso ao cluster Kubernetes local** (Minikube/Kind/Docker Desktop/Rancher Desktop).
4. **Validar execução local**:
    - Instale dependências:
      ```bash
      pip install fastapi uvicorn
      ```
    - Execute:
      ```bash
      uvicorn main:app --reload
      ```
    - Acesse [http://127.0.0.1:8000](http://127.0.0.1:8000) e [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)

---

### Fase 2: Conteinerização com Docker

1. **Crie o `Dockerfile`**:

    ```Dockerfile name=Dockerfile
    FROM python:3.9-slim-buster
    WORKDIR /app
    COPY requirements.txt .
    RUN pip install --no-cache-dir -r requirements.txt
    COPY . .
    EXPOSE 80
    CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "80"]
    ```

2. **Crie o `requirements.txt`**:

    ```text name=requirements.txt
    fastapi
    uvicorn
    ```

3. **(Opcional) Teste local com Docker Compose**:

    ```yaml name=docker-compose.yml
    version: '3.8'
    services:
      fastapi-app:
        build: .
        ports:
          - "8000:80"
        volumes:
          - .:/app
        restart: unless-stopped
    ```

    ```bash
    docker-compose up --build -d
    # Para parar
    docker-compose down
    ```

4. **Build e push da imagem Docker**:

    ```bash
    docker build -t seu-usuario-dockerhub/fastapi-hello:latest .
    docker login
    docker push seu-usuario-dockerhub/fastapi-hello:latest
    ```

---

### Fase 3: Arquivos de Deploy no Kubernetes

1. **Deployment YAML**:

    ```yaml name=deployment.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: fastapi-deployment
      labels:
        app: fastapi-app
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: fastapi-app
      template:
        metadata:
          labels:
            app: fastapi-app
        spec:
          containers:
          - name: fastapi-container
            image: seu-usuario-dockerhub/fastapi-hello:latest
            ports:
            - containerPort: 80
    ```

2. **Service YAML**:

    ```yaml name=service.yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: fastapi-service
      labels:
        app: fastapi-app
    spec:
      selector:
        app: fastapi-app
      ports:
        - protocol: TCP
          port: 80
          targetPort: 80
          nodePort: 30001
      type: NodePort
    ```

3. **Aplicar no cluster**:

    ```bash
    kubectl apply -f deployment.yaml
    kubectl apply -f service.yaml
    ```

4. **Acessar aplicação**:  
   - Minikube:  
     ```bash
     minikube ip
     # Acesse http://<IP_DO_MINIKUBE>:30001
     ```
   - Docker Desktop/Kind:  
     http://localhost:30001

---

### Fase 4: Jenkins - Build e Push

1. **Instale Jenkins** (exemplo via Docker):

    ```bash
    docker run -p 8080:8080 -p 50000:50000 --name jenkins --restart=on-failure \
      -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts-jdk11
    ```

2. **Configure plugins e credenciais** (GitHub e Docker Hub).

3. **Crie o Jenkinsfile**:

    ```groovy name=Jenkinsfile
    pipeline {
        agent any
        environment {
            DOCKER_CREDENTIALS_ID = 'dockerhub-credentials'
            DOCKER_IMAGE_NAME = "seu-usuario-dockerhub/fastapi-hello"
        }
        parameters {
            string(name: 'DOCKER_IMAGE_TAG', defaultValue: 'latest', description: 'Tag da imagem Docker')
        }
        stages {
            stage('Checkout SCM') {
                steps {
                    cleanWs()
                    git branch: 'main', credentialsId: '', url: 'https://github.com/seu-usuario/seu-repositorio.git'
                }
            }
            stage('Build Docker Image') {
                steps {
                    sh "docker build -t ${DOCKER_IMAGE_NAME}:${params.DOCKER_IMAGE_TAG} ."
                }
            }
            stage('Push Docker Image') {
                steps {
                    withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDENTIALS_ID, passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh "docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}"
                        sh "docker push ${DOCKER_IMAGE_NAME}:${params.DOCKER_IMAGE_TAG}"
                        sh "docker logout"
                    }
                }
            }
        }
        post {
            always { cleanWs() }
        }
    }
    ```

---

### Fase 5: Jenkins - Deploy no Kubernetes

1. **Permita que o Jenkins acesse o `kubectl`** (monte `.kube` e instale o `kubectl` no container Jenkins).
2. **Adicione estágio final ao Jenkinsfile**:

    ```groovy
    stage('Deploy to Kubernetes') {
        steps {
            sh "kubectl apply -f deployment.yaml"
            sh "kubectl apply -f service.yaml"
            sh "kubectl rollout restart deployment/fastapi-deployment"
            sh "kubectl rollout status deployment/fastapi-deployment"
        }
    }
    ```

3. **Acione a pipeline e valide o deploy acessando o serviço no Kubernetes.**

---

### Fase 6: Documentação

1. **Atualize este README.md** com todas as etapas, prints de telas, comandos, dicas e desafios extras (Trivy, Slack, SonarQube, Helm).

---

## Prints do Projeto

Inclua aqui screenshots do Jenkins, Docker Hub, aplicação rodando localmente e no Kubernetes, saídas do `kubectl`, etc.

---

## Desafios Extras (Opcional)

- Scanner de Vulnerabilidades com Trivy
- Notificações via Slack/Discord
- Análise SAST com SonarQube
- Deploy com Helm Chart

---

## Licença

[MIT](LICENSE)
