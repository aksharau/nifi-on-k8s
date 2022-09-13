# Installing Nifi on GKE

## Created a 3 node GKE standard cluster

## Ignix Controller installation

```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install nginx-ingress ingress-nginx/ingress-nginx


kubectl get deployment nginx-ingress-ingress-nginx-controller
kubectl get service nginx-ingress-ingress-nginx-controller
```

Ingress controller gets installed in the default namespace

## Installing Nifi

```
helm repo add cetic https://cetic.github.io/helm-charts
helm repo update


helm install myrel cetic/nifi --set auth.singleUser.username=admin --set auth.singleUser.password=admin --set ingress.enabled=true --set persistence.enabled=true --set persistence.storageClass=standard --set ingress.hosts={myrel.com}
```

Started Running

```
cloud_user_p_1118cebf@cloudshell:~ (playground-s-11-462b098e)$ kubectl get pods
NAME                                                      READY   STATUS    RESTARTS   AGE
myrel-nifi-0                                              4/4     Running   0          86s
myrel-zookeeper-0                                         1/1     Running   0          86s
myrel-zookeeper-1                                         1/1     Running   0          86s
myrel-zookeeper-2                                         1/1     Running   0          86s
nginx-ingress-ingress-nginx-controller-5d88d44856-nhkrp   1/1     Running   0          35m
```
