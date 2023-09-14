Você pode usar vários comandos de listagem no Kubernetes para visualizar diferentes tipos de recursos no seu cluster. Aqui estão alguns dos comandos de listagem mais comuns:

1. **Listar todos os Pods no namespace padrão:**

```bash
kubectl get pods
kubectl get pods -n metallb-system
kubectl -n metallb-system get all
```
OBS: -n siginifica NAMESPACE, voce coloca o -n e o nome do namespace, após isso você fornece o que você quer listar.

IPS DO NODES

```
lxc list
```

Para deletar pod metallb

```
kubectl delete pod controller-7d56b4f464-4k9dl -n metallb-system --grace-period=0 --force
kubectl delete pod speaker-gvkm7 -n metallb-system --grace-period=0 --force

```

Ver containers

```
kubectl get pods --all-namespaces -o jsonpath="{..image}" | tr -s '[[:space:]]' '\n' | sort | uniq -c

```

```
kubectl -n kube-system get cm
```

1. **Listar todos os Services no namespace padrão:**

```
kubectl get services
```

1. **Listar todos os Deployments no namespace padrão:**

```
kubectl get deployments
```

1. **Listar todos os StatefulSets no namespace padrão:**

```
kubectl get statefulsets
```

1. **Listar todos os ConfigMaps no namespace padrão:**

```
kubectl get configmaps
```

1. **Listar todos os Secrets no namespace padrão:**

```
kubectl get secrets
```

1. **Listar todos os Ingresses no namespace padrão:**

```

kubectl get ingresses
```

1. **Listar todos os PersistentVolumes:**

```
kubectl get pv
```

1. **Listar todos os PersistentVolumeClaims:**

```
kubectl get pvc
```

1. **Listar todos os Namespaces:**

```
kubectl get namespaces
```

1. **Listar todos os Nodes:**

```
kubectl get nodes
```

1. **Listar todos os ConfigMaps em um namespace específico (substitua `NOME_DO_NAMESPACE` pelo nome do namespace desejado):**

```
kubectl get configmaps -n NOME_DO_NAMESPACE
```

1. **Listar todos os Services em um namespace específico (substitua `NOME_DO_NAMESPACE` pelo nome do namespace desejado):**

```
kubectl get services -n NOME_DO_NAMESPACE
```

1. **Listar todos os Pods com base em um rótulo específico (substitua `CHAVE=VALOR` pelo rótulo desejado):**

```
kubectl get pods -l CHAVE=VALOR
```

1. **Listar todos os eventos no cluster para fins de diagnóstico:**

```
kubectl get events
```

