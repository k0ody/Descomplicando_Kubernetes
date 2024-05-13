 ## Principais Comandos para Verificação de Volumes, StorageClasses, PVs e PVCs

Listar todos os Persistent Volume Claims (PVCs):

 ```
kubectl get pvc
 ```

Descrever detalhes de um PVC especifico:

```
kubectl describe pvc <nome-do-pvc>
```

Listar todas as StorageClasses:

```
kubectl get storageclasses
```

Descrever detalhes de uma StorageClass específica:

```
kubectl describe storageclass <nome-da-storageclass>
```

Listar todos os Persistent Volumes (PVs):

```
kubectl get pv
```

Descrever detalhes de um PV específico:

```
kubectl describe pv <nome-do-pv>
```

