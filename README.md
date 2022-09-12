# nifi-on-k8s

Scratch pad for Nifi on K8S

## Steps

1. Create a K8S cluster 3 node cluster
2. Ensure there are disk mounts for Nifi Repositories
3. Create two namespaces since we are trying to emulate a multi-tenant Nifi such that each tenant gets their own namespace
4. Create service accounts, PSP. PodDisruptionBudget,Namespace resource quotas
5. Set up ingress-ngnix on the K8S cluster
6. We want to install Nifi 1.16.0 - create deployment yamls for the same
7. Create deployment yamls for the zookeeper, version 3.6.2
8. Set up certificates among the K8S cluster nodes. Better to keep master as separate

## Installation

Let me try to install via Helm
https://artifacthub.io/packages/helm/cetic/nifi

And then I can get the deployment yamls reverse engineered.

username : cloud_user_p_0e071f7e@linuxacademygclabs.com
password : XEowl4ZG
URL : https://accounts.google.com/AddSession?service=accountsettings&sarp=1&continue=https%3A%2F%2Fconsole.cloud.google.com%2Fhome%2Fdashboard%3Fproject%3Dplayground-s-11-22a827f5#Email=cloud_user_p_0e071f7e@linuxacademygclabs.com
