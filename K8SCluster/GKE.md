# Installing Nifi on GKE

## Created a 3 node GKE standard cluster

## Configuring kubectl

Open the cloud shell.

https://cloud.google.com/kubernetes-engine/docs/how-to/cluster-access-for-kubectl

```
gcloud container clusters get-credentials cluster-1 --zone us-central1-c
```

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

cloud_user_p_1118cebf@cloudshell:~ (playground-s-11-462b098e)$ kubectl get service
NAME                                               TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)                       AGE
kubernetes                                         ClusterIP      10.8.144.1     <none>          443/TCP                       80m
myrel-nifi                                         ClusterIP      10.8.155.171   <none>          8443/TCP,10000/TCP            15m
myrel-nifi-headless                                ClusterIP      None           <none>          8443/TCP,6007/TCP,10000/TCP   15m
myrel-zookeeper                                    ClusterIP      10.8.159.134   <none>          2181/TCP,2888/TCP,3888/TCP    15m
myrel-zookeeper-headless                           ClusterIP      None           <none>          2181/TCP,2888/TCP,3888/TCP    15m
nginx-ingress-ingress-nginx-controller             LoadBalancer   10.8.157.163   34.133.13.193   80:30508/TCP,443:31317/TCP    49m



cloud_user_p_1118cebf@cloudshell:~ (playground-s-11-462b098e)$ kubectl get pvc
NAME                                 STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
auth-conf-myrel-nifi-0               Bound    pvc-545890ec-be22-490a-bfec-3016d34b1b8a   1Gi        RWO            standard       30m
config-data-myrel-nifi-0             Bound    pvc-b19293e0-a337-4922-8415-c7a1d5f31a45   1Gi        RWO            standard       30m
content-repository-myrel-nifi-0      Bound    pvc-6b02d872-e4e6-48a4-afd6-cebc5127f912   10Gi       RWO            standard       30m
data-myrel-nifi-0                    Bound    pvc-4a4ed77b-e69b-4e5a-bd1d-07afd678a70a   1Gi        RWO            standard       30m
data-myrel-zookeeper-0               Bound    pvc-1401ec71-9cf7-4f9a-ac28-9df727dcb0a2   8Gi        RWO            standard       30m
data-myrel-zookeeper-1               Bound    pvc-2d80ba1a-8f37-4dfa-be13-9570dd6e024d   8Gi        RWO            standard       30m
data-myrel-zookeeper-2               Bound    pvc-5a29014a-8e42-4431-aa13-72735ec98ca6   8Gi        RWO            standard       30m
flowfile-repository-myrel-nifi-0     Bound    pvc-0cd18d34-f29a-462a-a82b-6ea5667c9bad   10Gi       RWO            standard       30m
logs-myrel-nifi-0                    Bound    pvc-2ffd55ae-b7b0-43ba-a660-63c55582bcff   5Gi        RWO            standard       30m
provenance-repository-myrel-nifi-0   Bound    pvc-2fd21d60-4fbe-4191-8184-40afd3202c2b   10Gi       RWO            standard       30m


cloud_user_p_1118cebf@cloudshell:~ (playground-s-11-462b098e)$ kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                        STORAGECLASS   REASON   AGE
pvc-0cd18d34-f29a-462a-a82b-6ea5667c9bad   10Gi       RWO            Delete           Bound    default/flowfile-repository-myrel-nifi-0     standard                35m
pvc-1401ec71-9cf7-4f9a-ac28-9df727dcb0a2   8Gi        RWO            Delete           Bound    default/data-myrel-zookeeper-0               standard                35m
pvc-2d80ba1a-8f37-4dfa-be13-9570dd6e024d   8Gi        RWO            Delete           Bound    default/data-myrel-zookeeper-1               standard                35m
pvc-2fd21d60-4fbe-4191-8184-40afd3202c2b   10Gi       RWO            Delete           Bound    default/provenance-repository-myrel-nifi-0   standard                35m
pvc-2ffd55ae-b7b0-43ba-a660-63c55582bcff   5Gi        RWO            Delete           Bound    default/logs-myrel-nifi-0                    standard                35m
pvc-4a4ed77b-e69b-4e5a-bd1d-07afd678a70a   1Gi        RWO            Delete           Bound    default/data-myrel-nifi-0                    standard                35m
pvc-545890ec-be22-490a-bfec-3016d34b1b8a   1Gi        RWO            Delete           Bound    default/auth-conf-myrel-nifi-0               standard                35m
pvc-5a29014a-8e42-4431-aa13-72735ec98ca6   8Gi        RWO            Delete           Bound    default/data-myrel-zookeeper-2               standard                35m
pvc-6b02d872-e4e6-48a4-afd6-cebc5127f912   10Gi       RWO            Delete           Bound    default/content-repository-myrel-nifi-0      standard                35m
pvc-b19293e0-a337-4922-8415-c7a1d5f31a45   1Gi        RWO            Delete           Bound    default/config-data-myrel-nifi-0             standard                35m
```

## Sample PV

```
apiVersion: v1
kind: PersistentVolume
metadata:
  annotations:
    pv.kubernetes.io/migrated-to: pd.csi.storage.gke.io
    pv.kubernetes.io/provisioned-by: kubernetes.io/gce-pd
  creationTimestamp: "2022-09-13T10:33:08Z"
  finalizers:
  - kubernetes.io/pv-protection
  - external-attacher/pd-csi-storage-gke-io
  labels:
    topology.kubernetes.io/region: us-central1
    topology.kubernetes.io/zone: us-central1-c
  name: pvc-545890ec-be22-490a-bfec-3016d34b1b8a
  resourceVersion: "23358"
  uid: 3354d818-0ad6-45b7-8b65-f36b093fdd04
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 1Gi
  claimRef:
    apiVersion: v1
    kind: PersistentVolumeClaim
    name: auth-conf-myrel-nifi-0
    namespace: default
    resourceVersion: "23207"
    uid: 545890ec-be22-490a-bfec-3016d34b1b8a
  gcePersistentDisk:
    fsType: ext4
    pdName: pvc-545890ec-be22-490a-bfec-3016d34b1b8a
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: topology.kubernetes.io/zone
          operator: In
          values:
          - us-central1-c
        - key: topology.kubernetes.io/region
          operator: In
          values:
          - us-central1
  persistentVolumeReclaimPolicy: Delete
  storageClassName: standard
  volumeMode: Filesystem
status:
  phase: Bound
```

## 3 node installation

```
helm uninstall myrel

helm install myrel cetic/nifi --set auth.singleUser.username=admin --set auth.singleUser.password=admin --set ingress.enabled=true --set persistence.enabled=true --set persistence.storageClass=standard --set ingress.hosts={myrel.com} --set replicaCount=3

```

```
NAME                                                      READY   STATUS    RESTARTS      AGE   IP          NODE                                       NOMINATED NODE   READINESS GATES
myrel-nifi-0                                              4/4     Running   0             79s   10.4.0.14   gke-cluster-1-default-pool-cb1adeba-wt75   <none>           1/1
myrel-nifi-1                                              4/4     Running   1 (10s ago)   79s   10.4.1.8    gke-cluster-1-default-pool-cb1adeba-jvxs   <none>           1/1
myrel-nifi-2                                              4/4     Running   1 (49s ago)   78s   10.4.2.9    gke-cluster-1-default-pool-cb1adeba-xjld   <none>           1/1
myrel-zookeeper-0                                         1/1     Running   0             80s   10.4.0.13   gke-cluster-1-default-pool-cb1adeba-wt75   <none>           <none>
myrel-zookeeper-1                                         1/1     Running   0             80s   10.4.2.8    gke-cluster-1-default-pool-cb1adeba-xjld   <none>           <none>
myrel-zookeeper-2                                         1/1     Running   0             80s   10.4.1.7    gke-cluster-1-default-pool-cb1adeba-jvxs   <none>           <none>
nginx-ingress-ingress-nginx-controller-5d88d44856-nhkrp   1/1     Running   0             74m   10.4.0.9    gke-cluster-1-default-pool-cb1adeba-wt75   <none>           <none>

cloud_user_p_1118cebf@cloudshell:~ (playground-s-11-462b098e)$ kubectl get pvc
NAME                                 STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
auth-conf-myrel-nifi-0               Bound    pvc-545890ec-be22-490a-bfec-3016d34b1b8a   1Gi        RWO            standard       51m
auth-conf-myrel-nifi-1               Bound    pvc-e78c3a9c-cba5-46af-aad1-db636a580fe1   1Gi        RWO            standard       105s
auth-conf-myrel-nifi-2               Bound    pvc-7b033cf0-7e0d-4153-9644-c158b48531f5   1Gi        RWO            standard       104s
config-data-myrel-nifi-0             Bound    pvc-b19293e0-a337-4922-8415-c7a1d5f31a45   1Gi        RWO            standard       51m
config-data-myrel-nifi-1             Bound    pvc-9c6656e4-47c7-4928-b1d4-0046253de86e   1Gi        RWO            standard       105s
config-data-myrel-nifi-2             Bound    pvc-e6ea64be-7628-4863-9c1e-8a3f2ec15cd1   1Gi        RWO            standard       104s
content-repository-myrel-nifi-0      Bound    pvc-6b02d872-e4e6-48a4-afd6-cebc5127f912   10Gi       RWO            standard       51m
content-repository-myrel-nifi-1      Bound    pvc-22d045b3-00bb-46d6-bcba-f450dd37d982   10Gi       RWO            standard       105s
content-repository-myrel-nifi-2      Bound    pvc-06486a61-64be-41fb-bf5c-f8a0208dda1a   10Gi       RWO            standard       104s
data-myrel-nifi-0                    Bound    pvc-4a4ed77b-e69b-4e5a-bd1d-07afd678a70a   1Gi        RWO            standard       51m
data-myrel-nifi-1                    Bound    pvc-35574243-f98b-4ccd-8f30-5a587475e729   1Gi        RWO            standard       105s
data-myrel-nifi-2                    Bound    pvc-8ee267c8-abac-46fd-9e05-e85fd6e00e67   1Gi        RWO            standard       104s
data-myrel-zookeeper-0               Bound    pvc-1401ec71-9cf7-4f9a-ac28-9df727dcb0a2   8Gi        RWO            standard       51m
data-myrel-zookeeper-1               Bound    pvc-2d80ba1a-8f37-4dfa-be13-9570dd6e024d   8Gi        RWO            standard       51m
data-myrel-zookeeper-2               Bound    pvc-5a29014a-8e42-4431-aa13-72735ec98ca6   8Gi        RWO            standard       51m
flowfile-repository-myrel-nifi-0     Bound    pvc-0cd18d34-f29a-462a-a82b-6ea5667c9bad   10Gi       RWO            standard       51m
flowfile-repository-myrel-nifi-1     Bound    pvc-2e8083e9-2680-41c8-b8a8-b14a7f36a5fa   10Gi       RWO            standard       105s
flowfile-repository-myrel-nifi-2     Bound    pvc-49ba08cc-c9b5-4e77-87b5-0a84093e7431   10Gi       RWO            standard       105s
logs-myrel-nifi-0                    Bound    pvc-2ffd55ae-b7b0-43ba-a660-63c55582bcff   5Gi        RWO            standard       51m
logs-myrel-nifi-1                    Bound    pvc-5a3da55f-844e-4550-a14a-e30d73742082   5Gi        RWO            standard       105s
logs-myrel-nifi-2                    Bound    pvc-2977d3df-6fd8-4fe2-9414-e013c1ca27ce   5Gi        RWO            standard       104s
provenance-repository-myrel-nifi-0   Bound    pvc-2fd21d60-4fbe-4191-8184-40afd3202c2b   10Gi       RWO            standard       51m
provenance-repository-myrel-nifi-1   Bound    pvc-c2165080-8067-494f-9937-9c28524a47ff   10Gi       RWO            standard       105s
provenance-repository-myrel-nifi-2   Bound    pvc-75bc31ed-8f6c-4ef8-b3fd-edf22099258f   10Gi       RWO            standard       104s
cloud_user_p_1118cebf@cloudshell:~ (playground-s-11-462b098e)$ kubectl get pv

NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                        STORAGECLASS   REASON   AGE
pvc-06486a61-64be-41fb-bf5c-f8a0208dda1a   10Gi       RWO            Delete           Bound    default/content-repository-myrel-nifi-2      standard                109s
pvc-0cd18d34-f29a-462a-a82b-6ea5667c9bad   10Gi       RWO            Delete           Bound    default/flowfile-repository-myrel-nifi-0     standard                52m
pvc-1401ec71-9cf7-4f9a-ac28-9df727dcb0a2   8Gi        RWO            Delete           Bound    default/data-myrel-zookeeper-0               standard                52m
pvc-22d045b3-00bb-46d6-bcba-f450dd37d982   10Gi       RWO            Delete           Bound    default/content-repository-myrel-nifi-1      standard                111s
pvc-2977d3df-6fd8-4fe2-9414-e013c1ca27ce   5Gi        RWO            Delete           Bound    default/logs-myrel-nifi-2                    standard                111s
pvc-2d80ba1a-8f37-4dfa-be13-9570dd6e024d   8Gi        RWO            Delete           Bound    default/data-myrel-zookeeper-1               standard                52m
pvc-2e8083e9-2680-41c8-b8a8-b14a7f36a5fa   10Gi       RWO            Delete           Bound    default/flowfile-repository-myrel-nifi-1     standard                111s
pvc-2fd21d60-4fbe-4191-8184-40afd3202c2b   10Gi       RWO            Delete           Bound    default/provenance-repository-myrel-nifi-0   standard                52m
pvc-2ffd55ae-b7b0-43ba-a660-63c55582bcff   5Gi        RWO            Delete           Bound    default/logs-myrel-nifi-0                    standard                52m
pvc-35574243-f98b-4ccd-8f30-5a587475e729   1Gi        RWO            Delete           Bound    default/data-myrel-nifi-1                    standard                110s
pvc-49ba08cc-c9b5-4e77-87b5-0a84093e7431   10Gi       RWO            Delete           Bound    default/flowfile-repository-myrel-nifi-2     standard                111s
pvc-4a4ed77b-e69b-4e5a-bd1d-07afd678a70a   1Gi        RWO            Delete           Bound    default/data-myrel-nifi-0                    standard                52m
pvc-545890ec-be22-490a-bfec-3016d34b1b8a   1Gi        RWO            Delete           Bound    default/auth-conf-myrel-nifi-0               standard                52m
pvc-5a29014a-8e42-4431-aa13-72735ec98ca6   8Gi        RWO            Delete           Bound    default/data-myrel-zookeeper-2               standard                52m
pvc-5a3da55f-844e-4550-a14a-e30d73742082   5Gi        RWO            Delete           Bound    default/logs-myrel-nifi-1                    standard                110s
pvc-6b02d872-e4e6-48a4-afd6-cebc5127f912   10Gi       RWO            Delete           Bound    default/content-repository-myrel-nifi-0      standard                52m
pvc-75bc31ed-8f6c-4ef8-b3fd-edf22099258f   10Gi       RWO            Delete           Bound    default/provenance-repository-myrel-nifi-2   standard                111s
pvc-7b033cf0-7e0d-4153-9644-c158b48531f5   1Gi        RWO            Delete           Bound    default/auth-conf-myrel-nifi-2               standard                110s
pvc-8ee267c8-abac-46fd-9e05-e85fd6e00e67   1Gi        RWO            Delete           Bound    default/data-myrel-nifi-2                    standard                109s
pvc-9c6656e4-47c7-4928-b1d4-0046253de86e   1Gi        RWO            Delete           Bound    default/config-data-myrel-nifi-1             standard                111s
pvc-b19293e0-a337-4922-8415-c7a1d5f31a45   1Gi        RWO            Delete           Bound    default/config-data-myrel-nifi-0             standard                52m
pvc-c2165080-8067-494f-9937-9c28524a47ff   10Gi       RWO            Delete           Bound    default/provenance-repository-myrel-nifi-1   standard                110s
pvc-e6ea64be-7628-4863-9c1e-8a3f2ec15cd1   1Gi        RWO            Delete           Bound    default/config-data-myrel-nifi-2             standard                109s
pvc-e78c3a9c-cba5-46af-aad1-db636a580fe1   1Gi        RWO            Delete           Bound    default/auth-conf-myrel-nifi-1               standard                110s
```

```
cloud_user_p_1118cebf@cloudshell:~ (playground-s-11-462b098e)$ kubectl get service
NAME                                               TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)                       AGE
kubernetes                                         ClusterIP      10.8.144.1     <none>          443/TCP                       107m
myrel-nifi                                         ClusterIP      10.8.149.91    <none>          8443/TCP,10000/TCP            3m1s
myrel-nifi-headless                                ClusterIP      None           <none>          8443/TCP,6007/TCP,10000/TCP   3m2s
myrel-zookeeper                                    ClusterIP      10.8.152.220   <none>          2181/TCP,2888/TCP,3888/TCP    3m2s
myrel-zookeeper-headless                           ClusterIP      None           <none>          2181/TCP,2888/TCP,3888/TCP    3m3s
nginx-ingress-ingress-nginx-controller             LoadBalancer   10.8.157.163   34.133.13.193   80:30508/TCP,443:31317/TCP    76m
nginx-ingress-ingress-nginx-controller-admission   ClusterIP      10.8.153.131   <none>          443/TCP                       76m

```

Somehow it did not create ingress.
Need to check why.

```
cloud_user_p_1118cebf@cloudshell:~ (playground-s-11-462b098e)$ kubectl get ingress
No resources found in default namespace.
```

```
To access the NiFi UI via kubectl port forwarding,
wait until the cluster is ready and then run:

kubectl port-forward -n default svc/myrel 8443:8443

...and point your web browser to https://localhost:8443/nifi/
```

I gave this
kubectl port-forward -n default svc/myrel-nifi 8443:8443

## Another instllation in a differnet namespace

```
helm install somerel cetic/nifi --set auth.singleUser.username=admin --set auth.singleUser.password=admin --set ingress.enabled=true --set persistence.enabled=true --set persistence.storageClass=standard --set ingress.hosts={somerel.com} --set replicaCount=3 --namespace nifi
```

Started 2 pods, but third failed with

```

  Type     Reason                   Age                    From                     Message
  ----     ------                   ----                   ----                     -------
  Normal   LoadBalancerNegNotReady  2m14s                  neg-readiness-reflector  Waiting for pod to become healthy in at least one of the NEG(s): [k8s1-a853b1d6-nifi-somerel-nifi-8443-d218e2b1]
  Normal   NotTriggerScaleUp        2m13s                  cluster-autoscaler       pod didn't trigger scale-up:
  Warning  FailedScheduling         2m11s (x3 over 2m14s)  default-scheduler        0/3 nodes are available: 3 pod has unbound immediate PersistentVolumeClaims.
  Warning  FailedScheduling         57s (x2 over 2m6s)     default-scheduler        0/3 nodes are available: 1 Insufficient cpu, 2 node(s) exceed max volume count
```

Resource usages

Refer the [./resourceUsage.png]
