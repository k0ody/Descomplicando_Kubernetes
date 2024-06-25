## Day 10

Os ServiceMonitors são um dos principais recursos utilizados pelo Kube-Prometheus. Eles são objetos do Prometheus Operator que permitem configurar o Prometheus para monitorar serviços específicos em um cluster Kubernetes. Para cada serviço que você deseja monitorar, é necessário criar um ServiceMonitor.

### O que é um ServiceMonitor?

Um ServiceMonitor define um conjunto de endpoints que o Prometheus deve monitorar. Ele inclui informações como o caminho e a porta dos endpoints e permite a definição de rótulos personalizados. Com isso, você pode criar regras para filtrar quais serão os alvos monitorados pelo Prometheus, como por exemplo, monitorar apenas os pods com o rótulo `app=nginx`.

### Exemplo de ServiceMonitors já configurados

O Kube-Prometheus já vem com vários ServiceMonitors configurados, como os do API Server, Node Exporter, Blackbox Exporter, etc. Para visualizar os ServiceMonitors existentes, você pode usar o comando:

```
kubectl get servicemonitors -n monitoring
```

Este comando retornará uma lista de ServiceMonitors:

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

### Visualizando o conteúdo de um ServiceMonitor

Para ver o conteúdo de um ServiceMonitor específico, você pode usar o seguinte comando:

```
kubectl get servicemonitor prometheus-k8s -n monitoring -o yaml
```

Aqui está um exemplo do conteúdo do ServiceMonitor `prometheus-k8s`:

```
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  annotations:
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

### Explicação do YAML

- `apiVersion`: Versão da API do Kubernetes que estamos utilizando.
- `kind`: Tipo de objeto que estamos criando.
- `metadata`: Informações sobre o objeto, incluindo anotações, rótulos, nome e namespace.
- `spec`: Especificações do objeto, incluindo endpoints e seletores.
  - `spec.endpoints`: Lista de endpoints que o ServiceMonitor irá monitorar.
    - `interval`: Intervalo de tempo para coleta de métricas.
    - `port`: Porta que o Prometheus irá utilizar para coletar as métricas.
  - `spec.selector`: Selector que o ServiceMonitor irá utilizar para encontrar os serviços a serem monitorados.

### Criando um ServiceMonitor para o Nginx

Para criar um ServiceMonitor para monitorar o serviço Nginx, você pode usar o seguinte YAML:

```
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: nginx-servicemonitor
  labels:
    app: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  endpoints:
    - interval: 10s
      path: /metrics
      targetPort: 9113
```

Salve o arquivo como `nginx-servicemonitor.yaml` e aplique no cluster:

```
kubectl apply -f nginx-servicemonitor.yaml
```

Com isso, o Prometheus irá monitorar os serviços que possuem o rótulo `app=nginx`, coletando métricas do endpoint `/metrics` na porta `9113` a cada 10 segundos.


### Criação do ConfigMap

Crie um ConfigMap para definir a configuração do Nginx:

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  nginx.conf: |
    server {
      listen 80;
      location / {
        root /usr/share/nginx/html;
        index index.html index.htm;
      }
      location /metrics {
        stub_status on;
        access_log off;
      }
    }
```

Comando para aplicar o ConfigMap:

```
kubectl apply -f nginx-config.yaml
```

Verifique o ConfigMap criado:

```
kubectl get configmaps
```

### Criação do Deployment

Crie um Deployment para o Nginx e o Exporter:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-server
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
      annotations:
        prometheus.io/scrape: 'true'
        prometheus.io/port: '9113'
    spec:
      containers:
        - name: nginx
          image: nginx
          ports:
            - containerPort: 80
              name: http
          volumeMounts:
            - name: nginx-config
              mountPath: /etc/nginx/conf.d/default.conf
              subPath: nginx.conf
        - name: nginx-exporter
          image: 'nginx/nginx-prometheus-exporter:0.11.0'
          args:
            - '-nginx.scrape-uri=http://localhost/metrics'
          resources:
            limits:
              memory: 128Mi
              cpu: 0.3
          ports:
            - containerPort: 9113
              name: metrics
      volumes:
        - name: nginx-config
          configMap:
            name: nginx-config
            defaultMode: 420
```

Comando para aplicar o Deployment:

```
kubectl apply -f nginx-deployment.yaml
```

Verifique os pods criados:

```
kubectl get pods
```

Verifique o Deployment criado:

```
kubectl get deployments
```

### Criação do Service

Crie um Service para expor o Deployment:

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  labels:
    app: nginx
spec:
  ports:
    - port: 9113
      name: metrics
  selector:
    app: nginx
```

Comando para aplicar o Service:

```
kubectl apply -f nginx-service.yaml
```

Verifique o Service criado:

```
kubectl get services
```

### Verificação

Verifique se o Nginx está rodando:

```
curl http://<EXTERNAL-IP-DO-SERVICE>:80
```

Verifique as métricas do Nginx:

```
curl http://<EXTERNAL-IP-DO-SERVICE>:80/nginx_status
```

Verifique as métricas do Nginx Exporter:

```
curl http://<EXTERNAL-IP-DO-SERVICE>:80/metrics
```

### Criação do ServiceMonitor

Crie um ServiceMonitor para monitorar o Nginx:

```
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: nginx-servicemonitor
  labels:
    app: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  endpoints:
    - interval: 10s
      path: /metrics
      targetPort: 9113
```

Comando para aplicar o ServiceMonitor:

```
kubectl apply -f nginx-servicemonitor.yaml
```

### Resumo dos comandos

1. Criar ConfigMap:

    ```
    kubectl apply -f nginx-config.yaml
    ```

2. Verificar ConfigMap:

    ```
    kubectl get configmaps
    ```

3. Criar Deployment:

    ```
    kubectl apply -f nginx-deployment.yaml
    ```

4. Verificar pods:

    ```
    kubectl get pods
    ```

5. Verificar Deployment:

    ```
    kubectl get deployments
    ```

6. Criar Service:

    ```
    kubectl apply -f nginx-service.yaml
    ```

7. Verificar Service:

    ```
    kubectl get services
    ```

8. Verificar Nginx:

    ```
    curl http://<EXTERNAL-IP-DO-SERVICE>:80
    ```

9. Verificar métricas do Nginx:

    ```
    curl http://<EXTERNAL-IP-DO-SERVICE>:80/nginx_status
    ```

10. Verificar métricas do Nginx Exporter:

    ```
    curl http://<EXTERNAL-IP-DO-SERVICE>:80/metrics
    ```

11. Criar ServiceMonitor:

    ```
    kubectl apply -f nginx-servicemonitor.yaml
    ```

### Criação do ServiceMonitor

Crie um ServiceMonitor para monitorar o Nginx:

```
apiVersion: monitoring.coreos.com/v1 # Versão da API do Kubernetes que estamos utilizando
kind: ServiceMonitor # Tipo de objeto que estamos criando, no nosso caso, um ServiceMonitor
metadata: # Informações sobre o objeto que estamos criando
  name: nginx-servicemonitor # Nome do nosso objeto
  labels: # Labels que serão utilizadas para identificar o nosso objeto
    app: nginx # Label que identifica o app
spec: # Especificações do nosso objeto
  selector: # Seletor que será utilizado para identificar os pods que serão monitorados
    matchLabels: # Labels que identificam os pods que serão monitorados
      app: nginx # Label que identifica o app que será monitorado
  endpoints: # Endpoints que serão monitorados pelo Prometheus
    - interval: 10s # Intervalo de tempo entre as requisições
      path: /metrics # Caminho para a requisição
      targetPort: 9113 # Porta do target
```

Comando para aplicar o ServiceMonitor:

```
kubectl apply -f nginx-service-monitor.yaml
```

### Verificação do ServiceMonitor

Verifique se o ServiceMonitor foi criado:

```
kubectl get servicemonitors
```

### Port-forward do Prometheus

Faça o port-forward do Prometheus para acessar o Prometheus localmente:

```
kubectl port-forward -n monitoring svc/prometheus-k8s 39090:9090
```

### Verificação das métricas

Verifique se o Prometheus está capturando as métricas do Nginx e do Nginx Exporter:

```
curl http://localhost:39090/api/v1/targets
```

### Resumo dos comandos com comentários

1. **Criar o ServiceMonitor:**

    ```
    kubectl apply -f nginx-service-monitor.yaml
    ```

2. **Verificar o ServiceMonitor criado:**

    ```
    kubectl get servicemonitors
    ```

3. **Port-forward do Prometheus:**

    ```
    kubectl port-forward -n monitoring svc/prometheus-k8s 39090:9090
    ```

4. **Verificar se o Prometheus está capturando as métricas:**

    ```
    curl http://localhost:39090/api/v1/targets
    ```

### Comentários linha a linha do manifesto YAML

```
apiVersion: monitoring.coreos.com/v1 # Versão da API do Kubernetes que estamos utilizando
kind: ServiceMonitor # Tipo de objeto que estamos criando, no nosso caso, um ServiceMonitor
metadata: # Informações sobre o objeto que estamos criando
  name: nginx-servicemonitor # Nome do nosso objeto
  labels: # Labels que serão utilizadas para identificar o nosso objeto
    app: nginx # Label que identifica o app
spec: # Especificações do nosso objeto
  selector: # Seletor que será utilizado para identificar os pods que serão monitorados
    matchLabels: # Labels que identificam os pods que serão monitorados
      app: nginx # Label que identifica o app que será monitorado
  endpoints: # Endpoints que serão monitorados pelo Prometheus
    - interval: 10s # Intervalo de tempo entre as requisições
      path: /metrics # Caminho para a requisição
      targetPort: 9113 # Porta do target
```

Com os comandos e comentários adicionados, agora está claro o que cada parte do manifesto YAML faz e como aplicar e verificar o ServiceMonitor no Kubernetes.


### Criação do PodMonitor

Crie um PodMonitor para monitorar o Nginx:

```
apiVersion: monitoring.coreos.com/v1 # Versão da API do Kubernetes que estamos utilizando
kind: PodMonitor # Tipo de objeto que estamos criando, no nosso caso, um PodMonitor
metadata: # Informações sobre o objeto que estamos criando
  name: nginx-podmonitor # Nome do nosso objeto
  labels: # Labels que serão utilizadas para identificar o nosso objeto
    app: nginx # Label que identifica o app
spec: # Especificações do nosso objeto
  namespaceSelector: # Seletor de namespaces
    matchNames: # Namespaces que serão monitorados
      - default # Namespace que será monitorado
  selector: # Seletor para identificar os pods que serão monitorados
    matchLabels: # Labels que identificam os pods que serão monitorados
      app: nginx # Label que identifica o app que será monitorado
  podMetricsEndpoints: # Endpoints que serão monitorados pelo Prometheus
    - interval: 10s # Intervalo de tempo entre as requisições
      path: /metrics # Caminho para a requisição
      targetPort: 9113 # Porta do target
```

Comando para aplicar o PodMonitor:

```
kubectl apply -f nginx-pod-monitor.yaml
```

### Verificação do PodMonitor

Verifique se o PodMonitor foi criado:

```
kubectl get podmonitors
```

### Detalhamento do PodMonitor

Veja os detalhes do PodMonitor:

```
kubectl describe podmonitors nginx-podmonitor
```

### Criação do Pod Nginx

Antes de criar o PodMonitor, crie o Pod do Nginx:

```
apiVersion: v1 # Versão da API do Kubernetes que estamos utilizando
kind: Pod # Tipo de objeto que estamos criando, no nosso caso, um Pod
metadata: # Informações sobre o objeto que estamos criando
  name: nginx-pod # Nome do nosso objeto
  labels: # Labels que serão utilizadas para identificar o nosso objeto
    app: nginx # Label que identifica o app
spec: # Especificações do nosso objeto
  containers: # Containers do template 
    - name: nginx-container # Nome do container
      image: nginx # Imagem do container do Nginx
      ports: # Portas do container
        - containerPort: 80 # Porta do container
          name: http # Nome da porta
      volumeMounts: # Volumes que serão montados no container
        - name: nginx-config # Nome do volume
          mountPath: /etc/nginx/conf.d/default.conf # Caminho de montagem do volume
          subPath: nginx.conf # Subpath do volume
    - name: nginx-exporter # Nome do container que será o exporter
      image: 'nginx/nginx-prometheus-exporter:0.11.0' # Imagem do container do exporter
      args: # Argumentos do container
        - '-nginx.scrape-uri=http://localhost/metrics' # Argumento para definir a URI de scraping
      resources: # Recursos do container
        limits: # Limites de recursos
          memory: 128Mi # Limite de memória
          cpu: 0.3 # Limite de CPU
      ports: # Portas do container
        - containerPort: 9113 # Porta do container que será exposta
          name: metrics # Nome da porta
  volumes: # Volumes do template
    - configMap: # ConfigMap do volume, nós iremos criar esse volume através de um configmap
        defaultMode: 420 # Modo padrão do volume
        name: nginx-config # Nome do configmap
      name: nginx-config # Nome do volume
```

Comando para criar o Pod:

```
kubectl apply -f nginx-pod.yaml
```

### Port-forward do Prometheus

Faça o port-forward do Prometheus para acessar o Prometheus localmente:

```
kubectl port-forward -n monitoring svc/prometheus-k8s 39090:9090
```

### Verificação das métricas

Verifique se o Prometheus está capturando as métricas do Nginx:

```
curl http://localhost:39090/api/v1/targets
```

### Acesso ao container Nginx Exporter

Acesse o container Nginx Exporter para verificar se está funcionando corretamente:

```
kubectl exec -it nginx-pod -c nginx-exporter -- bash
```

### Verificação das métricas do exporter

Use o curl para verificar se o exporter está funcionando corretamente:

```
curl localhost:9113/metrics
```

### Resumo dos comandos

1. **Criar o Pod do Nginx:**

    ```
    kubectl apply -f nginx-pod.yaml
    ```

2. **Criar o PodMonitor:**

    ```
    kubectl apply -f nginx-pod-monitor.yaml
    ```

3. **Verificar o PodMonitor criado:**

    ```
    kubectl get podmonitors
    ```

4. **Ver os detalhes do PodMonitor:**

    ```
    kubectl describe podmonitors nginx-podmonitor
    ```

5. **Port-forward do Prometheus:**

    ```
    kubectl port-forward -n monitoring svc/prometheus-k8s 39090:9090
    ```

6. **Verificar se o Prometheus está capturando as métricas:**

    ```
    curl http://localhost:39090/api/v1/targets
    ```

7. **Acessar o container Nginx Exporter:**

    ```
    kubectl exec -it nginx-pod -c nginx-exporter -- bash
    ```

8. **Verificar as métricas do exporter:**

    ```
    curl localhost:9113/metrics
    ```

Com esses passos, você pode monitorar pods individuais no Kubernetes usando o Prometheus Operator e o PodMonitor.


### Criando nosso primeiro alerta

Agora que já temos o Kube-Prometheus instalado, vamos configurar o Prometheus para monitorar nosso cluster EKS. Para isso, usaremos o `kubectl port-forward` para acessar o Prometheus localmente:

```
kubectl port-forward -n monitoring svc/prometheus-k8s 39090:9090
```

Se você quiser acessar o Alertmanager, execute o comando:

```
kubectl port-forward -n monitoring svc/alertmanager-main 39093:9093
```

Você pode acessar o Prometheus e o Alertmanager através das seguintes URLs:

- **Prometheus:** [http://localhost:39090](http://localhost:39090)
- **Alertmanager:** [http://localhost:39093](http://localhost:39093)

### Explorando ConfigMaps de Alertas

Para listar os ConfigMaps no namespace `monitoring`, execute:

```
kubectl get configmaps -n monitoring
```

O resultado deverá ser algo parecido com:

```
NAME                                                  DATA   AGE
adapter-config                                        1      7m20s
blackbox-exporter-configuration                       1      7m49s
grafana-dashboard-alertmanager-overview               1      7m46s
...
prometheus-k8s-rulefiles-0                            8      7m10s
```

Para visualizar o conteúdo do ConfigMap `prometheus-k8s-rulefiles-0`, execute:

```
kubectl get configmap prometheus-k8s-rulefiles-0 -n monitoring -o yaml
```

A saída contém exemplos de alertas. Aqui está um exemplo:

```
- alert: KubeMemoryOvercommit
  annotations:
    description: Cluster has overcommitted memory resource requests for Pods by {{ $value | humanize }} bytes and cannot tolerate node failure.
    runbook_url: https://runbooks.prometheus-operator.dev/runbooks/kubernetes/kubememoryovercommit
    summary: Cluster has overcommitted memory resource requests.
  expr: |
    sum(namespace_memory:kube_pod_container_resource_requests:sum{}) - (sum(kube_node_status_allocatable{resource="memory"}) - max(kube_node_status_allocatable{resource="memory"})) > 0
    and
    (sum(kube_node_status_allocatable{resource="memory"}) - max(kube_node_status_allocatable{resource="memory"})) > 0
  for: 10m
  labels:
    severity: warning
```

### Criando um novo alerta

Para criar um novo alerta, usaremos o recurso `PrometheusRule`. Vamos criar um arquivo chamado `nginx-prometheus-rule.yaml` com o seguinte conteúdo:

```
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: nginx-prometheus-rule
  namespace: monitoring
  labels:
    prometheus: k8s
    role: alert-rules
    app.kubernetes.io/name: kube-prometheus
    app.kubernetes.io/part-of: kube-prometheus
spec:
  groups:
  - name: nginx-prometheus-rule
    rules:
    - alert: NginxDown
      expr: up{job="nginx"} == 0
      for: 1m
      labels:
        severity: critical
      annotations:
        summary: "Nginx is down"
        description: "Nginx is down for more than 1 minute. Pod name: {{ $labels.pod }}"
```

### Aplicando o PrometheusRule

Para criar o `PrometheusRule` no cluster, execute:

```
kubectl apply -f nginx-prometheus-rule.yaml
```

Verifique se o `PrometheusRule` foi criado com sucesso:

```
kubectl get prometheusrules -n monitoring
```

### Adicionando uma nova regra de alerta

Vamos adicionar uma nova regra de alerta ao mesmo arquivo `nginx-prometheus-rule.yaml`:

```
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: nginx-prometheus-rule
  namespace: monitoring
  labels:
    prometheus: k8s
    role: alert-rules
    app.kubernetes.io/name: kube-prometheus
    app.kubernetes.io/part-of: kube-prometheus
spec:
  groups:
  - name: nginx-prometheus-rule
    rules:
    - alert: NginxDown
      expr: up{job="nginx"} == 0
      for: 1m
      labels:
        severity: critical
      annotations:
        summary: "Nginx is down"
        description: "Nginx is down for more than 1 minute. Pod name: {{ $labels.pod }}"
    - alert: NginxHighRequestRate
      expr: rate(nginx_http_requests_total{job="nginx"}[5m]) > 10
      for: 1m
      labels:
        severity: warning
      annotations:
        summary: "Nginx is receiving high request rate"
        description: "Nginx is receiving high request rate for more than 1 minute. Pod name: {{ $labels.pod }}"
```

### Atualizando o PrometheusRule

Para aplicar a atualização, execute:

```
kubectl apply -f nginx-prometheus-rule.yaml
```

Verifique se o `PrometheusRule` foi atualizado:

```
kubectl get prometheusrules -n monitoring nginx-prometheus-rule -o yaml
```

A saída deve incluir as novas regras de alerta:

```
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: nginx-prometheus-rule
  namespace: monitoring
  labels:
    prometheus: k8s
    role: alert-rules
    app.kubernetes.io/name: kube-prometheus
    app.kubernetes.io/part-of: kube-prometheus
spec:
  groups:
  - name: nginx-prometheus-rule
    rules:
    - alert: NginxDown
      expr: up{job="nginx"} == 0
      for: 1m
      labels:
        severity: critical
      annotations:
        summary: "Nginx is down"
        description: "Nginx is down for more than 1 minute. Pod name: {{ $labels.pod }}"
    - alert: NginxHighRequestRate
      expr: rate(nginx_http_requests_total{job="nginx"}[5m]) > 10
      for: 1m
      labels:
        severity: warning
      annotations:
        summary: "Nginx is receiving high request rate"
        description: "Nginx is receiving high request rate for more than 1 minute. Pod name: {{ $labels.pod }}"
```

Agora, os novos alertas estão configurados e aparecerão no Prometheus e no AlertManager conforme configurado.

---

Essa seção demonstrou como criar e aplicar um `PrometheusRule` para adicionar alertas ao seu sistema de monitoramento Prometheus. Explore outras possibilidades e configure mais alertas conforme necessário para monitorar outros aspectos do seu ambiente Kubernetes.

