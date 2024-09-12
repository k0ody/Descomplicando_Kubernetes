## Day 9

Hoje iremos explorar o uso do Prometheus em conjunto com o Kubernetes. A agenda inclui:

1. **Kube-prometheus**: Vamos conhecer este projeto, criado pelos mesmos desenvolvedores do Prometheus Operator, que facilita a monitoração de clusters Kubernetes.
2. **Prometheus Adapter**: Aprenderemos a usar o Prometheus como fonte de dados para o Horizontal Pod Autoscaler, permitindo a escalabilidade automática dos pods.
3. **Instalação de Kubernetes**: Vamos aprender a instalar um cluster EKS (Elastic Kubernetes Service) da AWS usando a ferramenta eksctl, que permite a criação de clusters em minutos.

Esses tópicos abrangem desde a monitoração eficiente do cluster até a escalabilidade automática e a configuração de um ambiente Kubernetes na AWS.

### O que é o kube-prometheus?

O kube-prometheus é um conjunto de manifestos do Kubernetes que facilita a instalação e configuração do Prometheus Operator, Grafana, AlertManager, Node Exporter, Kube-State-Metrics e Prometheus-Adapter. Ele oferece uma solução de alta disponibilidade e permite uma visão completa do cluster Kubernetes, monitorando componentes como kube-scheduler, kube-controller-manager, kubelet e kube-proxy.


### Instalando o Cluster Kubernetes no EKS

Para este exemplo, vamos usar o EKS da AWS e a ferramenta `eksctl`.

#### Instalação do `eksctl`

Execute os seguintes comandos para instalar o `eksctl` em uma máquina Linux:
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
```

#### Instalação do AWS CLI

Execute os seguintes comandos para instalar o AWS CLI:
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

#### Configuração das Credenciais AWS

Configure suas credenciais AWS com o comando:
```
aws configure
```

#### Criação do Cluster EKS

Use o `eksctl` para criar um cluster EKS:
```
eksctl create cluster --name=eks-cluster --version=1.24 --region=us-east-1 --nodegroup-name=eks-cluster-nodegroup --node-type=t3.medium --nodes=2 --nodes-min=1 --nodes-max=3 --managed
```

#### Instalação do `kubectl`

Para instalar o `kubectl`, execute:
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

#### Configuração do `kubectl` para o Cluster EKS

Configure o `kubectl` para usar o cluster EKS:
```
aws eks --region us-east-1 update-kubeconfig --name eks-cluster
```

#### Verificação dos Nós do Cluster

Verifique se o `kubectl` está funcionando corretamente:
```
kubectl get nodes
```

#### Comandos Adicionais do `eksctl`

- Listar clusters EKS em todas as regiões:
  ```
  eksctl get cluster -A
  ```

- Listar clusters EKS em uma região específica:
  ```
  eksctl get cluster -r us-east-1
  ```

- Aumentar o número de nós do cluster EKS:
  ```
  eksctl scale nodegroup --cluster=eks-cluster --nodes=3 --nodes-min=1 --nodes-max=3 --name=eks-cluster-nodegroup -r us-east-1
  ```

- Diminuir o número de nós do cluster EKS:
  ```
  eksctl scale nodegroup --cluster=eks-cluster --nodes=1 --nodes-min=1 --nodes-max=3 --name=eks-cluster-nodegroup -r us-east-1
  ```

- Deletar o cluster EKS:
  ```
  eksctl delete cluster --name=eks-cluster -r us-east-1
  ```


### Instalando o Kube-Prometheus

#### Clonar Repositório e Configurar CRDs
Execute os seguintes comandos para

clonar o repositório do Kube-Prometheus e configurar os CRDs:
```
git clone https://github.com/prometheus-operator/kube-prometheus
cd kube-prometheus
kubectl create -f manifests/setup
```

Após a execução, você verá mensagens indicando a criação de vários CRDs e o namespace `monitoring`.

#### Verificar Instalação dos CRDs
Para verificar se a instalação dos CRDs foi concluída, execute:
```
kubectl get servicemonitors -A
```

#### Instalar Prometheus e Alertmanager
Execute o comando abaixo para instalar Prometheus e Alertmanager:
```
kubectl apply -f manifests/
```

Você verá mensagens indicando a criação de vários recursos, incluindo `Alertmanager`, `Grafana`, `Kube-State-Metrics`, `Node Exporter`, `Prometheus`, entre outros.

#### Verificar Instalação
Para verificar se a instalação foi concluída, execute:
```
kubectl get pods -n monitoring
```

O resultado esperado deve ser semelhante a:
```
NAME                                  READY   STATUS    RESTARTS   AGE
alertmanager-main-0                   2/2     Running   0          57s
alertmanager-main-1                   2/2     Running   0          57s
alertmanager-main-2                   2/2     Running   0          57s
blackbox-exporter-cbb9c96b-t8z68      3/3     Running   0          94s
grafana-589787799d-pxsts              1/1     Running   0          80s
kube-state-metrics-557d857c5d-kt8dd   3/3     Running   0          78s
node-exporter-2n6sz                   2/2     Running   0          74s
node-exporter-mwq6b                   2/2     Running   0          74s
prometheus-adapter-758645c65b-54c7g   1/1     Running   0          64s
prometheus-adapter-758645c65b-cmjrv   1/1     Running   0          64s
prometheus-k8s-0                      2/2     Running   0          57s
prometheus-k8s-1                      2/2     Running   0          57s
prometheus-operator-c766b9756-vndp9   2/2     Running   0          63s
```

Pronto, você tem Prometheus, Alertmanager, Blackbox Exporter, Node Exporter e Grafana instalados no seu cluster Kubernetes!


### Acessando o Grafana do Kube-Prometheus

#### Port Forward para Grafana
Para acessar o Grafana localmente, execute o comando:
```
kubectl port-forward -n monitoring svc/grafana 33000:3000
```

#### Acessar Grafana no Navegador
Abra o navegador e acesse a URL: [http://localhost:33000](http://localhost:33000).

#### Login no Grafana
Utilize o usuário `admin` e a senha `admin`. No primeiro login, será solicitado que você altere a senha.

### Explorar os Dashboards

O Kube-Prometheus configura automaticamente uma variedade de dashboards no Grafana que fornecem insights detalhados sobre vários componentes do Kubernetes e do cluster EKS, incluindo:

- **API Server e componentes do Kubernetes**: Dashboards para Node, Pod, Deployment, etc.
- **Cluster EKS**:
  - **Cluster Resources**: CPU e memória usados por todos os nós.
  - **Namespace (Pods)**: CPU e memória usados por todos os pods em todos os namespaces.
  - **Namespace (Workloads)**: CPU e memória usados por deployments, statefulsets e daemonsets.
  - **Node**: CPU e memória usados por todos os nós.
  - **Pod (Containers)**: CPU e memória usados por todos os containers em todos os pods.

Explore a vasta quantidade de dashboards que o Kube-Prometheus oferece e obtenha insights detalhados sobre seu cluster!


### ServiceMonitor no Kube-Prometheus

Um dos principais recursos do Kube-Prometheus é o **ServiceMonitor**, que configura o Prometheus para monitorar serviços específicos. Vamos ver como visualizar e entender um ServiceMonitor.

#### Listar ServiceMonitors
Para listar os ServiceMonitors configurados, use:
```
kubectl get servicemonitors -n monitoring
```

Isso exibirá uma lista como:
```
NAME                      AGE
alertmanager              17m
blackbox-exporter         17m
coredns                   17m
grafana                   17m
kube-apiserver            17m
kube-controller-manager   17m
kube-scheduler            17m
kube-state-metrics        17m
kubelet                   17m
node-exporter             17m
prometheus-adapter        17m
prometheus-k8s            17m
prometheus-operator       17m
```

#### Ver o Conteúdo de um ServiceMonitor
Para visualizar o conteúdo de um ServiceMonitor específico, por exemplo, o `prometheus-k8s`, execute:
```
kubectl get servicemonitor prometheus-k8s -n monitoring -o yaml
```

#### Exemplo de Saída YAML
Aqui está uma versão simplificada do YAML para melhor compreensão:
```
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  labels:
    app.kubernetes.io/component: prometheus
    app.kubernetes.io/instance: k8s
    app.kubernetes.io/name: prometheus
    app.kubernetes.io/part-of: kube-prometheus
    app.kubernetes.io/version: 2.41.0
  name: prometheus-k8s
  namespace: monitoring
spec:
  endpoints:
  - interval: 30s
    port: web
  - interval: 30s
    port: reloader-web
  selector:
    matchLabels:
      app.kubernetes.io/component: prometheus
      app.kubernetes.io/instance: k8s
      app.kubernetes.io/name: prometheus
      app.kubernetes.io/part-of: kube-prometheus
```

#### Explicação do YAML
- `apiVersion`: Versão da API do Kubernetes usada.
- `kind`: Tipo de objeto (ServiceMonitor).
- `metadata`: Informações sobre o objeto, como nome, namespace e labels.
- `spec`: Especificações do ServiceMonitor.
  - `endpoints`: Endpoints monitorados pelo ServiceMonitor.
    - `interval`: Intervalo de coleta de métricas (30s).
    - `port`: Porta usada para coletar métricas (`web` e `reloader-web`).
  - `selector`: Selector para encontrar os serviços monitorados com as labels especificadas.

Com isso, o ServiceMonitor do Prometheus monitora serviços que possuem as labels `app.kubernetes.io/component: prometheus`, `app.kubernetes.io/instance: k8s`, `app.kubernetes.io/name: prometheus` e `app.kubernetes.io/part-of: kube-prometheus`, nas portas `web` e `reloader-web`, com um intervalo de 30 segundos.

### Criando seu Próprio ServiceMonitor
Para criar um ServiceMonitor para um serviço específico, crie um arquivo YAML com as informações necessárias e aplique no cluster com:
```
kubectl apply -f seu-servicemonitor.yaml
```
