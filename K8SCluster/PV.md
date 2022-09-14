## Testing PV and PVC

When the PV and PVC got created for Nifi, the PV was very GKE specific.
Will not be able to use it when deploying on prod clusters.

The kind of PV and PVCs we will need are thus:

The worker node will have a disk, or multiple.
We need PVs for different directories.

So I'll create the PVs like so:

```
/nifi/repo/content/teamA
/nifi/repo/content/teamB
/nifi/repo/flow/teamA
/nifi/repo/flow/teamB
```

Can follow this guide and install pvs

https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/

Can use a buysbox image to mount the volumnes.

Or this

https://kubernetes.io/docs/tutorials/stateful-application/basic-stateful-set/

So we will create 2 stateful sets - each with 2 replica count
One in each namespace: teamA and teamB

Per statefulset:
2 PODs, with 2 mounts each -
so we need 4 PV and 4 PVC for teamA

Similar we need 4PV and 4PVC for teamB

## GKE Cluster

The nodes are

```
gke-cluster-1-default-pool-e82b3219-n1d2	 Ready	483mCPU	940 mCPU	429.92 MB	2.95 GB	0 B	0 B
gke-cluster-1-default-pool-e82b3219-q2p6	 Ready	541mCPU	940 mCPU	560.99 MB	2.95 GB	0 B	0 B
gke-cluster-1-default-pool-e82b3219-qffh	 Ready	253mCPU	940 mCPU	335.06 MB	2.95 GB
```

When only giving for teama pods, the error was

```
Events:
  Type     Reason       Age               From               Message
  ----     ------       ----              ----               -------
  Normal   Scheduled    73s               default-scheduler  Successfully assigned teama/nifi-0 to gke-cluster-1-default-pool-e82b3219-n1d2
  Warning  FailedMount  9s (x8 over 72s)  kubelet            MountVolume.NewMounter initialization failed for volume "flow-0" : path "/nifi/repo/flow/teamA" does not exist
  Warning  FailedMount  9s (x8 over 72s)  kubelet            MountVolume.NewMounter initialization failed for volume "content-0" : path "/nifi/repo/content/teamA" does not exist
```

I went to Compute engine and then logged in to the hosts and created directories in /mnt/stateful_partition

After that two replica got spawned on the nodes as needed.
\o/
