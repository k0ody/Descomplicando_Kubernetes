## Day 4

**ReplicaSet**

Em Kubernetes, um ReplicaSet é um controlador que garante que um número especificado de réplicas de um pod esteja sempre em execução. Ele é uma abstração que define um conjunto de pods idênticos, garantindo que a quantidade desejada esteja sempre disponível, mesmo após falhas ou atualizações.

Os ReplicaSets são frequentemente usados para garantir alta disponibilidade e escalabilidade de aplicativos em um cluster do Kubernetes. Eles substituíram os controladores de replicação originais do Kubernetes.

Quando você cria um ReplicaSet, você especifica um número desejado de réplicas (pods) para manter em execução. O ReplicaSet então monitora constantemente o estado atual dos pods e faz ajustes para garantir que o número desejado de réplicas esteja em execução. Se houver menos réplicas do que o especificado, o ReplicaSet criará novos pods. Se houver mais réplicas, ele os eliminará até que o número desejado seja alcançado novamente.

Além disso, os ReplicaSets podem ser usados em conjunto com outras abstrações do Kubernetes, como os Deployments, para facilitar o gerenciamento de implantações de aplicativos, permitindo atualizações controladas e rollbacks.

Em resumo, um ReplicaSet é uma ferramenta fundamental para garantir a disponibilidade e escalabilidade de aplicativos em um ambiente Kubernetes.


**DaemonSet**

Em Kubernetes, um DaemonSet é outro tipo de controlador que garante que uma cópia específica de um pod seja executada em cada nó (node) do cluster. Enquanto um ReplicaSet garante um número específico de réplicas de um pod em todo o cluster, um DaemonSet garante que cada nó tenha uma instância desse pod em execução.

Os DaemonSets são úteis para casos em que você precisa executar um pod em cada nó para realizar tarefas de infraestrutura, como coleta de logs, monitoramento de recursos ou configurações de rede específicas do nó. Eles garantem que as tarefas sejam distribuídas uniformemente em todos os nós do cluster, independentemente do tamanho ou da dinâmica do cluster.

Por exemplo, se você estiver usando um DaemonSet para coletar logs de todos os nós, cada nó terá seu próprio pod de coleta de logs em execução, garantindo que nenhum nó seja negligenciado na coleta de logs.

Outro exemplo comum de uso de DaemonSet é para a implantação de agentes de monitoramento em cada nó do cluster Kubernetes. Vamos imaginar que você esteja usando o Prometheus para monitorar seus aplicativos e recursos no cluster. Você pode configurar um DaemonSet para garantir que um pod do Prometheus Node Exporter seja executado em cada nó do cluster.

Os DaemonSets são configurados com um conjunto de especificações semelhantes aos ReplicaSets, incluindo a especificação de qual imagem de contêiner usar, como configurar o contêiner e quais nodos devem ser selecionados para execução do pod.

Em resumo, um DaemonSet é usado para garantir que um pod específico seja executado em cada nó do cluster Kubernetes, facilitando a execução de tarefas específicas em todos os nós de forma automatizada e uniforme.

Lembre-se, não é possivel criar um daemonset na linha de comando, apenas com manifestos.

**Probes**

Em Kubernetes, as probes são mecanismos que permitem que o sistema avalie o estado de um contêiner em execução em um pod. Elas são usadas para determinar se um contêiner está em um estado saudável e pronto para receber tráfego de rede ou se precisa ser reiniciado.

Existem três tipos principais de probes em Kubernetes:

**Liveness Probe (sonda de vitalidade)**

Uma liveness probe é usada para determinar se o contêiner está em um estado saudável. Se a sonda detectar que o contêiner não está saudável (por exemplo, se um aplicativo travar), o Kubernetes reiniciará automaticamente o contêiner. Isso é útil para garantir que os aplicativos sejam mantidos em um estado funcional.

**Readiness Probe (sonda de prontidão)**

Uma readiness probe é usada para determinar se um contêiner está pronto para começar a receber tráfego de rede. Se um contêiner falhar em uma sonda de prontidão, o Kubernetes removerá o pod do serviço de balanceamento de carga, garantindo que o tráfego de rede não seja enviado para um contêiner que ainda não está pronto para receber solicitações.

**Startup Probe (sonda de inicialização)**

Uma startup probe é usada para determinar quando um contêiner está pronto para começar a receber tráfego de rede durante a inicialização. Ela é usada para casos em que o tempo de inicialização do contêiner pode ser mais longo do que o período de falha de inicialização padrão do Kubernetes. Se o Kubernetes detectar que um contêiner está demorando muito para inicializar, ele poderá reiniciar o pod.