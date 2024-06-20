# Day 8

# Kubernetes: Secrets e ConfigMaps

Vamos falar sobre duas peças fundamentais no Kubernetes: Secrets e ConfigMaps. Esses recursos armazenam informações sensíveis e de configuração para suas aplicações. Depois de hoje, você entenderá como armazenar e consumir essas informações no Kubernetes.

## O que são Secrets?

Secrets gerenciam informações sensíveis, como senhas, tokens OAuth, e chaves SSH. Eles são armazenados no Etcd e podem ser acessados por Pods ou pelo sistema Kubernetes.

### Funcionamento dos Secrets

- Armazenados no Etcd.
- Sem criptografia por padrão (criptografia disponível).
- Acesso restrito por RBAC.
- Montados em Pods como volumes ou usados como variáveis de ambiente.

### Tipos de Secrets

1. **Opaque Secrets**: Dados arbitrários como chaves de API e senhas.
2. **kubernetes.io/service-account-token**: Tokens de acesso de conta de serviço.
3. **kubernetes.io/dockercfg e kubernetes.io/dockerconfigjson**: Credenciais de registro do Docker.
4. **kubernetes.io/tls, kubernetes.io/ssh-auth, kubernetes.io/basic-auth**: Certificados TLS, chaves SSH e credenciais de autenticação básica.
5. **bootstrap.kubernetes.io/token**: Tokens de inicialização de cluster.

## Base64

Base64 não é criptografia, mas uma codificação binária para texto ASCII. Não fornece confidencialidade.

### Comandos Base64

```
# Codificar string em Base64
echo -n 'giropops' | base64
# Saída: Z2lyb3BvcHM=

# Decodificar string em Base64
echo -n 'Z2lyb3BvcHM=' | base64 -d
# Saída: giropops
```

## Criando um Secret

### Exemplo de YAML para Secret Opaque

```
apiVersion: v1
kind: Secret
metadata:
  name: giropops-secret
type: Opaque
data:
  username: SmVmZXJzb25fbGluZG8=
  password: Z2lyb3BvcHM=
```

### Comandos para Criar e Verificar Secrets

```
# Criar o Secret
kubectl create -f giropops-secret.yaml

# Verificar o Secret
kubectl get secret giropops-secret

# Ver detalhes do Secret
kubectl get secret giropops-secret -o yaml
kubectl describe secret giropops-secret
```

### Criando Secret com kubectl

```
kubectl create secret generic giropops-secret --from-literal=username=<SEGREDO> --from-literal=password=giropops
```

## Usando o Secret em um Pod

### Exemplo de YAML para Pod

```
apiVersion: v1
kind: Pod
metadata:
  name: giropops-pod
spec:
  containers:
  - name: giropops-container
    image: nginx
    env:
    - name: USERNAME
      valueFrom:
        secretKeyRef:
          name: giropops-secret
          key: username
    - name: PASSWORD
      valueFrom:
        secretKeyRef:
          name: giropops-secret
          key: password
```

### Comandos para Criar e Verificar Pod

```
# Criar o Pod
kubectl apply -f giropops-pod.yaml

# Verificar Pods
kubectl get pods

# Verificar variáveis de ambiente no Pod
kubectl exec giropops-pod -- env
```

Com isso, você tem um Pod que utiliza Secrets como variáveis de ambiente.


### Uso de Secrets e ConfigMaps no Kubernetes

Neste tutorial, vimos como utilizar dois componentes importantes do Kubernetes: **Secrets** e **ConfigMaps**.

---

### Secrets

**Secrets** no Kubernetes são usados para armazenar informações sensíveis, como senhas, tokens OAuth, chaves SSH, etc. Eles oferecem uma maneira segura de gerenciar essas informações sensíveis.

#### Criando um Secret para armazenar credenciais Docker

1. Primeiro, codifique o conteúdo do seu arquivo `config.json` do Docker em base64:
   
   ```
   base64 ~/.docker/config.json
   ```

2. Crie um arquivo chamado `dockerhub-secret.yaml`:
   ```
   apiVersion: v1
   kind: Secret
   metadata:
     name: docker-hub-secret
   type: kubernetes.io/dockerconfigjson
   data:
     .dockerconfigjson: |  # Conteúdo do seu config.json codificado em base64
       QXF1aSB0ZW0gcXVlIGVzdGFyIG8gY29udGXDumRvIGRvIHNldSBjb25maWcuanNvbiwgY29pc2EgbGluZGEgZG8gSmVmaW0=
   ```

3. Aplique o Secret no cluster:
   ```
   kubectl apply -f dockerhub-secret.yaml
   ```

4. Liste os Secrets para verificar:
   ```
   kubectl get secrets
   ```

5. Configure um Pod para usar o Secret:
   ```
   apiVersion: v1
   kind: Pod
   metadata:
     name: meu-pod
   spec:
     containers:
     - name: meu-container
       image: minha-imagem-privada
     imagePullSecrets:
     - name: docker-hub-secret  # Nome do Secret que armazena as credenciais do Docker Hub
   ```

#### Criando um Secret TLS

1. Gere um certificado TLS e uma chave privada:
   ```
   openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout chave-privada.key -out certificado.crt
   ```

2. Crie o Secret TLS:
   ```
   kubectl create secret tls meu-servico-web-tls-secret --cert=certificado.crt --key=chave-privada.key
   ```

3. Liste os Secrets para verificar:
   ```
   kubectl get secrets
   ```

4. Configure um Pod para usar o Secret TLS:
   ```
   apiVersion: v1
   kind: Pod
   metadata:
     name: nginx
     labels:
       app: nginx
   spec:
     containers:
     - name: nginx
       image: nginx
       ports:
       - containerPort: 80
       - containerPort: 443
       volumeMounts:
       - name: nginx-config-volume
         mountPath: /etc/nginx/nginx.conf
         subPath: nginx.conf
       - name: nginx-tls
         mountPath: /etc/nginx/tls
     volumes:
     - name: nginx-config-volume
       configMap:
         name: nginx-config
     - name: nginx-tls
       secret:
         secretName: meu-servico-web-tls-secret
         items:
           - key: tls.crt
             path: certificado.crt
           - key: tls.key
             path: chave-privada.key
   ```

---

### ConfigMaps

**ConfigMaps** são usados para armazenar dados de configuração, como variáveis de ambiente, arquivos de configuração, etc. Eles permitem desacoplar parâmetros de configuração das imagens de container, oferecendo flexibilidade e controle.

#### Criando e usando um ConfigMap

1. Crie um arquivo de configuração do Nginx (`nginx.conf`):
   ```
   events { }

   http {
     server {
       listen 80;
       listen 443 ssl;

       ssl_certificate /etc/nginx/tls/certificado.crt;
       ssl_certificate_key /etc/nginx/tls/chave-privada.key;

       location / {
         return 200 'Bem-vindo ao Nginx!\n';
         add_header Content-Type text/plain;
       }
     }
   }
   ```

2. Crie um ConfigMap a partir do arquivo:
   ```
   kubectl create configmap nginx-config --from-file=nginx.conf
   ```

3. Alternativamente, crie um ConfigMap usando um manifesto:
   ```
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: nginx-config
   data:
     nginx.conf: |
       events { }

       http {
         server {
           listen 80;
           listen 443 ssl;

           ssl_certificate /etc/nginx/tls/certificado.crt;
           ssl_certificate_key /etc/nginx/tls/chave-privada.key;

           location / {
             return 200 'Bem-vindo ao Nginx!\n';
             add_header Content-Type text/plain;
           }
         }
       }
   ```

4. Aplique o ConfigMap no cluster:
   ```
   kubectl apply -f nginx-config.yaml
   ```

5. Configure um Pod para usar o ConfigMap:
   ```
   apiVersion: v1
   kind: Pod
   metadata:
     name: nginx
     labels:
       app: nginx
   spec:
     containers:
     - name: nginx
       image: nginx
       ports:
       - containerPort: 80
       - containerPort: 443
       volumeMounts:
       - name: nginx-config-volume
         mountPath: /etc/nginx/nginx.conf
         subPath: nginx.conf
       - name: nginx-tls
         mountPath: /etc/nginx/tls
     volumes:
     - name: nginx-config-volume
       configMap:
         name: nginx-config
     - name: nginx-tls
       secret:
         secretName: meu-servico-web-tls-secret
         items:
           - key: tls.crt
             path: certificado.crt
           - key: tls.key
             path: chave-privada.key
   ```

6. Crie um Service para expor o Pod:
   ```
   kubectl expose pod nginx
   ```

7. Faça o port-forward para testar o Nginx:
   ```
   kubectl port-forward service/nginx 4443:443
   ```

8. Teste o Nginx:
   ```
   curl -k https://localhost:4443
   ```

---

### Final do Day-8

Hoje exploramos como usar **Secrets** e **ConfigMaps** no Kubernetes, criando exemplos práticos para armazenar credenciais Docker, certificados TLS e arquivos de configuração do Nginx. Essas ferramentas são fundamentais para gerenciar configurações e informações sensíveis de forma eficiente e segura em um cluster Kubernetes.

---

Espero que esta documentação tenha sido útil e que você tenha compreendido como utilizar esses recursos no Kubernetes. Vejo você no próximo dia! 👋 ✌️
