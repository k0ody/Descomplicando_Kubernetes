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
   - Exemplo de uso: `kubectl exec -it nome-do-pod -- bash`

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

