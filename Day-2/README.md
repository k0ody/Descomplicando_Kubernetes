# Day 2


## Pod


Um Pod é a menor unidade escalável no Kubernetes, uma plataforma de orquestração de contêineres. Ele representa um grupo de um ou mais contêineres que compartilham armazenamento em disco, rede e outras características do ambiente, como endereço IP e namespace de processos.

Os Pods são a unidade básica de implantação e execução de contêineres no Kubernetes. Eles encapsulam um ou mais contêineres, normalmente aqueles que estão intimamente relacionados e precisam ser co-localizados e coordenados. 

Por exemplo, um Pod pode conter um contêiner principal, responsável pela aplicação em si, e contêineres auxiliares, como sidecars (contêineres que fornecem funcionalidades adicionais, como logging ou monitoramento) ou contêineres de iniciadores.

Os Pods são efêmeros por natureza, o que significa que podem ser criados, escalados, replicados, movidos e excluídos conforme necessário pelo Kubernetes. Eles também fornecem isolamento de recursos, como CPU e memória, garantindo que os contêineres dentro de um Pod tenham recursos compartilhados ou isolados conforme configurado.

Em resumo, um Pod no Kubernetes é uma abstração que define um ambiente de execução para um ou mais contêineres relacionados, facilitando sua implantação, gerenciamento e escalabilidade em um cluster de contêineres.

**Criando um Pod**

Para criar um pod temos duas maneiras de criar um pod, a primeira é por linha de comando.

Resumindo, para criar e gerenciar Pods no Kubernetes via linha de comando, você pode usar uma variedade de comandos kubectl. Aqui está um resumo dos principais comandos e conceitos:

**Criar um Pod com um comando simples:**
Use kubectl run para criar um Pod com uma imagem específica e porta exposta:

```
kubectl run giropops --image=nginx --port=80
```

**Verificar os Pods criados:**
Use kubectl get pods para listar todos os Pods na namespace atual:

```
kubectl get pods
```

**Visualizar os detalhes de um Pod:**
Use kubectl describe pods seguido do nome do Pod para obter detalhes adicionais:

```
kubectl describe pods giropops

```

**Excluir um Pod:**
Use kubectl delete pods seguido do nome do Pod para removê-lo:

```
kubectl delete pods giropops
```

**Explorar outras namespaces:**
Use kubectl get pods -n <namespace> para ver os Pods em uma namespace específica:

```
kubectl get pods -n kube-system
```

**Exibir detalhes em diferentes formatos:**
Use a opção -o com json, yaml, ou wide para formatar a saída conforme desejado:

```
kubectl get pods giropops -o yaml
kubectl get pods giropops -o json
kubectl get pods giropops -o wide
```

**Criando Pod atras de um arquivo YAML**

Abaixo o arquivo yaml chamado pod.yaml com a descrição de cada linha:

```
# versão da API do Kubernetes
apiVersion: v1
# tipo do objeto que estamos criando
kind: Pod
# metadados do Pod
metadata:
  # nome do Pod que estamos criando
  name: giropops
  # labels do Pod
  labels:
    # label run com o valor giropops
    run: giropops
# especificação do Pod
spec:
  # containers que estão dentro do Pod
  containers:
  # nome do container
  - name: giropops
    # imagem do container
    image: nginx
    # portas que estão sendo expostas pelo container
    ports:
    # porta 80 exposta pelo container
    - containerPort: 80
```
**Aplicar o arquivo pod.yaml usaro o `kubectl apply`:**

```
kubectl apply -f pod.yaml
```
Isso irá criar o pod usando as especificaçãoes fornecidas no arquivo.

**Verificar se o Pod foi criado corretamente:**
```
kubectl get pods
```

Este comando listará todos os Pods e você deverá ver o Pod "giropops" na lista.


O comando kubectl apply é preferível ao kubectl create porque pode ser usado para criar objetos ou atualizar objetos existentes, enquanto o kubectl create pode retornar um erro se o objeto já existir. O kubectl describe fornece detalhes adicionais sobre o estado atual do Pod, incluindo eventos associados, e o kubectl logs permite visualizar os logs do container dentro do Pod. O kubectl delete é usado para remover objetos do cluster Kubernetes.


**Comando Attach e Exec**


**Comando `kubectl attach`:**
- O comando `kubectl attach` é usado para conectar o terminal local ao terminal de um contêiner em execução dentro de um Pod.
- Ele fornece uma interface interativa para interagir diretamente com um contêiner em execução, permitindo a visualização e a entrada de comandos.
- O comando `attach` é útil para depuração, monitoramento ou interações em tempo real com um contêiner.
- Uma vez conectado ao terminal do contêiner, você pode usar o terminal local para enviar comandos diretamente para o contêiner.

Exemplo:

```
kubectl attach nome-do-pod -c nome-do-container
```

**Comando `kubectl exec`:**
- O comando `kubectl exec` é usado para executar comandos dentro de um contêiner em execução dentro de um Pod.
- Ele permite executar comandos específicos em um contêiner, sem a necessidade de uma conexão interativa.
- É útil para executar comandos de administração, diagnóstico ou manutenção dentro do ambiente do contêiner.
- Você pode executar um único comando ou abrir uma sessão interativa para executar vários comandos no contêiner.

Exemplo:

```
kubectl exec -it nome-do-pod -- comando-a-executar

```


**Limitando recusros no Pod**

Segue abaixo um exemplo de como limitar recursos no pod:

```
apiVersion: v1
kind: Pod
metadata:
  name: giropops
  labels:
    run: giropops
spec:
  containers:
  - name: girus
    image: nginx
    ports:
    - containerPort: 80
    resources:
      limits:
        memory: "128Mi"
        cpu: "0.5"
      requests:
        memory: "64Mi"
        cpu: "0.3"

```

- **`resources:`**: Este campo especifica os recursos que estão sendo utilizados pelo contêiner dentro do Pod.

- **`limits:`**: Este subcampo define os limites máximos de recursos que o contêiner pode utilizar. No caso:
  - **`memory: "128Mi"`**: Define o limite máximo de memória para o contêiner como 128 mebibytes (128MiB). Isso significa que o contêiner não poderá consumir mais do que 128MiB de memória.
  - **`cpu: "0.5"`**: Define o limite máximo de CPU para o contêiner como 0.5, o que equivale a 50% de uma CPU. Isso significa que o contêiner não poderá consumir mais do que 50% da capacidade de uma CPU.

- **`requests:`**: Este subcampo define os recursos garantidos ao contêiner, que são reservados para ele independentemente da carga no nó. No caso:
  - **`memory: "64Mi"`**: Define a quantidade de memória garantida ao contêiner como 64 mebibytes (64MiB). Isso significa que o contêiner sempre terá pelo menos 64MiB de memória disponível.
  - **`cpu: "0.3"`**: Define a quantidade de CPU garantida ao contêiner como 0.3, o que equivale a 30% de uma CPU. Isso significa que o contêiner sempre terá pelo menos 30% da capacidade de uma CPU disponível.

Esses campos são úteis para controlar e gerenciar o consumo de recursos pelos contêineres em um cluster Kubernetes, garantindo que eles não consumam mais recursos do que o necessário e evitando problemas de sobrecarga.


## Adicionando um volume EmptyDir no Pod

Um volume do tipo `emptyDir` é um tipo de volume em ambientes de orquestração de contêineres, como Kubernetes, que é criado vazio quando um pod é iniciado e existe enquanto o pod estiver em execução. Ele fornece um espaço de armazenamento temporário que pode ser compartilhado entre os contêineres em um pod. Os dados armazenados em um `emptyDir` existem apenas durante o ciclo de vida do pod e são perdidos quando o pod é excluído ou reiniciado. Esse tipo de volume é útil para compartilhar arquivos temporários ou dados efêmeros entre os contêineres de um mesmo pod.

Abaixo um arquivo YAML para exemplo:

```
# Api que esta rodando, v1 mais estavel
apiVersion: v1
# O que quero rodar
kind: Pod
# Informações relacionadas ao pod
metadata:
  # São pares de chave-valor que podem ser usados para identificar e selecionar objetos Kubernetes
  labels:
    run: giropops
  # nome do pod
  name: emptydir
# Especificações do pod
spec:
  containers:
  # Imagem
  - image: nginx
    # Nome do container
    name: webserver
    # Lista de pontos de montagem para volumes nos contêineres.
    volumeMounts:
    # Ponto de montagem dentro do contêiner.
    - mountPath: /giropops
      # Nome do volume a ser montado.
      name: primeiro-emptydir
    # Limitação de recursos
    resources:
      # Limites máximos de recursos que o container pode consumir.
      limits:
        cpu: "1"
        memory: "128Mi"
      # Recursos mínimos que o container precisa para ser executado corretamente
      requests:
        cpu: "0.5"
        memory: "64Mi"
    
  # Define a política DNS para o Pod. "ClusterFirst" indica que o DNS do cluster deve ser consultado primeiro
  dnsPolicy: ClusterFirst
  # Define a política de reinicialização do Pod. "Always" indica que o Pod deve ser reiniciado sempre que falhar
  restartPolicy: Always
  # Lista de volumes disponíveis para montagem nos contêineres.
  volumes:
  # Nome do volume que será criado e montado.
  - name: primeiro-emptydir
    # Tipo de volume: diretório vazio.
    emptyDir:
      # Limite de tamanho do diretório vazio (256 megabytes).
      sizeLimit: 256Mi
```

**volumeMounts:** Define os pontos de montagem dentro do contêiner.

**mountPath:** /giropops
 Especifica o caminho de montagem dentro do contêiner.
 Este ponto de montagem será onde o volume será montado dentro do contêiner.
 Neste caso, o volume será montado em /giropops dentro do contêiner.
 Todos os arquivos e diretórios dentro deste caminho no contêiner estarão vinculados ao volume.
 Qualquer conteúdo pré-existente neste caminho dentro do contêiner será ocultado pelo conteúdo do volume.
 É possível definir vários pontos de montagem se necessário.
 Cada ponto de montagem é especificado como um item de uma lista.

**name:** primeiro-emptydir
 Nome do volume a ser montado.
 Este é o nome do volume definido no campo 'volumes'.
 Este volume será montado no ponto de montagem especificado no contêiner.

**volumes:**
 Define os volumes disponíveis para o Pod.
 Cada volume pode ser montado em um ou mais contêineres no Pod.
 Neste caso, é definido um único volume chamado 'primeiro-emptydir'.

**name:** primeiro-emptydir
 Nome do volume que será criado e montado.
 Este nome será usado para associar o volume aos pontos de montagem nos contêineres.

**emptyDir:**
 Define um tipo de volume 'emptyDir', que é um diretório vazio.
 Este tipo de volume é criado quando o Pod é iniciado e é efêmero, o que significa que
 qualquer conteúdo armazenado dentro dele será perdido quando o Pod for excluído ou reiniciado.

**sizeLimit:** 256Mi
 Opcional. Define um limite de tamanho para o diretório vazio.
 Neste caso, o diretório vazio criado pelo volume 'primeiro-emptydir' terá um limite máximo de tamanho de 256 megabytes.
