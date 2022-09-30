# Functional tests for Nifi on K8S

## Pre-requisite information

### MySQL installation

https://artifacthub.io/packages/helm/bitnami/mysql

```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install mysql bitnami/mysql
```

Connecting to it

```
$ kubectl get pods -n default
NAME                                                      READY   STATUS    RESTARTS   AGE
mysql-0                                                   1/1     Running   0          6m28s



Tip:

  Watch the deployment status using the command: kubectl get pods -w --namespace default

Services:

  echo Primary: mysql.default.svc.cluster.local:3306

Execute the following to get the administrator credentials:

  echo Username: root
  MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace default mysql -o jsonpath="{.data.mysql-root-password}" | base64 -d)

To connect to your database:

  1. Run a pod that you can use as a client:

      kubectl run mysql-client --rm --tty -i --restart='Never' --image  docker.io/bitnami/mysql:8.0.30-debian-11-r15 --namespace default --env MYSQL_ROOT_PASSWORD=$MYSQL_ROOT_PASSWORD --command -- bash

  2. To connect to primary service (read/write):
    mysql -h mysql.default.svc.cluster.local -uroot -p"$MYSQL_ROOT_PASSWORD"

```

### Hive installation

https://artifacthub.io/packages/helm/gradiant/hdfs

```
helm repo add bigdata-gradiant https://gradiant.github.io/bigdata-charts/

 helm install hdfs --set persistence.nameNode.enabled=true \
  --set persistence.nameNode.storageClass=standard \
  --set persistence.dataNode.enabled=true \
  --set persistence.dataNode.storageClass=standard \
 bigdata-gradiant/hdfs

```

Connecting

```
$ kubectl get pods -n default
NAME                                                      READY   STATUS    RESTARTS   AGE
hdfs-datanode-0                                           1/1     Running   0          95s
hdfs-datanode-1                                           1/1     Running   0          51s
hdfs-datanode-2                                           0/1     Pending   0          7s
hdfs-httpfs-59c989db6b-xf7kk                              1/1     Running   0          95s
hdfs-namenode-0                                           2/2     Running   0          95s


NOTES:
1. You can check the status of HDFS by running this command:
   kubectl exec -n default -it hdfs-namenode-0 -- hdfs dfsadmin -report

2. Create a port-forward to the hdfs manager UI:
   kubectl port-forward -n default hdfs-namenode-0 50070:50070

   Then open the ui in your browser:

   open http://localhost:50070
```

https://artifacthub.io/packages/helm/gradiant/hive-metastore

```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install my-release bitnami/postgresql

helm install hive-metastore  bigdata-gradiant/hive-metastore
```

Connecting to it

```
$ kubectl get pods -n default
NAME                                                      READY   STATUS    RESTARTS   AGE
my-release-postgresql-0                                   1/1     Running   0          37s

PostgreSQL can be accessed via port 5432 on the following DNS names from within your cluster:

    my-release-postgresql.default.svc.cluster.local - Read/Write connection

To get the password for "postgres" run:

    export POSTGRES_PASSWORD=$(kubectl get secret --namespace default my-release-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)

To connect to your database run the following command:

    kubectl run my-release-postgresql-client --rm --tty -i --restart='Never' --namespace default --image docker.io/bitnami/postgresql:14.5.0-debian-11-r14 --env="PGPASSWORD=$POSTGRES_PASSWORD" \
      --command -- psql --host my-release-postgresql -U postgres -d postgres -p 5432

    > NOTE: If you access the container using bash, make sure that you execute "/opt/bitnami/scripts/postgresql/entrypoint.sh /bin/bash" in order to avoid the error "psql: local user with ID 1001} does not exist"

To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace default svc/my-release-postgresql 5432:5432 &
    PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432
######################################

$ kubectl get pods -n default
NAME                                                      READY   STATUS    RESTARTS      AGE
hive-metastore-0                                          1/1     Running   2 (18s ago)   73s
hive-metastore-postgresql-0                               1/1     Running   0             73s

```

https://artifacthub.io/packages/helm/gradiant/hive

```
helm repo add bigdata-gradiant https://gradiant.github.io/bigdata-charts/

helm install hive bigdata-gradiant/hive
```

Got this error

```
Error: INSTALLATION FAILED: rendered manifests contain a resource that already exists. Unable to continue with install: ConfigMap "hive-metastore" in namespace "default" exists and cannot be imported into the current release: invalid ownership metadata; annotation validation error: key "meta.helm.sh/release-name" must equal "hive": current value is "hive-metastore"
```

So deleted all the deployments and only gave

```

helm install hive bigdata-gradiant/hive
```

And it worked

```
$ kubectl get pods -n default
NAME                                                      READY   STATUS    RESTARTS       AGE
hive-hdfs-datanode-0                                      1/1     Running   0              111s
hive-hdfs-datanode-1                                      1/1     Running   0              89s
hive-hdfs-datanode-2                                      1/1     Running   0              79s
hive-hdfs-httpfs-59bc69476b-vhhtl                         1/1     Running   0              111s
hive-hdfs-namenode-0                                      2/2     Running   1 (104s ago)   111s
hive-metastore-0                                          1/1     Running   0              111s
hive-postgresql-0                                         1/1     Running   0              111s
hive-server-0                                             1/1     Running   1 (20s ago)    111s
mysql-0                                                   1/1     Running   0              22m
nginx-ingress-ingress-nginx-controller-5d88d44856-99vsr   1/1     Running   0              25m
```

## Connecting to Nifi

Had to add this annotation to the ingress-controller pod

```
kubernetes.io/ingress.class: nginx
```

Else it was not working.The ingress-controller logs had error stating that this above annotation was missing.

There are following errors in the kube-dns logs

```
I0929 09:10:00.914814       1 dns.go:616] Could not find endpoints for service "myrel-zookeeper-headless" in namespace "default". DNS records will be created once endpoints show up.
I0929 09:10:00.968139       1 dns.go:616] Could not find endpoints for service "myrel-nifi-headless" in namespace "default". DNS records will be created once endpoints show up.
```

Looks like it can be ignored since the headless service was there.

### Port forwarding

```
kubectl port-forward -n default svc/myrel-nifi 8443:8443

https://ssh.cloud.google.com/devshell/proxy?port=8443

http://ssh.cloud.google.com/devshell/proxy?port=8443&_ga=2.34776609.-1421203454.1664441753

https://shell.cloud.google.com/devshell/proxy?port=8443&hl=en_US&fromcloudshell=true

http://ssh.cloud.google.com/devshell/cs-256205935429-default/proxy?port=8443

```

### Ingress

Lets try this
https://cloud.google.com/community/tutorials/nginx-ingress-gke

No luck

### POD to LB

From the GKE cluster I went to the pod and exposed it as a service.
Then I get following error

```
System Error
The request contained an invalid host header [35.188.180.131:80] in the request [/]. Check for request manipulation or third-party intercept.
Valid host headers are [empty] or:
127.0.0.1
127.0.0.1:8443
localhost
localhost:8443
[::1]
[::1]:8443
myrel-nifi-0.myrel-nifi-headless.default.svc.cluster.local
myrel-nifi-0.myrel-nifi-headless.default.svc.cluster.local:8443
10.4.0.7
10.4.0.7:8443
myrel-nifi.default.svc
```

Seems to be the issue with https://github.com/cetic/helm-nifi/issues/72

Lets try this
https://github.com/cetic/helm-nifi/issues/192

```
helm install myrel cetic/nifi --set auth.singleUser.username=admin --set auth.singleUser.password=admin --set ingress.enabled=true --set persistence.enabled=true --set persistence.storageClass=standard --set ingress.hosts={34.173.3.185.nip.io} --set properties.webProxyHost=34.173.3.185.nip.io
```

The above works.
Need to get the ingress service external IP and set it here

Give below annotations in the ingress

```
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/ssl-passthrough: "true"
    nginx.ingress.kubernetes.io/backend-protocol: HTTPS
```

Not sure if this is related, but I also enabled TCP load balancing: https://cloud.google.com/kubernetes-engine/docs/how-to/internal-load-balancing
And disabled all possible firewalls.

The user and password is generated - check it from logs.

But then got this issue
https://community.cloudera.com/t5/Support-Questions/NiFi-single-user-Certificate-and-Token-not-found/td-p/345007

Will check it on Monday. \o/

## Success Scenarios

1. Login to the UI, create a sample process group for Hive to MySQL transfer

## Failure Scenarios

1. One of the node from the Nifi cluster is restarted
2. One node is fully removed from the Nifi cluster
3. Nifi all nodes are restarted one by one
4. Nifi server is restarted on a given node
