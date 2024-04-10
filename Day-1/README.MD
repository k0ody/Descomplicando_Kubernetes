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

Abaixo os comandos mais usados:

1. **kubectl create**:
   - Descrição: Cria recursos Kubernetes a partir de arquivos de configuração YAML ou JSON.
   - Exemplo de uso: `kubectl create -f arquivo.yaml`

2. **kubectl apply**:
   - Descrição: Aplica alterações em recursos Kubernetes usando arquivos de configuração YAML ou JSON. Ele pode criar, atualizar e deletar recursos.
   - Exemplo de uso: `kubectl apply -f arquivo.yaml`

3. **kubectl get**:
   - Descrição: Exibe informações sobre recursos Kubernetes, como pods, serviços, deployments, etc.
   - Exemplo de uso: `kubectl get pods`

4. **kubectl describe**:
   - Descrição: Mostra detalhes sobre um recurso específico, como sua configuração, eventos e status atual.
   - Exemplo de uso: `kubectl describe pod nome-do-pod`

5. **kubectl logs**:
   - Descrição: Exibe os logs de um contêiner em um pod.
   - Exemplo de uso: `kubectl logs nome-do-pod`

6. **kubectl exec**:
   - Descrição: Executa um comando em um contêiner em execução em um pod.
   - Exemplo de uso: `kubectl exec -it nome-do-pod -- comando`

7. **kubectl delete**:
   - Descrição: Deleta um ou mais recursos Kubernetes.
   - Exemplo de uso: `kubectl delete pod nome-do-pod`

8. **kubectl scale**:
   - Descrição: Ajusta o número de réplicas de um deployment ou replicaset.
   - Exemplo de uso: `kubectl scale deployment nome-do-deployment --replicas=3`

9. **kubectl rollout**:
   - Descrição: Gerencia o rollout de novas versões de um deployment.
   - Exemplo de uso: `kubectl rollout status deployment nome-do-deployment`

10. **kubectl port-forward**:
   - Descrição: Cria um túnel seguro entre uma porta local e um pod no cluster Kubernetes.
   - Exemplo de uso: `kubectl port-forward nome-do-pod 8080:80`


Após isso ja podemos realizar os testes e brincar com o ambiente.