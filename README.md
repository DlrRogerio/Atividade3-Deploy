# Projeto DevOps: Deploy Automatizado de API com FastAPI, Jenkins e Kubernetes

## Descrição Geral

Este projeto demonstra, na prática, uma esteira de CI/CD (Integração Contínua e Entrega Contínua) para aplicações FastAPI, utilizando Docker para conteinerização, Jenkins para automação e Kubernetes para orquestração. O objetivo é capacitar profissionais e estudantes a implantar APIs modernas de forma automatizada, segura e escalável.

---

## Sumário

- [Tecnologias Utilizadas](#tecnologias-utilizadas)
- [Pré-requisitos](#pré-requisitos)
- [Estrutura do Projeto](#estrutura-do-projeto)
- [Fase 1: Preparação do Projeto](#fase-1-preparação-do-projeto)
- [Fase 2: Conteinerização com Docker](#fase-2-conteinerização-com-docker)
- [Fase 3: Deploy no Kubernetes](#fase-3-deploy-no-kubernetes)
- [Fase 4: Pipeline Jenkins - Build e Push](#fase-4-pipeline-jenkins---build-e-push)
- [Fase 5: Pipeline Jenkins - Deploy no Kubernetes](#fase-5-pipeline-jenkins---deploy-no-kubernetes)
- [Extras: Plugin Chuck Norris no Jenkins](#extras-plugin-chuck-norris-no-jenkins)
- [Resolução de Problemas](#resolução-de-problemas)
- [Desafios e Extensões](#desafios-e-extensões)
- [Licença](#licença)

---

## Tecnologias Utilizadas

- **GitHub:** Versionamento de código.
- **FastAPI:** Framework web Python moderno.
- **Docker:** Conteinerização da aplicação.
- **Docker Hub:** Registro e distribuição de imagens Docker.
- **Jenkins:** Ferramenta de automação CI/CD.
- **Kubernetes local:** Orquestração de containers (Minikube, Kind, Docker Desktop ou Rancher Desktop).

---

## Pré-requisitos

Antes de começar, instale:

- [Git](https://git-scm.com/)
- [Python 3.8+](https://www.python.org/downloads/)
- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose (opcional)](https://docs.docker.com/compose/install/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Minikube](https://minikube.sigs.k8s.io/docs/start/) ou [Kind](https://kind.sigs.k8s.io/) ou [Kubernetes via Docker Desktop](https://docs.docker.com/desktop/)
- [Jenkins](https://www.jenkins.io/doc/book/installing/) (recomendado rodar via Docker)

---

## Estrutura do Projeto

```
.
├── backend/
│   └── main.py
├── Dockerfile
├── requirements.txt
├── deployment.yaml
├── service.yaml
├── Jenkinsfile
├── docker-compose.yml (opcional)
└── README.md
```

> **Importante:** O código Python fica dentro da pasta `backend`.

---

## Fase 1: Preparação do Projeto

1. **Crie um repositório no GitHub**
   - Crie um novo repositório (público ou privado).
   - Clone para sua máquina:  
     `git clone https://github.com/seu-usuario/nome-do-repo.git`

3. **Crie uma conta no Docker Hub**
   - Acesse [hub.docker.com](https://hub.docker.com/), cadastre-se e anote seu username.

4. **Verifique acesso ao cluster Kubernetes local**

   - Inicie o cluster (exemplo com Minikube):
     ```bash
     minikube start
     ```
   - Verifique se o `kubectl` está instalado e conectado:
     ```bash
     kubectl version --client
     kubectl cluster-info
     kubectl get nodes
     ```
     Você deve ver pelo menos um nó em estado "Ready".

5. **Valide a execução local da aplicação**

   - Instale dependências:
     ```bash
     cd backend
     pip install fastapi uvicorn
     ```
   - Execute:
     ```bash
     uvicorn main:app --reload
     ```
   - Acesse [http://127.0.0.1:8000](http://127.0.0.1:8000) para ver a mensagem e [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) para testar a API.

---

## Fase 2: Conteinerização com Docker

1. **Crie o `requirements.txt`:**

    ```text name=requirements.txt
    fastapi
    uvicorn
    ```

2. **Crie o `Dockerfile` (note a referência ao diretório `backend`):**

    ```Dockerfile name=Dockerfile
    FROM python:3.9-slim-buster
    WORKDIR /app
    COPY requirements.txt .
    RUN pip install --no-cache-dir -r requirements.txt
    COPY backend/ ./backend/
    EXPOSE 80
    CMD ["uvicorn", "backend.main:app", "--host", "0.0.0.0", "--port", "80"]
    ```

3. **(Opcional) Teste local com Docker Compose:**

    ```yaml name=docker-compose.yml
    version: '3.8'
    services:
      fastapi-app:
        build: .
        ports:
          - "8000:80"
        volumes:
          - ./backend:/app/backend
        restart: unless-stopped
    ```

    - Suba o serviço:
      ```bash
      docker-compose up --build -d
      ```
    - Acesse [http://localhost:8000](http://localhost:8000)
    - Para parar:
      ```bash
      docker-compose down
      ```

4. **Build e push da imagem Docker:**

    ```bash
    docker build -t seu-usuario-dockerhub/fastapi-hello:latest .
    docker login
    docker push seu-usuario-dockerhub/fastapi-hello:latest
    ```

    > Substitua `seu-usuario-dockerhub` pelo seu usuário do Docker Hub.

---

## Fase 3: Deploy no Kubernetes

1. **Crie o Deployment YAML:**

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

2. **Crie o Service YAML:**

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

3. **Aplique ao cluster:**

    ```bash
    kubectl apply -f deployment.yaml
    kubectl apply -f service.yaml
    ```

4. **Acesse a aplicação via Kubernetes:**
    - Minikube:
      ```bash
      minikube ip
      # Acesse http://<IP_DO_MINIKUBE>:30001
      ```
    - Docker Desktop/Kind:
      ```
      http://localhost:30001
      ```
    - (Alternativa) Faça port-forward:
      ```bash
      kubectl port-forward service/fastapi-service 8000:80
      # Acesse http://localhost:8000
      # CTRL+C para parar o forward
      ```

---

## Fase 4: Pipeline Jenkins - Build e Push

1. **Instale Jenkins (via Docker):**

    ```bash
    docker run -p 8080:8080 -p 50000:50000 --name jenkins --restart=on-failure \
      -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts-jdk11
    ```

2. **Acesse o Jenkins:**
    - Navegue até [http://localhost:8080](http://localhost:8080)
    - Siga as instruções iniciais (senha, plugins sugeridos, criação de usuário admin).

3. **Configure plugins:**
    - Git, Docker Pipeline, Pipeline, e GitHub Integration.

4. **Adicione credenciais do Docker Hub:**
    - *Manage Jenkins > Credentials > Global > Add Credentials*  
      Tipo: Username with password, ID: `dockerhub-credentials`.

5. **Crie a Pipeline no Jenkins:**
    - *New Item > Pipeline*, configure para buscar o `Jenkinsfile` no seu repositório GitHub.

6. **Exemplo de `Jenkinsfile`:**

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

    > Substitua URLs e IDs conforme sua configuração.

---

## Fase 5: Pipeline Jenkins - Deploy no Kubernetes

1. **Permita que Jenkins acesse o cluster:**
   - Monte o diretório `.kube` do host no container Jenkins (exemplo: `-v ~/.kube:/var/jenkins_home/.kube`)
   - Instale o `kubectl` dentro do container Jenkins:
     ```bash
     docker exec -it jenkins bash
     # Dentro do container:
     apt-get update && apt-get install -y apt-transport-https ca-certificates curl
     curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
     echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | tee /etc/apt/sources.list.d/kubernetes.list
     apt-get update
     apt-get install -y kubectl
     exit
     ```

2. **Adicione o estágio de deploy ao `Jenkinsfile`:**

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

3. **Execute a pipeline completa!**
   - Após rodar, acesse a aplicação conforme explicado na [Fase 3](#fase-3-deploy-no-kubernetes).

---

## Extras: Plugin Chuck Norris no Jenkins

O plugin Chuck Norris adiciona mensagens motivacionais do Chuck Norris em cada build do Jenkins.

**Como instalar:**
1. No Jenkins, vá em **Gerenciar Jenkins > Gerenciar Plugins**.
2. Aba **Disponíveis**, procure por **Chuck Norris**.
3. Marque e instale (não requer reiniciar).
4. Após instalado, ao abrir um Job, você verá uma piada do Chuck Norris na barra lateral do Jenkins.

> **Dica:** O plugin é apenas para diversão, mas traz um toque de humor ao seu pipeline!

---

## Resolução de Problemas

- **kubectl não conecta ao cluster:**  
  Verifique se o cluster está rodando e se o contexto está correto (`kubectl config get-contexts`).
- **Jenkins não acessa Docker ou kubectl:**  
  Verifique se os volumes estão corretamente montados e se as ferramentas estão instaladas no container.
- **Imagem não sobe para Docker Hub:**  
  Confirme o usuário, senha e permissões no Docker Hub, e se a imagem foi corretamente tagueada.

---

## Desafios e Extensões

- **Scanner de Vulnerabilidades:**  
  Integre o [Trivy](https://aquasecurity.github.io/trivy/) à pipeline.
- **Notificações:**  
  Configure alertas por Slack ou Discord em caso de falha na pipeline.
- **SAST:**  
  Use SonarQube para análise de código.
- **Deploy com Helm Chart:**  
  Automatize deploys com Helm.

---

## Licença

[MIT](LICENSE)

---

> **Dica:** Sempre faça commits frequentes e documentados em cada fase do projeto!
