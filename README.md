# ☸️ Deploy da Aplicação Spring Boot + PostgreSQL com Kustomize e ArgoCD

Este repositório contém os **manifestos Kubernetes** utilizados para implantar a aplicação **Spring Boot `funcionarios`** e seu banco de dados **PostgreSQL**.  
O gerenciamento de versões é feito com **Kustomize**, e o deploy contínuo é automatizado via **ArgoCD**, que detecta alterações neste repositório e atualiza o cluster automaticamente.

---

## 🧩 Estrutura do Repositório

```
.
└── kubernetes/
    └── app.yaml            # Deployment + Service da aplicação Spring Boot
    └── database.yaml       # Deployment + Service + PVC do PostgreSQL
    └── kustomization.yaml  # Controla o build com Kustomize
```

---

## ⚙️ Pré-requisitos

Antes de iniciar, é necessário ter instalado localmente:

| Ferramenta | Função | 
|-------------|---------|
| [Docker](https://www.docker.com/) | Criação de containers |
| [Kind](https://kind.sigs.k8s.io/) | Cluster Kubernetes local | 
| [kubectl](https://kubernetes.io/docs/tasks/tools/) | CLI Kubernetes |
| [Kustomize](https://kubectl.docs.kubernetes.io/installation/kustomize/) | Gerenciador de manifestos |

---

## 🚀 Como subir o ambiente local

### 1️⃣ Criar o cluster Kubernetes com Kind

```bash
kind create cluster --name funcionarios-cluster
```

Verifique o contexto:

```bash
kubectl cluster-info --context kind-funcionarios-cluster
```

---

### 2️⃣ Instalar o ArgoCD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Acompanhe os pods do ArgoCD:

```bash
kubectl get pods -n argocd
```

---

### 3️⃣ Expor o ArgoCD localmente

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Acesse no navegador:  
👉 [https://localhost:8080](https://localhost:8080)

Obtenha a senha inicial:
```bash
kubectl -n argocd get secret argocd-initial-admin-secret   -o jsonpath="{.data.password}" | base64 -d && echo
```
ou instalando [argo cli](https://argo-cd.readthedocs.io/en/stable/cli_installation/) e usando comando abaixo:
```bash
argocd admin initial-password -n argocd
```
Usuário: `admin`

---

### 4️⃣ Configurar a aplicação no ArgoCD

No painel do ArgoCD:
- **NEW APP**
  - **Application Name:** funcionarios-app  
  - **Project:** default  
  - **Repository URL:** `https://github.com/amakusashirou/funcionarios-k8s.git` *(exemplo)*  
  - **Path:** `/kubernetes`  
  - **Cluster URL:** `https://kubernetes.default.svc`  
  - **Namespace:** `funcionariosapp`  
  - **Sync Policy:** *Automatic (recomendado)*  

Salve e sincronize. O ArgoCD criará todos os recursos automaticamente.

---

## 🧱 Estrutura dos manifestos

### `app.yaml`
Contém o **Deployment** e **Service** da aplicação Spring Boot.

### `database.yaml`
Contém o banco de dados PostgreSQL e o PVC para persistência.

### `kustomization.yaml`
Controla a composição dos recursos e a imagem da aplicação.

---

## 🔁 Como funciona a atualização automática

1. O repositório da **aplicação Java** faz o build da imagem Docker (`amakusashirou/funcionarios`) e a envia para o Docker Hub.  
2. O pipeline (GitHub Actions) atualiza o campo `newTag` dentro deste `kustomization.yaml`.  
3. Ao detectar a alteração no GitHub, o **ArgoCD** aplica automaticamente a nova versão no cluster.  

👉 Nenhuma ação manual é necessária.

---

## 🧪 Teste manual (opcional)

```bash
kubectl apply -k kubernetes/
kubectl get pods -n funcionariosapp
kubectl get svc -n funcionariosapp
```
### 🚀 Acessar a aplicação

A aplicação pode ser acessada localmente utilizando o comando abaixo para realizar o port-forward do serviço `spring-app`:

```bash
kubectl port-forward svc/spring-app 8080:8080 --address=0.0.0.0
```
---

## 🧹 Comandos úteis

| Ação | Comando |
|------|----------|
| Criar cluster Kind | `kind create cluster --name funcionarios-cluster` |
| Apagar cluster | `kind delete cluster --name funcionarios-cluster` |
| Aplicar manifestos manualmente | `kubectl apply -k kubernetes/` |
| Ver pods e serviços | `kubectl get pods,svc -n funcionariosapp` |
| Expor ArgoCD | `kubectl port-forward svc/argocd-server -n argocd 8080:443` |

---
