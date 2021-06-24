# Kubernetes

Kubernetes é um sistema de orquestração de contêineres open-source que automatiza a implantação, o dimensionamento e a gestão de aplicações em contêineres.

**Conceitos Fundamentais Kubernetes:**

- **Control Pane:** Onde é gerenciado o controle dos processos dos Nodes (similar ao host manager do Docker Swarm);
- **Nodes:** Máquinas que são gerenciadas pelo Control Pane;
- **Deployment:** A execução de uma imagem/projeto em um Pod;
- **Pod:** Um ou mais containers que estão em um Node;
- **Services:** Serviços que expõe os Pods ao mundo externo;
- **kubectl:** Cliente de linha de comando para o Kubernetes.

Dependências Necessárias:

**kubectl:** Ferramente de linha de comando que permite controlar Clusters Kubernetes

**Minikube:** O Minikube é uma implementação leve do Kubernetes que cria uma VM em sua máquina local e implanta um cluster simples contendo apenas um nó. O Minikube está disponível para sistemas Linux, macOS e Windows. Resumidamente, o Minikube é uma espécie de simulador de Kubernetes, para que não precisemos de vários computadores/servidores.

**Instalação do Kubernetes Linux:**

[https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)

**Instalação do Kubernetes Windows:**

[https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/)

**Instalação Minikube:**

[https://minikube.sigs.k8s.io/docs/start/](https://minikube.sigs.k8s.io/docs/start/)

**\*Obs.:** No Windows preferir instalação via Chocolatey.\*

**Inicializando o Minikube:**

```bash
# drivers disponíveis -> virtualbox, hyperv, docker
# inicializar toda vez que o PC for reiniciado
minikube start --driver=<DRIVER>

# parar o minikube
minikube stop

# testar minikube
minikube status
```

O Minikube também disponibiliza uma dashboard para visualização e controle dos serviços, pods deployments etc.

**Acessando a Dashboard do Minikube:**

```bash
# inicia a dashboard e abre a página no navegador
minikube dashboard

# inicia a dashboard e exibe a URL de acesso
minikube dashboard -url
```

**Deployment Kubernetes:** Com deployments criamos nosso serviço que vai rodar nos Pods. Nele definimos uma imagem e um nome, para posteriormente ser replicado entre os servidores. A partir da criação do deployment teremos containers rodando. A imagem usada para gerar o deployment deve ser uma imagem de um repositório no Hub do Docker.

```bash
# criação do deployment que rodará uma image do Docker Hub
kubectl create deployment <NOME> --image=<IMAGEM DOCKER HUB>

# visualizar deployments ativos no Kubernetes
kubectl get deployments

# visualizar detalhes de um deployment
kubectl describe deployments

# visualizar pods ativos no Kubernetes
kubectl get pods

# visualizar detalhes de um pod
kubectl describe pods

# verificar como o Kubernetes está configurado
kubectl config view
```

**Services Kubernetes:** As aplicações do Kubernetes não tem conexão com o mundo externo. Por isso, é necessário criar um Service, que é o que possibilita expor os Pods. O Service é um entidade separada dos Pods, que expõe eles a uma rede.

```bash
# expor um Pod
kubectl expose deployment <NOME> --type=<TIPO> --port=<PORT>
```

- Tipo do Serviço: há vários tipos de serviços que podem ser usados, porém o LoadBalancer é o mais comum, onde todos os Pods são expostos em modo de balanceamento de carga.

**Gerando um IP pelo Minikube para que o serviço fique acessível:**

```bash
# cria serviço no minikube e disponibiliza um IP para acesso
minikube service <NOME>

# visualizar detalhes dos serviços
kubectl get services

# visualizar detalhes de um serviço em específico
kubectl describe services/<NOME>
```

**Escalando aplicação no Kubernetes:**

```bash
# escalar aplicação informando o número de replicas (ou diminuir)
kubectl scale deployment/<NOME> —replicas=<NUMERO>

# visualizar pods rodando
kubectl get pods

# checar número de réplicas
kubectl get rs
```

**Atualização de Imagem:**

Para atualizar a imagem de um deployment, a nova imagem dever ser uma outra versão da atual, ou seja, é preciso dar um push na imagem com uma nova tag.

```bash
kubectl set image deployment/<NOME> <NOME_CONTAINER>=<NOVA_IMAGEM>
```

**Desfazer Alteração:**

Para desfazer uma alteração utilizamos uma ação conhecida como rollback.

```bash
# verificar uma alteração
kubectl rollout status deployment/<NOME>

# desfazer alteração
kubectl rollout undo deployment/<NOME>
```

**Deletar Serviço:**

Desta maneira nossos Pods não terão mais a conexão externa, ou seja, não poderemos mais acessar eles.

```bash
kubectl delete service <NOME>
```

**Deletar Deployment:**

Desta maneira o container não estará mais rodando, ou seja, para acessar novamente o projeto precisaremos criar um deployment com a mesma ou outra imagem, para acessar algum projeto.

```bash
kubectl delete deployment <NOME>
```

## **Modo Declarativo Kubernetes:**

Todos os passos anteriores são considerados comandos do **_modo imperativo_** do Kubernetes, ou seja, iniciamos a aplicação através de comandos no terminal. O _modo declarativo_ é guiado por um arquivo YAML, semelhante ao _docker-compose_. Desta maneira tornamos as configurações mais simples e centralizamos tudo em um único comando.

**Chaves mais utilizadas:**

- **apiVersion:** versão utilizada da ferramenta;
- **kind:** tipo do arquivo (Deployment, Service);
- **metadata:** descrever algum objeto, inserindo chaves como name;
- **replicas:** número de réplicas de Nodes/Pods;
- **containers:** definir as especificações de containers como nome e imagem, por exemplo.

Exemplo de arquivo .yaml para criação de um **_deployment_** no Kubernetes:

```yaml
apiVertsion: apps/v1
kind: Deployment
metadata:
  name: flask-app-deployment
spec:
  replicas: 4
  selector:
    matchLabels:
      app: flask-app
  template:
    metadata:
      labels:
        app: flask-app
    spec:
      containers:
        - name: flask
          image: thauam09/flask-kub-projeto:2
```

Exemplo de arquivo .yaml para criação de um **_service_** para o deployment anterior no Kubernetes:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: flask-service
spec:
  selector:
    app: flask-app
  ports:
    - protocol: "TCP"
      port: 5000
      targetPort: 5000
  type: LoadBalancer
```

Para executar um arquivo .yaml do Kubernetes, tanto para Deployments quanto para Services, basta rodar o comando:

```bash
# execução de comandos no modo declarativo por meio de arquio yaml
kubectl apply -f <ARQUIVO>

# deleção do deployment/service no modo declarativo
kubectl delete -f <ARQUIVO>

# visualizar execução do container através do minikube
minikube service flask-service
```

**Atualizando o projeto no modo declarativo:**

Para atualizar a imagem de um deployment feito através de um arquivo .yaml basta alterar a tag da imagem no arquivo de configuração e, em seguida, replicar o comando apply novamente para que o deployment seja reiniciado com a nova imagem.

### Unindo arquivos do projeto

Semelhante ao docker-compose, no Kubernetes é possível unir um deployment e um service em um único arquivo, para que uma aplicação seja disponibilizada de forma mais fácil e organizada em apenas um lugar.

**\*Obs.:** Uma boa prática é colocar o service antes do deployment no arquivo .yaml.\*

A separação dos objetos para o YAML é com "—-". Exemplo:

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: flask-service
spec:
  selector:
    app: flask-app
  ports:
    - protocol: "TCP"
      port: 5000
      targetPort: 5000
  type: LoadBalancer
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-app-deployment
spec:
  replicas: 4
  selector:
    matchLabels:
      app: flask-app
  template:
    metadata:
      labels:
        app: flask-app
    spec:
      containers:
        - name: flask
          image: thauam09/flask-kub-projeto:3
```
