## Day 7

Hoje vamos falar sobre dois recursos muito importantes para o Kubernetes: StatefulSet e Service.

## StatefulSet

- O StatefulSet é utilizado para criar e administrar aplicações que precisam manter a identidade do Pod e persistir dados em volumes locais.
- Exemplos comuns de uso incluem bancos de dados, sistemas de mensageria e sistemas de arquivos distribuídos.
- O StatefulSet garante que cada Pod mantenha uma identidade única e persistente, facilitando a gestão de aplicações com estado.

## Service

- O Service permite expor uma aplicação para o mundo externo.
- Ele faz o balanceamento de carga entre os Pods expostos e resolve nomes DNS para esses Pods.
- Existem diversos tipos de Services, cada um com funcionalidades específicas que serão discutidas.

Prepare-se para uma jornada interessante sobre esses dois recursos essenciais para o Kubernetes e para o cotidiano de quem trabalha com a plataforma.


## StatefulSet no Kubernetes

### O que é um StatefulSet?
StatefulSets são uma funcionalidade do Kubernetes que gerencia o deployment e o scaling de um conjunto de Pods, garantindo a ordem de deployment e a singularidade desses Pods. Diferente dos Deployments e Replicasets (que são stateless), os StatefulSets são utilizados quando você precisa de mais garantias sobre o deployment e scaling, garantindo nomes e endereços dos Pods consistentes e estáveis.

### Quando usar StatefulSets?
StatefulSets são úteis para aplicações que necessitam de:
- Identidade de rede estável e única.
- Armazenamento persistente estável.
- Ordem garantida de deployment e scaling.
- Ordem garantida de rolling updates e rollbacks.

Aplicações típicas incluem bancos de dados, sistemas de filas e qualquer aplicação que necessite de persistência de dados ou identidade de rede estável.

### Como ele funciona?
StatefulSets criam uma série de Pods replicados, cada réplica sendo uma instância da mesma aplicação, diferenciada por seu índice e hostname. Cada Pod tem um índice persistente e um hostname que define sua identidade. Por exemplo, um StatefulSet com três réplicas cria Pods com nomes como `giropops-0`, `giropops-1`, `giropops-2`, garantindo a ordem de criação e update.

### StatefulSet e Volumes Persistentes
StatefulSets integram-se com Volumes Persistentes. Quando um Pod é recriado, ele se reconecta ao mesmo Volume Persistente, garantindo a persistência dos dados. O Kubernetes cria um PersistentVolume para cada Pod em um StatefulSet, vinculado a esse Pod enquanto ele existir.

### StatefulSet e Headless Service
Um Headless Service é um tipo especial de serviço no Kubernetes que não tem um IP próprio e retorna diretamente os IPs dos Pods associados a ele. StatefulSets e Headless Services trabalham juntos para gerenciar aplicações stateful. O Headless Service permite a comunicação de rede entre os Pods de um StatefulSet, atribuindo a cada Pod um hostname DNS único e estável.

Exemplo:
- StatefulSet: `giropops` com três réplicas.
- Headless Service: `nginx`.
- Pods: `giropops-0`, `giropops-1`, `giropops-2`.
- Hostnames DNS: `giropops-0.nginx.default.svc.cluster.local`, `giropops-1.nginx.default.svc.cluster.local`, `giropops-2.nginx.default.svc.cluster.local`.

Essa combinação facilita a comunicação entre diferentes instâncias de uma aplicação stateful.

Agora que você entende o funcionamento do StatefulSet, podemos dar os primeiros passos com essa funcionalidade.


## Criando um StatefulSet

Para criar um StatefulSet no Kubernetes, é necessário um arquivo de configuração YAML que descreva o StatefulSet. Não é possível criar um StatefulSet sem um manifesto YAML, diferente do que acontece com o Pod e o Deployment.

Para o nosso exemplo, vamos utilizar o Nginx como aplicação gerenciada pelo StatefulSet, associando cada Pod a um volume persistente, resultando em uma página web diferente para cada Pod.

### Exemplo de Configuração (nginx-statefulset.yaml)

```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx
spec:
  serviceName: "nginx"
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

Para criar o StatefulSet:

```
kubectl apply -f nginx-statefulset.yaml
```

Verificar a criação do StatefulSet:

```
kubectl get statefulset
```

Ver mais detalhes sobre o StatefulSet:

```
kubectl describe statefulset nginx
```


## Criando o Headless Service

Nosso StatefulSet está criado, mas ainda precisamos criar o Headless Service para acessar os Pods individualmente. Para isso, vamos criar o arquivo `nginx-headless-service.yaml` com o seguinte conteúdo:

```
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None # Como estamos criando um Headless Service, não queremos que ele tenha um IP, então definimos o clusterIP como None
  selector:
    app: nginx
```

Para criar o Headless Service, utilize o comando:
```
kubectl apply -f nginx-headless-service.yaml
```

Para verificar se o Headless Service foi criado, utilize o comando:
```
kubectl get service
```

Para ver mais detalhes do Headless Service, utilize o comando:
```
kubectl describe service nginx
```

## Excluindo um StatefulSet

```
kubectl delete statefulset nginx
```

Ou

```
kubectl delete -f nginx-statefulset.yaml
```

## Excluindo um Headless Service

Para excluir um Headless Service, utilize o comando:

```
kubectl delete service nginx
```

ou

```
kubectl delete -f nginx-headless-service.yaml
```

## Excluindo um PVC

Para excluir um Volume Persistente, utilize o comando:

```
kubectl delete pvc www-0
```



## O que são Services?

Os Services no Kubernetes são uma abstração que define um conjunto lógico de Pods e uma política para acessá-los. Eles permitem que você exponha uma ou mais Pods para serem acessados por outros Pods, independentemente de onde estejam em execução no cluster. Services são definidos usando a API do Kubernetes, normalmente através de um arquivo de manifesto YAML.

## Tipos de Services

Existem quatro tipos principais de Services:

1. **ClusterIP (padrão)**: Expõe o Service em um IP interno no cluster, tornando-o acessível apenas dentro do cluster.
2. **NodePort**: Expõe o Service na mesma porta de cada Node selecionado no cluster usando NAT, tornando-o acessível de fora do cluster usando `<NodeIP>:<NodePort>`.
3. **LoadBalancer**: Cria um balanceador de carga externo no ambiente de nuvem atual (se suportado) e atribui um IP fixo e externo ao cluster, ao Service.
4. **ExternalName**: Mapeia o Service para o conteúdo do campo `externalName` (por exemplo, foo.bar.example.com), retornando um registro CNAME com seu valor.

## Funcionamento dos Services

Services no Kubernetes fornecem uma abstração que define um conjunto lógico de Pods e uma política para acessá-los, usando seletores de rótulo (Label Selectors). Embora cada Pod tenha um endereço IP único, esses IPs não são expostos fora do cluster sem um Service.

### Importância dos Labels

Labels são fundamentais para a maioria das operações no Kubernetes, incluindo a definição de Services. Cuidar bem dos Labels é essencial para garantir que os Services funcionem corretamente.

## Services e Endpoints

Services representam um conjunto estável de Pods que fornecem determinada funcionalidade, mantendo um endereço IP estável e uma porta de serviço constantes, mesmo que os Pods subjacentes sejam substituídos. Para implementar essa abstração, o Kubernetes usa uma outra abstração chamada Endpoint.

Quando um Service é criado, o Kubernetes também cria um objeto Endpoint com o mesmo nome. Esse objeto rastreia os IPs e as portas dos Pods que correspondem aos critérios de seleção do Service.

### Verificando Endpoints

Para ver os Endpoints criados por um Service, use o comando:

```
kubectl get endpoints meu-service
```
Os Endpoints representam os Pods que estão sendo expostos pelo Service e são fundamentais para a comunicação e estabilidade dos Services no Kubernetes.



## Criando um Service

Para criar um Service, utilize o comando:

```
kubectl expose deployment MEU_DEPLOYMENT --port=80 --type=NodePort
```
Substitua MEU_DEPLOYMENT pelo nome do deployment que você deseja expor. Este comando cria um Service do tipo NodePort, tornando-o acessível de fora do cluster.


## Tipos de Services

Existem quatro tipos principais de Services:

- **ClusterIP:** Acessível apenas dentro do cluster.
- **NodePort:** Acesso externo usando a porta do nó.
- **LoadBalancer:** Cria um balanceador de carga externo.
- **ExternalName:** Mapeia um nome externo para o Service.

## ClusterIP

O Service ClusterIP é o tipo padrão de Service no Kubernetes. Ele expõe o Service em um IP interno no cluster, tornando-o acessível apenas dentro do cluster. Este tipo de Service é frequentemente usado para comunicação interna entre os componentes do aplicativo dentro do cluster.

## Características do Service ClusterIP

- **Acessibilidade Interna:** O Service ClusterIP só é acessível dentro do cluster Kubernetes. Ele não é roteado para fora do cluster.
- **Balanceamento de Carga Interno:** Ele distribui o tráfego entre os Pods correspondentes usando balanceamento de carga interno.
- **Endereço IP Estável:** O Service tem um IP interno estável que é mantido mesmo que os Pods subjacentes mudem.
- **Comunicação Interna:** É comumente usado para comunicação entre os componentes do aplicativo dentro do cluster.

## Casos de Uso Comuns

- **Comunicação entre Serviços:** Permite que diferentes partes de um aplicativo se comuniquem internamente no cluster.
- **Serviços Internos:** Útil para expor serviços internos que não precisam ser acessados fora do cluster.
- **Microserviços:** Facilita a comunicação entre microserviços dentro de uma arquitetura distribuída.

O Service ClusterIP é uma ferramenta fundamental para construir arquiteturas de aplicativos escaláveis e confiáveis no Kubernetes. Ele fornece uma maneira consistente e confiável de expor e acessar os serviços dentro do cluster.

## Criando ClusterIP

Criando um Service ClusterIP via kubectl:

```
kubectl expose deployment meu-deployment --port=80 --target-port=8080
```

Ou via YAML:

```
apiVersion: v1
kind: Service
metadata:
  name: meu-service
spec:
  selector:
    app: meu-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```


## NodePort

Um NodePort é um tipo de serviço no Kubernetes que expõe um serviço para fora do cluster, tornando-o acessível através de um número de porta específico em cada nó do cluster. Ele permite o acesso externo aos serviços executados dentro do cluster, encaminhando o tráfego para os pods correspondentes. É útil para casos em que você precisa acessar um serviço Kubernetes de fora do cluster, como uma aplicação da web, APIs ou serviços RESTful.

## Características do Service NodePort

- **Acessibilidade Externa e Interna:** O Service NodePort é acessível tanto dentro quanto fora do cluster Kubernetes. Ele expõe o serviço em uma porta específica em cada nó do cluster, permitindo o acesso externo via endereço IP do nó e porta designada.

- **Não é Roteado para Fora do Cluster:** Embora seja acessível externamente, o tráfego do Service NodePort não é roteado diretamente para fora do cluster. Ele é roteado através dos nós do cluster usando NAT (Network Address Translation).

- **Balanceamento de Carga Simples:** O Service NodePort não fornece balanceamento de carga automático. O tráfego externo é distribuído para os Pods correspondentes de forma round-robin pelos nós do cluster.

- **Porta Designada em Cada Nó:** Cada nó do cluster escuta na mesma porta designada para o Service NodePort. Quando o tráfego é recebido nessa porta em um nó, ele é encaminhado para o Pod correspondente.

## Casos de Uso Comuns

- **Acesso Externo a Aplicativos:** É útil quando você precisa expor um serviço para acesso externo, como um aplicativo da web, APIs ou serviços RESTful.

- **Testes e Desenvolvimento:** Facilita o acesso a aplicativos em desenvolvimento ou em teste diretamente a partir de um navegador da web ou de outras ferramentas de cliente.

- **Integração com Serviços Externos:** Pode ser usado para integrar aplicativos em execução no Kubernetes com serviços externos ou sistemas legados.

## Limitações e Considerações

- **Segurança:** Como o tráfego externo é roteado diretamente para os nós do cluster, é importante configurar as regras de firewall e segurança corretas para proteger os nós e os serviços expostos.

- **Portas Designadas:** As portas designadas no intervalo de 30000-32767 podem entrar em conflito com outras aplicações ou serviços em execução nos nós do cluster. É importante garantir que não haja conflitos de portas.

- **Balanceamento de Carga:** O Service NodePort não oferece recursos avançados de balanceamento de carga. Se for necessário um balanceamento de carga mais sofisticado, pode ser necessário utilizar um Service LoadBalancer ou uma solução de balanceamento de carga externa.

## Criando um NodePort

```
Criando um Service NodePort via kubectl:
```

Ou via YAML:

```
apiVersion: v1
kind: Service
metadata:
  name: meu-service
spec:
  type: NodePort
  selector:
    app: meu-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
      nodePort: 30080
```

## LoadBalancer

Um Service LoadBalancer no Kubernetes é um tipo de serviço que expõe um aplicativo para o tráfego externo da Internet. Ele provisiona automaticamente um balanceador de carga externo no ambiente de nuvem onde o cluster Kubernetes está hospedado, se suportado pelo provedor de nuvem. Isso permite que o aplicativo seja acessado por usuários externos através de um IP fixo externo, fornecendo escalabilidade e alta disponibilidade.

## Características do Service LoadBalancer

- **Acessibilidade Externa:** O Service LoadBalancer é uma das formas mais comuns de expor um serviço ao tráfego da internet no Kubernetes. Ele provisiona automaticamente um balanceador de carga do provedor de nuvem onde seu cluster Kubernetes está rodando, se houver algum disponível.

- **IP Externo e Acesso pela Internet:** Ele fornece um IP acessível externamente que encaminha o tráfego para os pods do serviço, permitindo que o serviço seja acessado pela internet.

- **Balanceamento de Carga Automático:** O balanceador de carga provisionado pelo Service LoadBalancer distribui automaticamente o tráfego entre os pods correspondentes, proporcionando um balanceamento de carga eficaz.

- **Configuração Simples:** A criação de um Service LoadBalancer é relativamente simples, pois o Kubernetes lida com a configuração do balanceador de carga e a atribuição do IP externo automaticamente.

- **Uso com Provedores de Nuvem:** É especialmente útil quando seu cluster Kubernetes está hospedado em um provedor de nuvem que suporta balanceadores de carga, como AWS, GCP, Azure, etc.

## Casos de Uso Comuns

- **Serviços Acessíveis pela Internet:** É útil quando você precisa que seu serviço seja acessível pela internet, como um aplicativo da web, APIs públicas, ou serviços expostos para clientes externos.

- **Alta Disponibilidade:** O Service LoadBalancer é uma escolha comum para garantir alta disponibilidade e escalabilidade para serviços críticos que precisam lidar com um grande volume de tráfego.

- **Balanceamento de Carga Avançado:** É apropriado quando você precisa de recursos avançados de balanceamento de carga, como monitoramento de tráfego, roteamento baseado em regras, ou escalabilidade automática.

## Limitações e Considerações

- **Custos Adicionais:** O uso de LoadBalancers pode ter custos adicionais, pois você está usando recursos adicionais do seu provedor de nuvem. Portanto, é sempre bom verificar os custos associados antes de implementar.

- **Dependência do Provedor de Nuvem:** A disponibilidade e os recursos específicos do Service LoadBalancer dependem do provedor de nuvem em que seu cluster Kubernetes está hospedado.

- **Configuração Personalizada:** Se você precisa de uma configuração personalizada para o balanceador de carga, pode ser necessário usar uma solução de balanceamento de carga externa em vez de depender do Service LoadBalancer padrão do Kubernetes.


## Criando um LoadBalancer

Criando um Service LoadBalancer via kubectl:

```
kubectl expose deployment meu-deployment --type=LoadBalancer --port=80 --target-port=8080
```

Ou via YAML:

```
apiVersion: v1
kind: Service
metadata:
  name: meu-service
spec:
  type: LoadBalancer
  selector:
    app: meu-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

## ExternalName

O Service ExternalName é um tipo de serviço no Kubernetes que não expõe um conjunto de Pods, mas sim um nome de host externo. Ele fornece uma maneira de acessar serviços externos ou sistemas legados de dentro do cluster Kubernetes usando um alias ou um nome de host conhecido.

### Características e Considerações:

- **Acesso a Serviços Externos:** O Service ExternalName é útil quando você precisa acessar serviços hospedados fora do cluster Kubernetes, como bancos de dados, sistemas de arquivos, ou qualquer outro serviço externo acessível por nome de host.

- **Alias para Serviços Externos:** Ele permite criar um alias para um serviço externo, permitindo que os aplicativos dentro do cluster se refiram a ele pelo mesmo nome de host que usariam se estivessem fora do cluster.

- **Abstração do Ambiente:** É útil para abstrair serviços que podem variar dependendo do ambiente (por exemplo, desenvolvimento, teste, produção), permitindo que os aplicativos usem o mesmo nome de host em diferentes ambientes, mas apontando para diferentes endereços externos.

- **Configuração Simples:** O Service ExternalName não requer a definição de seletores ou a especificação de portas, já que não expõe um conjunto de Pods. Basta especificar o nome de host externo que você deseja expor para o cluster Kubernetes.

- **Compatibilidade com DNS RFC-1123:** Aceita um nome de host válido de acordo com o DNS RFC-1123, que pode ser um nome de domínio ou um endereço IP.

Em resumo, o Service ExternalName é uma ferramenta útil para facilitar o acesso a serviços externos a partir de aplicativos em execução dentro do cluster Kubernetes, fornecendo uma camada de abstração e simplificando a configuração de comunicação entre diferentes ambientes e sistemas.


## Criando um ExternalName

Criando um Service ExternalName via kubectl:

```
kubectl create service externalname meu-service --external-name meu-db.giropops.com.br
```

Ou via YAML:

```
apiVersion: v1
kind: Service
metadata:
  name: meu-service
spec:
  type: ExternalName
  externalName: meu-db.giropops.com.br
```

## Verificando os Services

Para listar os Service:

```
kubectl get service
```

Para detalhes de um Service especifico:

```
kubectl describe service meu-service
```

## Verificando os Endpoints

Para listar os endpoints de service:

```
kubectl get endpoints meu-service
```

Para detalhers de um endpoint especifico:

```
kubectl describe endpoints meu-service
```

## Removendo um service

Para remover um Service:

```
kubectl delete service meu-service
```

Ou via YAML

```
kubectl delete -f meu-service.yaml
```

## Conclusão

Durante o Day-7 você aprendeu dois recursos essenciais do Kubernetes: **StatefulSets** e **Services**. Através deles, podemos gerenciar aplicações que precisam manter a identidade do Pod, persistir dados e ainda expor essas aplicações para o mundo externo.