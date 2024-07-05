## Day 11

O Ingress é um recurso do Kubernetes que gerencia o acesso externo aos serviços dentro de um cluster. Ele atua como uma camada de roteamento HTTP/HTTPS, definindo regras para direcionar o tráfego externo para diferentes serviços back-end. Implementado através de um controlador de Ingress, ele suporta várias soluções como NGINX, Traefik e Istio.

Principais componentes e funcionalidades:

- **Controlador de Ingress:** Implementação real que aplica as regras do Ingress, como NGINX ou HAProxy.
- **Regras de Roteamento:** Definidas em YAML, determinam como as requisições externas são encaminhadas aos serviços internos.
- **Backend Padrão:** Serviço de fallback para requisições que não correspondem a nenhuma regra.
- **Balanceamento de Carga:** Distribui automaticamente o tráfego entre múltiplos pods de um serviço.
- **Terminação SSL/TLS:** Configura certificados SSL/TLS para criptografia no ponto de entrada do cluster.
- **Anexos de Recurso:** Permite adicionar ConfigMaps ou Secrets para configurações adicionais como autenticação e controle de acesso.

Essas funcionalidades tornam o Ingress uma ferramenta poderosa para gerenciar o tráfego e a segurança dos serviços dentro de um cluster Kubernetes.

### Usando KinD com Ingress

#### Introdução
KinD é uma ferramenta útil para testes e desenvolvimento com Kubernetes. Nesta seção, fornecemos detalhes para garantir que o Ingress funcione corretamente em um cluster KinD.

#### Criando o Cluster com Configurações Especiais
Crie um arquivo chamado `kind-config.yaml` com o seguinte conteúdo:

```
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
```

Crie o cluster usando este arquivo de configuração:

```
kind create cluster --config kind-config.yaml
```

#### Instalando um Ingress Controller
Vamos usar o Nginx Ingress Controller:

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/kind/deploy.yaml
```

Espere até que os pods do Ingress Controller estejam prontos:

```
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s
```

Nosso comando espera que os pods do Ingress Controller estejam prontos no namespace `ingress-nginx`, com o rótulo `app.kubernetes.io/component=controller`, e falha após 90 segundos se não estiverem prontos.


### Instalando Ingress Controller em Diferentes Plataformas

#### Minikube
Para instalar o Ingress Controller no Minikube, use o sistema de addons do próprio Minikube:

```
minikube addons enable ingress
```

#### MicroK8s
Para o MicroK8s, instale o Ingress Controller via sistema de addons:

```
microk8s enable ingress
```

#### AWS (Amazon Web Services)
Na AWS, o Ingress Controller é exposto através de um Network Load Balancer (NLB):

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/aws/deploy.yaml
```

Você pode configurar a terminação TLS no Load Balancer editando o `deploy.yaml` com as informações do seu Certificate Manager (ACM).

#### Azure
No Azure, instale o Ingress Controller com o comando:

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/cloud/deploy.yaml
```

#### GCP (Google Cloud Platform)
No GCP, certifique-se de que seu usuário tem permissões de cluster-admin e instale o Ingress Controller:

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/cloud/deploy.yaml
```

Pode ser necessário configurar regras de firewall se estiver usando um cluster privado.

#### Bare Metal
Para instalações em Bare Metal ou VMs "cruas", use um NodePort para testes rápidos:

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/baremetal/deploy.yaml
```

Este comando mapeará o Ingress Controller para uma porta no intervalo de 30000-32767.

### Arquivos utilizados para faze o deploy do App

### app-deployment.yaml

```
apiVersion: apps/v1                # Especifica a versão da API usada para criar o Deployment
kind: Deployment                   # Tipo de recurso, neste caso, um Deployment
metadata:
  labels:
    app: giropops-senhas           # Rótulo identificador para o Deployment
  name: giropops-senhas            # Nome do Deployment
spec:
  replicas: 2                      # Número de réplicas/pods a serem criados
  selector:
    matchLabels:
      app: giropops-senhas         # Seleciona os pods que possuem o rótulo app: giropops-senhas
  template:
    metadata:
      labels:
        app: giropops-senhas       # Rótulo identificador para os pods criados
    spec:
      containers:
      - image: linuxtips/giropops-senhas:1.0  # Imagem Docker do container
        name: giropops-senhas                  # Nome do container
        env:
        - name: REDIS_HOST                     # Variável de ambiente no container
          value: redis-service                 # Valor da variável de ambiente REDIS_HOST
        ports:
        - containerPort: 5000                  # Porta exposta pelo container
        imagePullPolicy: Always                # Política para puxar a imagem: Always garante que a última versão será puxada sempre
```

### app-service.yaml

```
apiVersion: v1                    # Versão da API usada para criar o Service
kind: Service                     # Tipo de recurso, neste caso, um Service
metadata:
  name: giropops-senhas           # Nome do Service
  labels:
    app: giropops-senhas          # Rótulo identificador para o Service
spec:
  selector:
    app: giropops-senhas          # Seleciona os pods com o rótulo app: giropops-senhas
  ports:
    - protocol: TCP               # Protocolo usado pelo Service
      port: 5000                  # Porta exposta pelo Service
      targetPort: 5000            # Porta do container que o Service direciona o tráfego
      name: tcp-app               # Nome do mapeamento da porta
  type: ClusterIP                 # Tipo de Service, ClusterIP torna o Service acessível apenas dentro do cluster
```

### redis-deployment.yaml

```
apiVersion: apps/v1                # Especifica a versão da API usada para criar o Deployment
kind: Deployment                   # Tipo de recurso, neste caso, um Deployment
metadata:
  labels:
    app: redis                     # Rótulo identificador para o Deployment
  name: redis-deployment           # Nome do Deployment
spec:
  replicas: 1                      # Número de réplicas/pods a serem criados
  selector:
    matchLabels:
      app: redis                   # Seleciona os pods que possuem o rótulo app: redis
  template:
    metadata:
      labels:
        app: redis                 # Rótulo identificador para os pods criados
    spec:
      containers:
      - image: redis               # Imagem Docker do container
        name: redis                # Nome do container
        ports:
          - containerPort: 6379    # Porta exposta pelo container
        resources:                 # Define limites e requisições de recursos
          limits:
            memory: "256Mi"        # Limite máximo de memória que o container pode usar
            cpu: "500m"            # Limite máximo de CPU que o container pode usar
          requests:
            memory: "128Mi"        # Memória solicitada pelo container ao iniciar
            cpu: "250m"            # CPU solicitada pelo container ao iniciar
```

### redis-service.yaml

```
apiVersion: v1                    # Versão da API usada para criar o Service
kind: Service                     # Tipo de recurso, neste caso, um Service
metadata:
  name: redis-service             # Nome do Service
spec:
  selector:
    app: redis                    # Seleciona os pods com o rótulo app: redis
  ports:
    - protocol: TCP               # Protocolo usado pelo Service
      port: 6379                  # Porta exposta pelo Service
      targetPort: 6379            # Porta do container que o Service direciona o tráfego
  type: ClusterIP                 # Tipo de Service, ClusterIP torna o Service acessível apenas dentro do cluster
```

### Criando um Recurso de Ingress

#### Ingress Manifest (ingress-1.yaml)

```
apiVersion: networking.k8s.io/v1                  # Versão da API usada para criar o Ingress
kind: Ingress                                     # Tipo de recurso, neste caso, um Ingress
metadata:
  name: giropops-senhas                           # Nome do Ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: / # Anotação para reescrever a URL de destino
spec:
  rules:
  - http:
      paths:
      - path: /giropops-senhas                    # Caminho da URL que o Ingress deve gerenciar
        pathType: Prefix                          # Tipo de caminho, Prefix corresponde a /giropops-senhas e qualquer subcaminho
        backend:
          service:
            name: giropops-senhas                 # Nome do Service a ser encaminhado
            port:
              number: 5000                        # Porta do Service a ser encaminhado
```

#### Aplicar o Ingress

```
kubectl apply -f ingress-1.yaml                   # Aplica a configuração do Ingress definida no arquivo ingress-1.yaml
```

#### Verificar a Criação do Ingress

```
kubectl get ingress                               # Lista todos os recursos Ingress no cluster
```

#### Detalhes do Ingress

```
kubectl describe ingress ingress-1                # Exibe detalhes do recurso Ingress especificado (ingress-1)
```

#### Obter o Endereço IP do Ingress

```
kubectl get ingress ingress-1 -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
# Obtém o hostname do Ingress, útil para clusters gerenciados que fornecem um hostname

kubectl get ingress ingress-1 -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
# Obtém o endereço IP do Ingress, útil para clusters gerenciados que fornecem um endereço IP
```

#### Testar o Ingress

```
curl ENDEREÇO_DO_INGRESS/giropops-senhas          # Testa o Ingress usando o IP, hostname ou Load Balancer do Ingress
```

### Explicações

- **apiVersion**: Define a versão da API para o recurso.
- **kind**: Especifica o tipo de recurso (Ingress).
- **metadata**: Metadados do recurso, incluindo o nome e anotações.
- **annotations**: Fornece informações adicionais para o Ingress Controller, como regras de reescrita de URL.
- **spec**: Especifica a configuração do Ingress.
- **rules**: Define as regras de roteamento HTTP.
- **paths**: Lista de caminhos gerenciados pelo Ingress.
- **pathType**: Tipo de correspondência de caminho (Prefix).
- **backend**: Serviço de backend para onde o tráfego deve ser direcionado.
- **service**: Nome do serviço Kubernetes e a porta a ser usada.

Os comandos permitem aplicar a configuração do Ingress, verificar sua criação e detalhes, obter o endereço IP ou hostname, e testar o roteamento configurado.

