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

## Success Scenarios

1. Login to the UI, create a sample process group for Hive to MySQL transfer

## Failure Scenarios

1. One of the node from the Nifi cluster is restarted
2. One node is fully removed from the Nifi cluster
3. Nifi all nodes are restarted one by one
4. Nifi server is restarted on a given node
