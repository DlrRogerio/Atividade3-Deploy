# Projeto DevOps: Deploy Automatizado de API com FastAPI, Jenkins e Kubernetes

**Data de desenvolvimento:** 28/05/2025  
**Data da última atualização:** 04/06/2025

**Autores:**  
- Davi Santos Cardoso da Silva (davi.silva@compasso.com.br)  
- Thiago Geremias de Oliveira (thiago.geremias@compasso.com.br)  

## Descrição Geral

Este projeto tem como objetivo ensinar, na prática, os conceitos de CI/CD utilizando FastAPI, Docker, Jenkins e Kubernetes. Os bolsistas trabalharão com um código base em FastAPI e desenvolverão uma esteira de automação que fará o deploy automatizado da aplicação em um cluster Kubernetes local.

---

## Tecnologias Utilizadas

- **GitHub**: Versionamento de código.
- **FastAPI**: Framework web em Python.
- **Docker**: Conteinerização da aplicação.
- **Docker Hub**: Registro público de imagens.
- **Jenkins**: Ferramenta de CI/CD.
- **Kubernetes local**: Minikube, Kind, Docker Desktop ou Rancher Desktop.

---

## Código Base da Aplicação (Backend)

- [projeto-kubernetes-pb-desafio-jenkins/backend/main.py](https://github.com/box-genius/projetokubernetes-pb-desafio-jenkins/blob/main/backend/main.py)

---

## Fases do Projeto

### Fase 1: Preparação do Projeto

**Atividades:**
- Criar um repositório no GitHub para a aplicação de exemplo.
- Criar conta no Docker Hub.
- Verificar acesso ao cluster Kubernetes local.
- Validar execução local com Uvicorn.

**Entregáveis:**  
- Código rodando localmente.
- Repositório do GitHub criado.
- Ambiente preparado.

---

### Fase 2: Conteinerização com Docker

**Atividades:**
- Executar o backend em containers.
- Publicar a imagem no Docker Hub.
- (Opcional) Criar docker-compose para testes locais.
- Versionar Dockerfile junto ao código no GitHub.

**Comandos sugeridos:**
```sh
docker build -t usuario/fastapi-hello:latest .
docker push usuario/fastapi-hello:latest
```

**Entregáveis:**  
- Imagem publicada no Docker Hub.

---

### Fase 3: Arquivos de Deploy no Kubernetes

**Atividades:**
- Criar arquivo YAML de deployment e aplicar no cluster.
- Criar arquivo YAML de service e aplicar no cluster.

**Entregáveis:**  
- Aplicação exposta em `localhost:30001` via NodePort ou por `port-forward`, rodando a partir do Kubernetes.

---

### Fase 4: Jenkins - Build e Push

**Atividades:**
- Criar pipeline no Jenkins.
- Realizar stages de build e push.
- Configurar acionamento automático da pipeline ao realizar git push.

**Entregáveis:**  
- Pipeline funcional no Jenkins até o push da imagem.

---

### Fase 5: Jenkins - Deploy no Kubernetes

**Atividades:**
- Configurar Jenkins para acesso ao `kubectl` e ao cluster local.
- Adicionar etapa de deploy no Jenkinsfile.
- Testar pipeline completa.

**Entregáveis:**  
- Pipeline completa com deploy automatizado.

---

### Fase 6: Documentação

**Atividades:**
- Criar `README.md` detalhado com:
  - Passos para reprodução do projeto.
  - Tecnologias utilizadas.
  - Prints do Jenkins e dos testes.
  - Apresentação da pipeline funcionando (build, push, deploy).

**Entregáveis:**  
- Documentação completa e apresentação final.

---

## Desafios Extras

- Adicionar etapa de scanner de vulnerabilidades (Trivy) após o push da imagem.
- Criar webhook (Slack/Discord) para avisos de atualização no ambiente Kubernetes.
- Subir SonarQube em Docker, conectar ao Jenkins e realizar análise SAST.
- Utilizar Helm Chart para implantar a aplicação no Kubernetes.

---

## Referências

- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [Docker Documentation](https://docs.docker.com/)
- [Jenkins Documentation](https://www.jenkins.io/doc/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
