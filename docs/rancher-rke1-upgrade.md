
# Rancher and RKE1 Upgrades
_Update 2024-07-23_

## Backup
```
rke etcd snapshot-save --config cluster.yml --name mysnapshot
...
INFO[0003] Finished saving/uploading snapshot [init] on all etcd hosts
```
```
ls -lh /opt/rke/etcd-snapshots/
```

## Restore
```
rke etcd snapshot-restore --config cluster.yml --name mysnapshot
```

## Upgrade RKE command
```
rke config --list-version --all
WARN[0000] This is not an officially supported version (v1.4.20-rc3) of RKE. Please download the latest official release at https://github.com/rancher/rke/releases
```
```
whereis rke
```
```
rm rke; mv rke_linux-amd64 rke_linux-amd64.bkp
```
```
wget https://github.com/rancher/rke/releases/download/v1.5.11/rke_linux-amd64; chmod +x rke_linux-amd64; ln -s rke_linux-amd64 rke
```
```
rke -version
```

## Upgrade Rancher with Debug
```
export RKE_HOSTNAME=rancher2.myrancher.lab
```
```
export RKE_ADMIN_PASSWORD=YOUR_PASSWORD
```
```
helm upgrade rancher rancher-stable/rancher --version 2.8.5 --namespace=cattle-system --set hostname=${RKE_HOSTNAME} --set bootstrapPassword=${RKE_ADMIN_PASSWORD} --set auditLog.level=3
```

## Upgrade Kubernetes Downstream Cluster
1. Go to Cluster Management Menu;
2. Identify the Downstream Cluster and click on "Edit Config";
3. In "Kubernetes Version", select the version above and click "Save";
4. Waiting for update process.
