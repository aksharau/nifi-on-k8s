# nifi-on-k8s

Scratch pad for Nifi on K8S

The Deployment folder contains the kubernetes manifest files to have a 3 node Nifi set up.
The installation was tried on GKE cluster, not all PV PVCs are captured.

The PV folder contains POC files for setting up host path PV such that two separate namespaces can mount to two different directories
