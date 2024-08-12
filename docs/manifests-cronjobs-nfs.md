# Lab for create CronJob to run a Shell Script in NFS share

Using the manifests to create this Lab

_2024-07-30_

## Login with SSH to NFS Server (192.168.100.100)
Create the NFS Server to will share folder to your Downstream Cluster (repeat "echo" for all IPs Workers Nodes)
```
mkdir /opt/nfs1
echo "/opt/nfs1 192.168.100.XX(rw,async,no_subtree_check,no_root_squash)" >> /etc/exports
exportfs -va
```

Create the Shell Script
```
touch /opt/nfs1/run.sh
chmod +x /opt/nfs1/run.sh
```

Copy/Paste in run.sh
```
#!/bin/sh
echo `date` >> /mnt/run.log
```

## Manifest for NFS Persistent Volume
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs1
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 10Gi
  nfs:
    path: /opt/nfs1
    server: 192.168.100.100
  volumeMode: Filesystem
status:
  phase: Available
```

## Manifest for NFS Persistent Volume Claim
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-nfs1
  namespace: prod
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  volumeMode: Filesystem
  volumeName: pv-nfs1
status:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 10Gi
  phase: Bound
```

## Manifest for CronJobs
```
apiVersion: batch/v1
kind: CronJob
metadata:
  name: job1
  namespace: prod
spec:
  concurrencyPolicy: Forbid
  failedJobsHistoryLimit: 1
  jobTemplate:
    metadata:
      namespace: prod
    spec:
      template:
        spec:
          containers:
            - command:
                - /mnt/run.sh
              image: busybox
              imagePullPolicy: Always
              name: busybox1
              terminationMessagePath: /dev/termination-log
              terminationMessagePolicy: File
              volumeMounts:
                - mountPath: /mnt
                  name: vol-nfs1
          dnsPolicy: ClusterFirst
          restartPolicy: Never
          schedulerName: default-scheduler
          terminationGracePeriodSeconds: 30
          volumes:
            - name: vol-nfs1
              persistentVolumeClaim:
                claimName: pvc-nfs1
  timeZone: "America/Sao_Paulo"
  schedule: '*/1 * * * *'
  successfulJobsHistoryLimit: 3
  suspend: false
```

## After create those three Yaml files in new folder, apply to your Downstream Cluster
```
kubectl apply -f .
```

## Check
After 5 minutes, back to the NFS Server and verify lines in run.log
```
cat /opt/nfs1/run.log
```
