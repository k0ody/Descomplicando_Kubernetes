## Day 6

Hoje, focaremos em descomplicar o entendimento sobre volumes no Kubernetes, abordando os conceitos de ConfigMap, PersistentVolume (PV) e PersistentVolumeClaim (PVC). Vamos utilizar exemplos práticos em diversos tipos de clusters Kubernetes para ilustrar esses conceitos e facilitar a compreensão.

Volumes são diretórios dentro de Pods que servem para armazenar dados. Eles podem ser utilizados para persistir informações importantes, como dados de bancos de dados ou sistemas de arquivos distribuídos. Existem dois tipos principais de volumes no Kubernetes: ephemeral volumes e persistent volumes.

**Ephemeral Volumes**

Os ephemeral volumes, como o EmptyDir, são criados e destruídos junto com o Pod. Eles não são persistentes, ou seja, se o Pod for removido, o volume também será. São úteis para compartilhar dados entre containers dentro de um Pod, mas não são recomendados para armazenamento de dados críticos.

**Persistent Volumes (PV)**

Os persistent volumes são volumes que persistem mesmo após a remoção do Pod. São essenciais para aplicações que precisam armazenar dados de forma durável, como bancos de dados. Os dados em um PV são mantidos mesmo quando o Pod é excluído.

**ConfigMap**

Um ConfigMap é um recurso no Kubernetes usado para armazenar dados de configuração em formato de chave-valor. Eles podem ser montados como volumes em Pods para fornecer configurações para os aplicativos em execução.

**Storage Class no Kubernetes: Uma Visão Abrangente**

Uma StorageClass no Kubernetes é um objeto que descreve e define diferentes classes de armazenamento disponíveis no cluster. Essas classes permitem provisionar dinamicamente PersistentVolumes (PVs) de acordo com os requisitos dos PersistentVolumeClaims (PVCs) dos usuários.

**Conceitos Funcamentais**

 - **StorageClass:** Define classes de armazenamento no cluster, com políticas específicas para provisionamento, retenção e outras características.
 - **Provisionador:** Responsável por criar PVs conforme necessário. Pode ser interno (fornecido pelo Kubernetes) ou externo (fornecido por provedores de armazenamento específicos).
 - **PV:** PersistentVolumes persistem mesmo após a remoção do Pod. Criados dinamicamente pela StorageClass.
 - **PVC:** PersistentVolumeClaims são solicitações feitas pelos usuários para o armazenamento. Usados para vincular aplicativos a PVs.

**Exempls de Provisionadores e Provedores**

 - **kubernetes.io/aws-ebs:** AWS Elastic Block Store (EBS)
 - **kubernetes.io/azure-disk:** Azure Disk
 - **kubernetes.io/gce-pd:** Google Compute Engine (GCE) Persistent Disk
 - **kubernetes.io/cinder:** OpenStack Cinder
 - **kubernetes.io/vsphere-volume:** vSphere
 - **kubernetes.io/no-provisioner:** Volumes locais
 - **kubernetes.io/host-path:** Volumes locais

**Exemplo Prático:**

Vamos criar uma nova StorageClass chamada "local-storage" no cluster Kubernetes usando Kind, com o provisionador "kubernetes.io/host-path".

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
provisioner: kubernetes.io/host-path
reclaimPolicy: Retain
volumeBindingMode: WaitForFirstConsumer
```

Executamos o comando:

```
kubectl apply -f storageclass.yaml
```

Agora, temos um novo StorageClass chamado "local-storage" criado. Podemos visualizar detalhes usando:

```
kubectl describe storageclass local-storage
```

**Observações**

 - No Kind, o provisionador padrão é "rancher.io/local-path".
 - No EKS, o provisionador padrão é "kubernetes.io/aws-ebs".
 - Cada StorageClass pode ter políticas e comportamentos específicos, como a política de retenção e o modo de vinculação de volume.
 - O StorageClass padrão é usado se um PVC não especificar um StorageClass.

Com uma compreensão completa dos conceitos e exemplos práticos de StorageClass no Kubernetes, os administradores podem efetivamente gerenciar e provisionar armazenamento de maneira eficiente para suas aplicações.


**Persistent Volume (PV) no Kubernetes: Explorando Opções de Armazenamento**

O PV é um recurso que representa um armazenamento físico no cluster Kubernetes, podendo ser um disco local em um nó, um dispositivo de armazenamento em rede (NAS) ou até mesmo um serviço de armazenamento em nuvem, como AWS EBS ou Google Cloud Persistent Disk. Ele oferece armazenamento durável, garantindo que os dados permaneçam disponíveis mesmo após reinicializações ou movimentações de contêineres.

**Tipos de Armazenamento:**

**Armazenamento em Rede:**
 - **NFS (Network File System):** Sistema de arquivos de rede que permite compartilhar arquivos entre várias máquinas na rede, comum para armazenamento compartilhado em clusters Kubernetes.
 - **iSCSI (Internet Small Computer System Interface):** Protocolo para conexão de dispositivos de armazenamento de blocos via redes IP.
 - **Ceph RBD (RADOS Block Device):** Solução de armazenamento distribuído e escalável que suporta armazenamento em bloco, objeto e arquivo.
 - **GlusterFS:** Sistema de arquivos distribuído e escalável para criar volumes compartilhados em vários nós do cluster.
 - **Serviços de Armazenamento em Nuvem:** Provedores de nuvem oferecem soluções integradas, como AWS EBS, Google Cloud Persistent Disk e Azure Disk Storage.

 **Gerenciamento de PVs com kubectl:**

 Primeiro, listamos os PVs no cluster:

```
kubectl get pv -A
```

Em seguida, criamos um novo PV. Exemplo utilizando HostPath:

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: meu-pv
  labels:
    storage: local
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: "/mnt/data"
  storageClassName: standard
  ```

Executando:

```
kubectl apply -f pv.yaml
```

Verificando se foi criado corretamente:

```
kubectl get pv
```

**Exempllo Prático - NFS**

Primeiro, configuramos o servidor NFS e compartilhamos um diretório:

```
mkdir /mnt/nfs
sudo apt-get install nfs-kernel-server nfs-common
sudo vi /etc/exports
sudo exportfs -arv
showmount -e
```

Criando um **StorageClass** para o provisionador NFS:

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs
provisioner: kubernetes.io/no-provisioner
reclaimPolicy: Retain
volumeBindingMode: WaitForFirstConsumer
parameters:
  archiveOnDelete: "false"
```

Criando **PV** associado ao **StorageClass** NFS:

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: meu-pv-nfs
  labels:
    storage: nfs
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  nfs:
    server: IP_DO_SERVIDOR_NFS
    path: "/mnt/nfs"
  storageClassName: nfs

```

Aplicando os arquivo:

```
kubectl apply -f storageclass-nfs.yaml
kubectl apply -f pv-nfs.yaml
```

Dessa forma, exploramos diversas opções de armazenamento com PVs no Kubernetes, permitindo escolher a mais adequada para cada aplicação.


**Persistent Volume Claim (PVC) no Kubernetes: Solicitando e Utilizando Armazenamento**

O PVC é uma solicitação de armazenamento feita no Kubernetes, permitindo que usuários e aplicativos solicitem um volume específico com base em requisitos como tamanho, tipo de armazenamento e modo de acesso. Ele age como uma "assinatura" que reivindica um PV para ser usado por um contêiner.

**Criando um PVC**

Criamos um arquivo chamado pvc.yaml com o seguinte conteúdo:

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: meu-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: nfs
  selector:
    matchLabels:
      storage: nfs
```

 - **accessModes:** Define o modo de acesso ao volume (ReadWriteOnce, ReadOnlyMany, ReadWriteMany).
 - **resources:** Define os recursos solicitados, neste caso, 1Gi de armazenamento.
 - **storageClassName:** Define a classe de armazenamento a ser utilizada.
 - **selector:** Define o seletor de labels para selecionar o PV.

Aplicando o arquivo:

```
kubectl apply -f pvc.yaml
```

Verificando se foi criado corretamente:

```
kubectl get pvc
```

**Associando o PVC a um Pod:**

Criando um Pod para usar o PVC. Criamos um arquivo pod.yaml com o seguinte conteúdo:

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
    volumeMounts:
    - name: meu-pvc
      mountPath: /usr/share/nginx/html
  volumes:
  - name: meu-pvc
    persistentVolumeClaim:
      claimName: meu-pvc
```

 - Definimos um volume chamado meu-pvc e montamos em /usr/share/nginx/html.
 - O volume é definido como um PVC chamado meu-pvc.

Aplicando o arquivo:

```
kubectl apply -f pod.yaml
```

Verificando se o Pod está em execução e se o PVC foi vinculado ao PV:

```
kubectl get pods
kubectl get pvc
```
**Testando o Volume:**

Criando um arquivo HTML simples no diretorio do servidor NFS:

```
echo "<h1>GIROPOPS STRIGUS GIRUS</h1>" > /mnt/data/index.html
```

Verificando se o arquivo foi criado dentro do Pod:

```
kubectl exec -it nginx-pod -- ls /usr/share/nginx/html
```

Testando se o Nginx está servindo o arquivo:

```
kubectl exec -it nginx-pod -- curl localhost
```

Criamos com sucesso um PVC, associamos a um Pod e verificamos se o volume estava funcionando corretamente. Isso demonstra como o Kubernetes facilita a gestão e utilização de armazenamento em ambientes de contêineres.



















































