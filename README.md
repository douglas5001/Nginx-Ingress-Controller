# Nginx-Ingress-Controller
O projeto abaixo consiste em um Cluster Kubernetes criado em ambiente On-premise, sem a dependência de um provedor cloud, para isso vamos utilizar a solução Bare Metal, serviço MetalLB irá atender a demanda.

Segue uma imagem mostrando como funcionaria a comunicação entres os agentes

![Untitled](https://github.com/douglas5001/Nginx-Ingress-Controller/blob/main/nodeport.jpg)

Nessa configuração para do Nginx com proxy reverse é necessário que voce já tenha um cluster com no mínimo 1 laço como mostrado abaixo

```python
root@vmi1274367:~# kubectl get nodes
NAME                           STATUS   ROLES                  AGE    VERSION
node1                          Ready    <none>                 4d4h   v1.27.5+k3s1
host.master                    Ready    control-plane,master   4d4h   v1.27.5+k3s1
```

Caso ainda não tenha configurado, poderá realizar a configuração do k3s no link abaixo https://docs.k3s.io/quick-start

Você pode configurar o k8s no repositório git abaixo.

https://github.com/douglas5001/K8s-Control-Plane

Nesse caso vamos utilizar o k3s pela simplicidade da configuração e remoção.

Com o seu cluster configurado vamos partir para os próximos requisitos

## Configuração do MetalLB

Como estamos configurando o Kubernetes em ambiente onPremise, não temos o suporte do Load balance Cloud que é oferecido sob demanda pelos provedores cloud, portando temos o **`MetalLB`** como solucao para ambiente onPremises.

Realize a instalação seguindo a documentação official

(ABAIXO ESTARA OS COMANDOS, MAS É IMPORTANTE QUE VOCE CHEQUE A DOCUMENTACAO OFICIAL)

https://metallb.universe.tf/installation/

-Realize a alteração do arquivo  kube-proxy caso esteja utilizando o K8s

Você pode conseguir isso editando a configuração do kube-proxy no cluster atual:

```bash
kubectl edit configmap -n kube-system kube-proxy
```

E definir:

```yaml
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: "ipvs"
ipvs:
  strictARP: true
```

Você também pode adicionar este snippet de configuração ao seu kubeadm-config, basta anexá-lo `---`após a configuração principal.

Se você está tentando automatizar essa alteração, estes trechos de shell podem ajudá-lo:

```bash
# see what changes would be made, returns nonzero returncode if different
kubectl get configmap kube-proxy -n kube-system -o yaml | \
sed -e "s/strictARP: false/strictARP: true/" | \
kubectl diff -f - -n kube-system# actually apply the changes, returns nonzero returncode on errors only
kubectl get configmap kube-proxy -n kube-system -o yaml | \
sed -e "s/strictARP: false/strictARP: true/" | \
kubectl apply -f - -n kube-system
```

-Caso esteja utilizando o k3s poderá pular para os passos abaixo

## Instalação Por Manifesto

Para instalar o MetalLB, aplique o manifesto:

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.11/config/manifests/metallb-native.yaml
```

## Definindo Os IPs A Serem Atribuídos Aos Serviços Do Load Balancer

Documentação Oficial https://metallb.universe.tf/configuration/

Para atribuir um IP aos serviços, o MetalLB deverá ser instruído a fazê-lo através do `IPAddressPool`CR.

Todos os IPs alocados via `IPAddressPool`s contribuem para o conjunto de IPs que o MetalLB utiliza para atribuir IPs aos serviços.

Crie um yaml e guarde no repósitorio de sua escolha

```python
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: first-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.10.0/24
  - 192.168.9.1-192.168.9.5
  - fc00:f853:0ccd:e799::/124
```

Salve como `IPAddressPool.yaml`

OBS: caso voce não tenha mais de um ip para distribuir, podemos setar um quando criarmos o LoadBalance.

 

## Instalação do Nginx-Ingress-Controller

Instalação do Helm

Documentação Official

https://helm.sh/docs/intro/install/

Após instalara o helm, valide se a updates com o comando abaixo

```python
helm repo update
```

Instalando Nginx-Controller

```python
helm install ingress-nginx ingress-nginx/ingress-nginx
```

Após a instalação, já devera aparecer o namespace do Nginx e seus pods, como o exemplo abaixo

```python
root@vmi1274367:~# kubectl get pods -n ingress-nginx
NAME                                       READY   STATUS             RESTARTS          AGE
ingress-nginx-admission-create-pkh8t       0/1     Completed          0                 4d10h
ingress-nginx-admission-patch-krnpw        0/1     Completed          1                 4d10h
ingress-nginx-controller-f7f5995cc-8przl   0/1     CrashLoopBackOff   182 (3m31s ago)   4d10h
```

---

# Criando a Web

Agora com o nosso ambiente Kubernetes pronto já podemos dar seguimento na criação do nosso Namespace e seus componentes como, service, ingress, loadBalance e Deployment.

Interessante para seguir esses próximos passos é que voce tenha um conhecimento básico sobre a funcionalidade de yamls, como funciona o proxy-revers, ingress, service, namespace e os deployments.

para isso indico dar uma olhada no video abaixo.

https://www.youtube.com/watch?v=oxWEVQP5_Rg

## Namespace

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: meu-site-namespace
```

Salve como `namespace.yaml`

execute o comando 

```python
kubectl apply -f namespace.yaml
```

## Service Node-port

```yaml
apiVersion: v1
kind: Service
metadata:
  name: meu-site-service
  namespace: meu-site-namespace
spec:
  selector:
    app: meu-site
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: NodePort
```

Salve como `service.yaml`

execute o comando

```yaml
kubectl apply -f service.yaml
```

## Ingress-Controller

(OBS) Seu domínio precisa esta com algum DNS setado, voce pode utilizar o Cloudflare que é gratuito

https://www.cloudflare.com/pt-br/

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: meu-site-ingress
  namespace: meu-site-namespace
spec:
  rules:
    - host: seu-dominio #Dominio precisa estar com dns
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: meu-site-service
                port:
                  number: 80
```

Salve o arquivo como `ingress.yaml`

execute o comando

```yaml
kubectl apply -f service.yaml
```

## LoadBalance

Darei dois tipos de opção para criação do loadBalance.

Caso voce ira utilizado o ip externo de um dos seu nodes faca como eu e sete ele no LoadBalance

```yaml
apiVersion: v1
kind: Service
metadata:
  name: meu-site-loadbalancer
  namespace: meu-site-namespace
spec:
  type: LoadBalancer
  externalIPs:
    - 154.53.53.239  
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  selector:
    app: meu-site
```

Caso voce possua um range de Ip, ou esteja em um ambiente cloud onde load-balance são por demandas, poderá utilizar yaml abaixo, se for cloud, o provedor adicionara para você, se for BareMetal, o MetalLB que configuramos irá seta-lo, de acordo com a lista de range que voce passou.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: meu-site-loadbalancer
  namespace: meu-site-namespace
spec:
  selector:
    app: meu-site
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
```

Após finalizar, Salve como `loadbalance.yaml`

Execute o comando abaixo

```yaml
Kubectl apply -f loadbalnce.yaml
```

## Deployment

OBS: Nessa parte é essencial que voce tenha sua imagem docker, voce pode cria-la utilizando dockerfile, segue uma documentação 

https://www.mundodocker.com.br/publicando-imagens-no-docker-hub/ 

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: meu-site-deployment
  namespace: meu-site-namespace
spec:
  replicas: 1
  selector:
    matchLabels:
      app: meu-site
  template:
    metadata:
      labels:
        app: meu-site
    spec:
      containers:
        - name: meu-site-container
          image: douglasportella/meu-portfolio:1.0 #crie sua imagem docker
          ports:
            - containerPort: 80
```

Salve o arquivo como `deployment.yaml`

execute o comando

```yaml
kubectl apply -f deployment.yaml
```

---

## Agora com tudo estando OK, podemos partir par o ultimo passo.

Vinculando um dos nodes para a mesma porta do `service/meu-site-service` que seria o nosso nodePort.

Acesse o seu Node via SSH

Onde esta a porta `Sua-porta` substitua pela porta do seu nodePort do service dentro do Namespace criado para Web.

```yaml
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port Sua-porta
```

voce pode verificar a conexão do seu laco com o nodePort utilizando o comando abaixo.

```yaml
telnet ip-interno-do-nodePort 80
```

Ou

```yaml
curl http://ip-interno-do-nodePort
```

Agora que criamos todos os Yaml e vinculamos o node ao NodePort, já podemos ver os status

```yaml
kubectl get namespace
kubectl -n meu-site-namespace get all
kubectl -n meu-site-namespace get ingress
```

Com os comandos de status acima voce vai ver algo como o exemplo abaixo

```yaml
root@vmi1274367:~# kubectl get namespace
NAME                 STATUS   AGE
default              Active   4d5h
kube-system          Active   4d5h
kube-public          Active   4d5h
kube-node-lease      Active   4d5h
metallb-system       Active   4d5h
ingress-nginx        Active   4d1h
meu-site-namespace   Active   8h
root@vmi1274367:~# kubectl -n meu-site-namespace get all
NAME                                       READY   STATUS    RESTARTS        AGE
pod/meu-site-deployment-58985ff47d-nbqph   1/1     Running   1 (6h18m ago)   7h9m

NAME                            TYPE           CLUSTER-IP      EXTERNAL-IP                    PORT(S)        AGE
service/meu-site-service        NodePort       10.43.181.212   <none>                         80:30199/TCP   8h
service/meu-site-loadbalancer   LoadBalancer   10.43.225.226   154.13.13.102   80:32544/TCP   5h51m

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/meu-site-deployment   1/1     1            1           7h9m

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/meu-site-deployment-58985ff47d   1         1         1       7h9m
root@vmi1274367:~# kubectl -n meu-site-namespace get ingress
NAME               CLASS     HOSTS                     ADDRESS   PORTS   AGE
meu-site-ingress   traefik   douglaasportella.com.br             80      8h
root@vmi1274367:~#
```

Pronto, se o status retornou algo parecido com o meu, voce já pode testar na Web externa utilizando o seu domínio que foi inserido no arquivos ingress.yaml.
