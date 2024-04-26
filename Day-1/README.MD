# Day 1


## Introdução

Antes de falarmos do kubernetes precisamos entender um pouco mais sobre o ecossistema do kubernetes.

O Container Engine é essencial no ecossistema do Kubernetes, gerenciando imagens, volumes e garantindo o isolamento dos recursos dos containers. Opções como Docker, CRI-O e Podman são populares, cada um utilizando o runc como runtime padrão. O Docker, como todos sabem, é o Container Engine mais popular e ele utiliza como Container Runtime o containerd.

A OCI (Open Container Initiative) padroniza a criação de containers, com o runc como seu principal projeto de runtime, utilizado pelo Docker.

O Container Runtime é necessário para executar os containers nos nós, podendo ser de baixo nível (como runc), alto nível (como containerd, CRI-O) ou em sandbox (para segurança adicional, como gVisor). Existem também runtimes virtualizados, como o Kata Containers, para execução em máquinas virtuais.


## O que é Kubernetes

Kubernetes é uma plataforma de código aberto para automação, implantação, escalonamento e gerenciamento de aplicativos em contêineres. Ele fornece uma estrutura para implantar aplicativos em vários hosts, oferecendo recursos como balanceamento de carga, auto-recuperação e dimensionamento automático, facilitando a orquestração eficiente de contêineres em ambientes de nuvem e em infraestruturas locais.

## Arquitetura do Kubernetes

A arquitetura do Kubernetes é composta por vários componentes que trabalham juntos para fornecer uma plataforma de orquestração de contêineres. Os principais componentes incluem:

**Master Node/Control Plane:** Este é o ponto central do cluster Kubernetes e é responsável por coordenar todas as operações no cluster. Ele consiste em:

**API Server:** Componente que expõe a API do Kubernetes, permitindo que os usuários e os componentes do sistema interajam com o cluster.
**Controller Manager:** Gerencia os controladores que regulam o estado do cluster.
**Scheduler:** Responsável por distribuir os pods (unidades básicas de implantação no Kubernetes) nos nodes do cluster com base nos requisitos de recursos e outras restrições.
**etcd:** Um armazenamento de chave-valor consistente usado para armazenar o estado do cluster Kubernetes.

**Node(s)/Workers:** São as máquinas individuais (físicas ou virtuais) que compõem o cluster Kubernetes. Cada nó executa os seguintes componentes:

**Kubelet:** Agente que executa nos nós e gerencia os pods e seus contêineres.

**Kube-proxy:** Responsável por encaminhar o tráfego de rede para os pods.

**Container Runtime:** O software responsável por executar os contêineres, como Docker ou containerd.

Além desses componentes principais, há outros recursos e objetos no Kubernetes, como:

**Pods:** Um grupo de um ou mais contêineres que compartilham armazenamento e uma única configuração de rede. É a menor unidade implantável no Kubernetes.

**Service:** Define um conjunto lógico de pods e uma política de acesso a eles.

**Namespace:** Um mecanismo para dividir recursos de cluster entre vários usuários ou finalidades.

**Deployment:** Gerencia a implantação de aplicativos em contêineres, permitindo atualizações sem tempo de inatividade.

Esses componentes trabalham juntos para facilitar a implantação, escalonamento e gerenciamento de aplicativos em contêineres em ambientes Kubernetes.

A comunicação entre os componentes do Kubernetes ocorre principalmente através de APIs e protocolos específicos, e não necessariamente através de portas específicas. No entanto, algumas portas são comumente utilizadas para comunicação interna dentro do cluster e para interação externa:

- **kube-api-server:** 6443 TCP
- **etcd:** 2379-2380 TCP
- **kube-scheduler:** 10251 TCP
- **kubelet:** 10250 TCP
- **kube-control-plane:** 10252 TCP
- **NodePort:** 30000-32767 - TCP
- **Weave net:** 6783-6784 TCP/UDP

**Conceitos-chave do Kubernetes (k8s)**

O Kubernetes (k8s) tem conceitos-chave que o diferenciam de outros orquestradores, como o Docker Swarm. Ele organiza contêineres em "pods", abstrações que compartilham recursos. Principais conceitos incluem:

- **Pod**: Menor unidade do k8s, agrupando contêineres e compartilhando recursos como endereços, volumes e recursos de CPU e memória.

- **Deployment**: Controla a execução de um número específico de réplicas de um pod nos nós do cluster, gerenciando o ciclo de vida das aplicações e permitindo configurações detalhadas.

- **ReplicaSets**: Garante a quantidade correta de pods em execução nos nós do cluster, trabalhando em conjunto com os deployments.

- **Services**: Exponencia a comunicação entre pods, utilizando ClusterIP, NodePort ou LoadBalancer para distribuir as requisições entre eles, funcionando como um balanceador de carga.


## Laboratorio

Para os exemplos vamos utilizar o Kind (Kubernetes IN Docker) é uma ferramenta que permite criar clusters k8s locais usando containers como o "nós". É usado para desenvolvimento e testes.

Segue passo a passo da instalação em ambientes Linux

```
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.14.0/kind-linux-amd64

chmod +x ./kind

sudo mv ./kind /usr/local/bin/kind

```
Para criar um cluster o comando é:

```
kind create cluster
```

Para criar multiplos nós, como control-plane e workers, precisamos criar um arquivo de configuração .yaml. Abaixo um exemplo:

```
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
```

Dessa forma ele ira criar um nó do tipo control-plane/master node e outros dois nós workes. Para iniciar o esse cluster precismos antes deletar o anterior:

```
kind delete cluster
```

Apos isso podemos iniciar nosso cluster a partir do nosso arquivo .yaml:

```
kind create cluster --config <nome do arquivo.yaml> --name <nome do cluester>
```

## Kubectl

O kubectl é uma ferramenta de linha de comando para interagir com clusters Kubernetes, permitindo aos usuários gerenciar e operar recursos do Kubernetes, como implantações de aplicativos, escalabilidade e monitoramento.

Para instalar o kubectl no linux:

```
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl

chmod +x ./kubectl

sudo mv ./kubectl /usr/local/bin/kubectl

kubectl version --client
```

Após isso ja podemos realizar os testes e brincar com o ambiente.