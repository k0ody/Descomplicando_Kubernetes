 ## Day 4

 **Deployment**

 Um **Deployment** no Kubernetes é um recurso fundamental para gerenciar e garantir a disponibilidade de uma aplicação. Ele oferece uma abstração para gerenciar os Pods que compõem a aplicação, permitindo atualizações e rollbacks de forma controlada.

### Principais pontos sobre Deployment:

1. **Gerenciamento de Réplicas:** Um Deployment permite definir o número de réplicas desejado para a aplicação. Ele garante que o número de Pods em execução corresponda ao número de réplicas definido, substituindo automaticamente os Pods que falharam.

2. **Declarativo:** A configuração de um Deployment é declarativa, o que significa que você descreve o estado desejado da aplicação e o Kubernetes se encarrega de garantir que o estado atual corresponda ao estado desejado.

3. **Atualizações Controladas:** Com um Deployment, você pode atualizar sua aplicação de forma controlada, garantindo que apenas uma parte dos Pods seja atualizada por vez (estratégia de rolling update). Isso minimiza o impacto das atualizações na disponibilidade da aplicação.

4. **Rollbacks:** Se uma atualização causar problemas, você pode facilmente fazer um rollback para uma versão anterior do aplicativo, restaurando o estado anterior da aplicação.

5. **Integração com ReplicaSets:** Ao criar um Deployment, você também cria automaticamente um ReplicaSet associado, que gerencia os Pods específicos daquela versão do Deployment.

Em resumo, um Deployment no Kubernetes simplifica e automatiza o gerenciamento de aplicativos, garantindo alta disponibilidade, controle de atual

Abaixo um exemplo de um arquivo chamado deployment.yaml.

```
apiVersion: apps/v1   # Versão da API do Kubernetes utilizada
kind: Deployment      # Tipo de recurso Kubernetes sendo criado
metadata:             # Metadados do Deployment
  labels:             # Rótulos associados ao Deployment
    app: nginx-deployment   # Rótulo que identifica este Deployment como parte da aplicação nginx-deployment
  name: nginx-deployment    # Nome do Deployment
spec:                 # Especificações do Deployment
  replicas: 1         # Número de réplicas desejadas do Pod gerenciado pelo Deployment
  selector:           # Seletor para encontrar os Pods gerenciados por este Deployment
    matchLabels:
      app: nginx-deployment   # Rótulo usado para selecionar os Pods gerenciados por este Deployment
  strategy: {}         # Estratégia de update (por padrão, é uma estratégia de rolling update)
  template:            # Template para os Pods gerenciados pelo Deployment
    metadata:         # Metadados do template
      labels:         # Rótulos aplicados aos Pods gerenciados por este Deployment
        app: nginx-deployment   # Rótulo associado aos Pods gerenciados por este Deployment
    spec:             # Especificações do Pod
      containers:     # Lista de contêineres que serão executados no Pod
      - image: nginx  # Imagem do contêiner a ser executada nos Pods
        name: nginx   # Nome do contêiner
        resources:    # Recursos do contêiner (CPU e memória)
          limits:     # Limites máximos de recursos que o contêiner pode usar
            cpu: "0.5"        # Limite máximo de CPU que o contêiner pode usar
            memory: 256Mi     # Limite máximo de memória que o contêiner pode usar
          requests:   # Solicitações mínimas de recursos que o contêiner precisa
            cpu: "0.25"       # Quantidade de CPU solicitada para o contêiner
            memory: 128Mi     # Quantidade de memória solicitada para o contêiner


```

**Para aplicar o deplpyment:**

```
kubectl apply -f deployment.yaml
```

**Verificar se o deployment foi criado:**

Estamos passando a label nginx-deployment para filtrar a consulta

```
kubectl get deployments -l app=nginx-deployment
```

**Verificar se o ReplicaSet:**

Estamos passando a label nginx-deployment para filtrar a consulta

```
kubectl get replicaset -l app=nginx-deployment
```

**Detalhes de um deployment**

```
kubectl describe deployment nginx-deployment
```

Abaixo os principais detalhes que podem ser verificados:

- **Nome e Namespace:** Indicam o nome e o namespace do Deployment.
- **Labels:** Mostra os rótulos associados ao Deployment.
- **Replicas:** Informa o número de réplicas desejadas, atualizadas, totais e disponíveis para o Deployment.
- **Selector:** Define os rótulos usados pelo Deployment para identificar os Pods que ele gerencia.
- **Estratégia de Atualização:** Especifica a estratégia de atualização utilizada pelo Deployment (por exemplo, rolling update).
- **Pod Template:** Descreve o template usado para criar os Pods, incluindo a imagem do contêiner, limites de CPU e memória, solicitações de recursos e outras        configurações.
- **Condições:** Mostra o status das condições do Deployment, como disponibilidade e progresso.
- **ReplicaSets:** Lista os ReplicaSets associados ao Deployment, incluindo o ReplicaSet antigo e o novo, se houver.
- **Eventos:** Registra os eventos relacionados ao Deployment, como escala para cima ou para baixo, e a criação de novos ReplicaSets.


## Update e Rollback do Deployment

**Update**

Existe dois modos para realizar a atualização dos pods em um deployment

**RollingUpdate:**

- O modo de atualização rolling update é o padrão usado pelo Kubernetes.
- Com essa estratégia, o Kubernetes atualiza os Pods gradualmente, um após o outro, mantendo uma quantidade específica de Pods antigos em execução enquanto inicia - - novos Pods com a versão atualizada.
- Durante a atualização, o número de Pods em execução é gradualmente ajustado para garantir que sempre haja um número mínimo de Pods disponíveis, garantindo assim a disponibilidade contínua da aplicação.
- Essa estratégia é ideal para aplicativos que precisam de alta disponibilidade e não podem tolerar interrupções significativas durante a atualização.

**Recreate**

- No modo de atualização recreate, todos os Pods existentes são interrompidos e substituídos por novos Pods com a versão atualizada.
- Isso significa que não há garantia de disponibilidade contínua da aplicação durante a atualização, pois todos os Pods antigos são interrompidos antes que os novos sejam iniciados.
- Essa estratégia é mais simples e direta, mas pode resultar em períodos de tempo em que a aplicação não está disponível.
É útil em situações onde a disponibilidade imediata da aplicação não é crítica ou quando há necessidade de realizar uma reinicialização completa dos Pods.

Em resumo, o modo de atualização rolling update é mais adequado para aplicativos que exigem alta disponibilidade e tolerância a falhas mínima durante a atualização, enquanto o modo de atualização recreate é mais simples, mas pode resultar em períodos de indisponibilidade da aplicação durante a atualização. A escolha entre os dois depende dos requisitos específicos da aplicação e das necessidades de disponibilidade.

**Exemplos**

**RollingUpdate:**

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deployment
  name: nginx-deployment
spec:
  replicas: 10
  selector:
    matchLabels:
      app: nginx-deployment
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 2
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - image: nginx:1.15.0
        name: nginx
        resources:
          limits:
            cpu: "0.5"
            memory: 256Mi
          requests:
            cpu: "0.25"
            memory: 128Mi
```

Onde foi adicionado os campos:

```
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 2
```

Onde:

**maxSurge**: define a quantidade máxima de Pods que podem ser criados a mais durante a atualização, ou seja, durante o processo de atualização, nós podemos ter 1 Pod a mais do que o número de Pods definidos no Deployment. Isso é útil pois agiliza o processo de atualização, pois o Kubernetes não precisa esperar que um Pod seja atualizado para criar um novo Pod.

**maxUnavailable**: define a quantidade máxima de Pods que podem ficar indisponíveis durante a atualização, ou seja, durante o processo de atualização, nós podemos ter 1 Pod indisponível por vez. Isso é útil pois garante que o serviço não fique indisponível durante a atualização.

**type**: define o tipo de estratégia de atualização que será utilizada, no nosso caso, nós estamos utilizando a estratégia RollingUpdate.


Comando util para verificar a atualização dos pods:

```
kubectl rollout status deployment <nome do deployment>
```

**Recreate:**

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deployment
  name: nginx-deployment
spec:
  replicas: 10
  selector:
    matchLabels:
      app: nginx-deployment
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - image: nginx:1.15.0
        name: nginx
        resources:
          limits:
            cpu: "0.5"
            memory: 256Mi
          requests:
            cpu: "0.25"
            memory: 128Mi
```

O compo que foi adiciona:

```
  strategy:
    type: Recreate
```

Perceba que agora somente temos a configuração type: Recreate. O Recreate não possui configurações de atualização, ou seja, não é possível definir o número máximo de Pods indisponíveis durante a atualização, afinal o Recreate irá remover todos os Pods do Deployment e criar novos Pods.


**Rollback**

Um rollback em um Deployment no Kubernetes é o processo de reverter uma atualização para uma versão anterior da aplicação em caso de problemas ou falhas. Essa funcionalidade é crucial para garantir a disponibilidade contínua da aplicação e minimizar o impacto de atualizações mal-sucedidas.

Quando um rollback é executado, o Kubernetes faz o seguinte:

- Encerra os Pods da versão atual que foram implantados durante a atualização.
- Inicia os Pods da versão anterior que estavam em execução antes da atualização.
- Atualiza o ReplicaSet associado ao Deployment para apontar para a versão anterior dos Pods.
- Garante que a aplicação retorne ao estado anterior antes da atualização, restaurando assim a funcionalidade anterior.

O Kubernetes permite realizar rollbacks de forma rápida e automática usando comandos simples. Aqui estão alguns exemplos de comandos para realizar um rollback em um Deployment:

**Verificar historico de revisão do Deployment:**

```
kubectl rollout history deployment <nome do deployment>
```

**Reverter para ultima revisão**

```
kubectl rollout undo deployment <nome do deployment>
```

**Reverter para uma revisão especifica**

```
kubectl rollout undo deployment <nome do deployment> --to-revision=<numero da revisao>
```

**Verificar status do rollback**

```
kubectl rollout status deployment <nome do deployment>
```

**Pausar um Deployment**

```
kubectl rollout pause deployment <nome do deployment>
```
- Este comando é usado para pausar uma atualização em andamento de um Deployment.
- Quando uma atualização é pausada, o Kubernetes interrompe a progressão para a próxima etapa de atualização, mantendo os Pods existentes em execução.
- Isso é útil quando você precisa interromper temporariamente uma atualização para investigar problemas ou realizar tarefas de manutenção

**Retormar um Deployment**
```
kubectl rollout resume deployment <nome do deployment>
```
- Este comando é usado para retomar uma atualização que foi pausada anteriormente em um Deployment.
- Quando uma atualização é retomada, o Kubernetes continua a progressão da atualização, iniciando os Pods da nova versão e terminando gradualmente os Pods da versão anterior.

**Restar Deployment**

```
kubectl rollout restart deployment <nome do deployment>
```
- Este comando é usado para reiniciar uma atualização que falhou ou para forçar uma reinicialização de todos os Pods do Deployment, independentemente do status da atualização.
- Ao reiniciar, o Kubernetes interrompe todos os Pods existentes e inicia novos Pods com a configuração atualizada.


Esses comandos permitem verificar o histórico de revisões do Deployment, reverter para uma revisão anterior específica e verificar o status do rollback em andamento.

É importante notar que o rollback no Kubernetes é uma operação segura e controlada, garantindo que a aplicação retorne a um estado funcional anterior em caso de problemas durante uma atualização.

Podemos também gerenciar o ciclo de vida das atualizações em Deployments, permitindo pausar, retomar e reiniciar atualizações conforme necessário para garantir a disponibilidade e a integridade da aplicação.


## Conclusão

- Aprendemos que um Deployment é um recurso fundamental para gerenciar e garantir a disponibilidade de uma aplicação no Kubernetes.
- Vimos como criar um Deployment usando um arquivo YAML que define as especificações desejadas para a aplicação.
- Exploramos como atualizar um Deployment, tanto usando a estratégia padrão de rolling update quanto fazendo um rollback para uma versão anterior em caso de problemas.
- Discutimos como remover um Deployment quando não precisamos mais dele.
- Conhecemos comandos úteis, como `kubectl describe deployment` para verificar os detalhes do Deployment, e `kubectl rollout` para gerenciar atualizações e rollbacks.
- Aprendemos sobre diferentes estratégias de atualização, como rolling update e recreate, e como cada uma delas pode afetar a disponibilidade da aplicação durante o processo de atualização.
- Finalmente, discutimos comandos adicionais, como `rollout pause`, `rollout resume` e `rollout restart`, que nos ajudam a gerenciar o ciclo de vida das atualizações em Deployments.

Com esse conhecimento, estamos prontos para começar a trabalhar com Deployments no Kubernetes e aproveitar ao máximo suas capacidades de implantação e gerenciamento de aplicativos.
