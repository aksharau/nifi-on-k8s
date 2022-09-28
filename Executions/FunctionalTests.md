# Functional tests for Nifi on K8S

## Success Scenarios

1. Login to the UI, create a sample process group for Hive to MySQL transfer

### MySQL installation

https://artifacthub.io/packages/helm/bitnami/mysql


### Hive installation

https://artifacthub.io/packages/helm/gradiant/hive#requirements



## Failure Scenarios

1. One of the node from the Nifi cluster is restarted
2. One node is fully removed from the Nifi cluster
3. Nifi all nodes are restarted one by one
4. Nifi server is restarted on a given node