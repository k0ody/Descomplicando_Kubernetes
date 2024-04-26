## Day 5

**O que é um cluster kubernetes**

Um cluster Kubernetes é uma infraestrutura de computação distribuída composta por nodes que trabalham juntos para orquestrar a execução de containers, organizados em pods. Ele consiste em um control plane, responsável por gerenciar e coordenar as operações do cluster, e workers, que executam os containers dos pods.

Os principais componentes do control plane incluem: 

**etcd** para armazenamento de configuração
**scheduler** para distribuir os pods nos workers
**api-server** para fornecer uma interface de comunicação e o controller-manager para monitorar e ajustar o estado do cluster.
**controller-manager** é responsável por gerenciar diversos controladores que supervisionam o estado do cluster e garantem seu funcionamento adequado. Ele realiza uma monitoração contínua dos recursos do cluster, comparando seu estado atual com o estado desejado e efetuando ajustes conforme necessário para manter a integridade e a estabilidade do ambiente. Em resumo, o controller-manager desempenha um papel crucial na manutenção da consistência e na execução eficiente das operações no cluster Kubernetes.

Nos workers:

**kubelet** supervisiona a execução dos containers
**kube-proxy** facilita a comunicação entre os pods e o exterior. 

Essencialmente, o cluster Kubernetes oferece uma plataforma escalável e automatizada para implantar, dimensionar e gerenciar aplicações em ambientes de contêineres.

**Formas de instalar o kubernetes**

Existem várias formas de instalar o Kubernetes, cada uma com suas próprias características e usos:

1. **kubeadm:** Ferramenta para criar e gerenciar clusters Kubernetes em vários nós. Automatiza tarefas de configuração e é altamente configurável.

2. **Kubespray:** Utiliza Ansible para implantar e gerenciar clusters Kubernetes em vários nós, oferecendo opções de personalização.

3. **Cloud Providers:** Provedores de nuvem como AWS, Google Cloud Platform e Microsoft Azure oferecem opções para implantar clusters Kubernetes em sua infraestrutura, incluindo modelos predefinidos e serviços gerenciados.

4. **Kubernetes Gerenciados:** Serviços como Amazon EKS, Google Kubernetes Engine (GKE) e Azure Kubernetes Service (AKS) oferecem clusters Kubernetes gerenciados, onde o provedor cuida da configuração, atualização e manutenção do cluster.

5. **Kops:** Ferramenta para implantar e gerenciar clusters Kubernetes em nuvens públicas, oferecendo personalização, escalabilidade e segurança, mas pode ser mais complexa de usar.

6. **Minikube e kind:** Ferramentas para criar clusters Kubernetes localmente em um único nó, úteis para testes, aprendizado e desenvolvimento de aplicações.

Cada opção atende a diferentes necessidades e contextos, desde ambientes de produção até desenvolvimento e aprendizado.


**Criando um cluster com kubeadm**

Antes de iniciarmos, vamos precisar de 3 maquinas, no nosso exemplo iremos utilizar a AWS, dessa forma precisamos de 3 maquinas EC2 com os seguintes requisitos:

## Pré-requisitos para instalação do Kubernetes com kubeadm:

- **Sistema Operacional:** Linux
- **Recursos mínimos por máquina:**
  - 2 GB ou mais de RAM (recomendado)
  - 2 CPUs ou mais
- **Conexão de rede:**
  - Conexão de rede entre todas as máquinas no cluster, podendo ser via rede pública ou privada
- **Portas necessárias abertas:**
  - Porta TCP 6443: Usada pelo Kubernetes API Server para comunicação com os componentes do cluster
  - Portas TCP 10250-10255: Usadas pelo kubelet para comunicação com o control plane do Kubernetes
  - Portas TCP 30000-32767: Usadas para serviços NodePort acessíveis fora do cluster
  - Portas TCP 2379-2380: Usadas pelo etcd, o banco de dados distribuído usado pelo control plane do Kubernetes
  - Porta TCP 6783: usada pelo Weave Net para funcionamento correto da comunicação entre o control plane e os workes (CNI)
  - Portas UDP 6783-6784: usada pelo Weave Net para funcionamento correto da comunicação entre o control plane e os workes (CNI)

O CNI (Container Network Interface) no Kubernetes é uma especificação padrão que define como redes de contêineres devem ser configuradas e gerenciadas. Ele permite que diferentes soluções de rede sejam integradas ao Kubernetes de forma consistente, facilitando a comunicação entre os pods e a conectividade de rede dentro do cluster. O CNI oferece flexibilidade para escolher a solução de rede mais adequada às necessidades específicas do ambiente, seja ela baseada em virtualização, roteamento de pacotes, overlay, entre outras. Em suma, o CNI é essencial para o funcionamento da rede no Kubernetes, fornecendo uma interface padronizada para configurar e gerenciar redes de contêineres.

Para realizar a liberação das portas na AWS precisa editar as configurações do security group das instancias.

**Instalando o kubeadm**

Após iniciar as instancias, precisamos realizar algumas configurações, segue abaixo:

**1.Desativando o usod do swap**

O kubernetes não trabalha bem com swap ativado

```
sudo swapoff -a
```

**2. Carregando os modulos do kernel necessários**

Módulos do kernel necessários para o funcionamento do Kubernetes

```
sudo sh -c "cat <<EOF > /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF"

sudo modprobe overlay
sudo modprobe br_netfilter
```

**3.Configurando parametros do sistema**

Vamos configurar alguns parâmetros do sistema. Isso garantirá que nosso cluster funcione corretamente

```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
```

**4. Instalando os pacotes do kubernetes**

instalar os pacotes do Kubernetes

```
sudo apt-get update && sudo apt-get install -y apt-transport-https ca-certificates curl gpg

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update && sudo apt-get install -y kubelet kubeadm kubectl

sudo apt-mark hold kubelet kubeadm kubectl

```

Nota: O comando sudo apt-mark hold kubelet kubeadm kubectl é usado para "marcar" os pacotes do Kubernetes (kubelet, kubeadm e kubectl) como "retidos" no sistema de gerenciamento de pacotes apt. Quando um pacote é marcado como retido, o sistema de gerenciamento de pacotes não o atualizará automaticamente para uma versão mais recente quando novas atualizações estiverem disponíveis.


**Instalando o containerd**

o containerd é um componente essencial para o ecossistema de contêineres, fornecendo uma base sólida e confiável para a execução de contêineres em ambientes de produção. Ele desempenha um papel crucial no suporte à criação e execução de contêineres de forma eficiente e confiável.

```
sudo apt-get update && sudo apt-get install -y apt-transport-https ca-certificates curl gnupg lsb-release

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update && sudo apt-get install -y containerd.io

```

**Configurando o containerd**

```
sudo containerd config default | sudo tee /etc/containerd/config.toml

sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml

sudo systemctl restart containerd
sudo systemctl status containerd
```


**Habilitando o servilo do kubelet**

```
sudo systemctl enable --now kubelet
```

**Iniciando o cluster**

Das 3 maquinas, escolha uma para ser o control plane, e nela iniremos iniciar o cluster com o seguinte comando:

```
sudo kubeadm init --pod-network-cidr=10.10.0.0/16 --apiserver-advertise-address=<IP do control plane>

```

- **sudo kubeadm init**: Inicia o processo de inicialização do cluster Kubernetes.
- **--pod-network-cidr=10.10.0.0/16**: Especifica o intervalo de endereços IP que serão atribuídos aos pods no cluster. Nesse exemplo, estamos usando o intervalo 10.10.0.0/16.
- **--apiserver-advertise-address=**<IP do control plane>: Define o endereço IP pelo qual o API Server do Kubernetes será anunciado para os nodes. Substitua <IP do control plane> pelo endereço IP da máquina que será o control plane.


Após a execução bem-sucedida do comando, o cluster Kubernetes será inicializado e você verá uma mensagem informando sobre o sucesso da inicialização, juntamente com detalhes adicionais sobre a configuração do cluster.

Copie e cole os comandos no seu terminal

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Aqui está um resumo sobre o arquivo admin.conf no contexto do Kubernetes:

**Finalidade:** O arquivo admin.conf é uma configuração do cliente kubectl, usado para se comunicar com o cluster Kubernetes.
**Informações de Acesso:** Ele contém informações essenciais de acesso ao cluster, como endereço do servidor API, certificado de cliente e token de autenticação.
**Múltiplos Contextos:** Pode conter vários contextos, cada um representando um cluster Kubernetes. Isso permite alternar facilmente entre diferentes ambientes, como produção e desenvolvimento.
**Segurança:** Como o arquivo contém dados de acesso ao cluster, deve ser protegido adequadamente. Se alguém tiver acesso a esse arquivo, terá acesso ao cluster.
**Criação:** O admin.conf é gerado durante a inicialização do cluster Kubernetes.

Este arquivo é fundamental para configurar o acesso ao cluster Kubernetes usando o kubectl e deve ser tratado com segurança para evitar acesso não autorizado ao ambiente do cluster.

Outras informaçoes continda no arquivo


- **Token de Autenticação:** Um token de acesso gerado automaticamente durante a inicialização do cluster, usado para autenticar o usuário ao executar comandos kubectl.

- **certificate-authority-data:** Contém a representação em base64 do certificado da autoridade de certificação (CA) do cluster. Este certificado é usado para verificar a autenticidade dos certificados apresentados pelo servidor de API e pelos clientes, garantindo a segurança e confiabilidade da comunicação.

- **client-certificate-data:** Contém a representação em base64 do certificado do cliente, assinado pela CA do cluster. Este certificado é usado para autenticar o usuário ao se comunicar com o servidor de API do Kubernetes.

- **client-key-data:** Contém a representação em base64 da chave privada do cliente, usada para assinar as solicitações enviadas ao servidor de API do Kubernetes. Garante que o servidor possa verificar a autenticidade da solicitação.

Esses campos são essenciais para estabelecer uma comunicação segura e autenticada entre o cliente (geralmente o kubectl) e o servidor de API do Kubernetes. Eles garantem que apenas usuários e sistemas autorizados possam acessar e gerenciar os recursos do cluster. Os arquivos que contêm essas credenciais são geralmente encontrados em `/etc/kubernetes/pki/` e são automaticamente adicionados ao arquivo admin.conf durante a inicialização do cluster.

**Adicioando os demais nodes ao cluster**

Antes de adiconar mais nodes, precisamos do parametro de comunicação, no control plane rode o seguinte comando:

```
kubeadm token create --print-join-command
```

A saida será parecida com isso:

```
sudo kubeadm join 172.31.57.89:6443 --token if9hn9.xhxo6s89byj9rsmd \
    --discovery-token-ca-cert-hash sha256:ad583497a4171d1fc7d21e2ca2ea7b32bdc8450a1a4ca4cfa2022748a99fa477

```

Este comando deve ser executado nos worker nodes que desejam se juntar ao cluster Kubernetes. Após a execução bem-sucedida do comando, o novo node será autenticado e adicionado ao cluster, começando a receber instruções do control plane e a executar Pods conforme necessário.

Onde:

**kubeadm join:** Comando utilizado para adicionar um novo nó (worker) ao cluster Kubernetes existente.
**172.31.57.89:6443:** Endereço IP e porta do servidor de API do nó mestre (control plane) ao qual o nó worker será adicionado.
**--token if9hn9.xhxo6s89byj9rsmd:** Token de autenticação gerado pelo nó mestre, usado para autenticar o nó worker durante o processo de adesão.
**--discovery-token-ca-cert-hash sha256:ad583497a4171d1fc7d21e2ca2ea7b32bdc8450a1a4ca4cfa2022748a99fa477:** Hash criptográfico do certificado da autoridade de certificação (CA) do control plane. Usado para garantir a autenticidade do nó worker e do control plane durante o processo de adesão.

**Instalando Waeve Net**

Weave Net é uma solução de rede poderosa e flexível para Kubernetes, que fornece conectividade confiável e segura entre os pods em um cluster, independentemente da infraestrutura subjacente. Ele desempenha um papel crucial na facilitação da comunicação entre os componentes de aplicativos distribuídos executados em contêineres Kubernetes.

No control plane rode o seguinte comando:

```
$ kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```

Aguarde alguns minutos até que todos os componentes do cluster estejam em funcionamento. Você pode verificar o status dos componentes do cluster com o seguinte comando:

```
kubectl get pods -n kube-system
```

Apoś todos os componentes estivem up, voce pode verificar os nodes, a saída será parecida com essa:

```
NAME     STATUS   ROLES           AGE   VERSION
k8s-01   Ready    control-plane   7m   v1.26.3
k8s-02   Ready    <none>          6m   v1.26.3
k8s-03   Ready    <none>          6m   v1.26.3
```

Onde o status estará Ready em todos os nodes.

## Resumo

Esse foi apenas um breve resumo de como realizar a instalação do kubernetes utilizando o kubeadm, existe outras formas e maneiras de ter um cluester kubernetes.






