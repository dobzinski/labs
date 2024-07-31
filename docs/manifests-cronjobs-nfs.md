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

## Using Config Map and CronJob do mont NFS
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: nfs-mount-script
  namespace: default
data:
  mount-nfs.sh: |
#!/bin/bash

# Variáveis
NFS_DIR="/opt/nfs1"
EXPORTS_FILE="/etc/exports"
EXPORTS_ENTRY="$NFS_DIR 192.168.100.XX(rw,async,no_subtree_check,no_root_squash)"
LOG_FILE="/var/log/nfs_mount.log"

# Função para log
log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" >> $LOG_FILE
}

# Criação do diretório NFS
if mkdir -p $NFS_DIR; then
    log "Diretório $NFS_DIR criado com sucesso."
else
    log "Erro ao criar o diretório $NFS_DIR."
    exit 1
fi

# Verificar se a entrada já existe em /etc/exports
if grep -q "$EXPORTS_ENTRY" $EXPORTS_FILE; then
    log "A entrada NFS já existe em $EXPORTS_FILE."
else
    # Adiciona a entrada ao /etc/exports
    if echo "$EXPORTS_ENTRY" >> $EXPORTS_FILE; then
        log "Entrada '$EXPORTS_ENTRY' adicionada ao $EXPORTS_FILE."
    else
        log "Erro ao adicionar a entrada ao $EXPORTS_FILE."
        exit 1
    fi
fi

# Atualiza as exportações NFS
if exportfs -va; then
    log "Exportações NFS atualizadas com sucesso."
else
    log "Erro ao atualizar as exportações NFS."
    exit 1
fi

log "Script concluído com sucesso."

```
kubectl apply -f configmap.yaml

## Job ou DaemonSet:
## Job:
```
apiVersion: batch/v1
kind: Job
metadata:
  name: nfs-mount-job
  namespace: default
spec:
  template:
    spec:
      containers:
      - name: nfs-mount
        image: alpine
        command: ["/bin/sh", "-c", "/scripts/mount-nfs.sh"]
        volumeMounts:
        - name: script
          mountPath: /scripts
      restartPolicy: OnFailure
      volumes:
      - name: script
        configMap:
          name: nfs-mount-script
          items:
          - key: mount-nfs.sh
            path: mount-nfs.sh
```
## DaemonSet:
```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nfs-mount-daemonset
  namespace: default
spec:
  selector:
    matchLabels:
      name: nfs-mount
  template:
    metadata:
      labels:
        name: nfs-mount
    spec:
      containers:
      - name: nfs-mount
        image: alpine
        command: ["/bin/sh", "-c", "/scripts/mount-nfs.sh"]
        volumeMounts:
        - name: script
          mountPath: /scripts
      volumes:
      - name: script
        configMap:
          name: nfs-mount-script
          items:
          - key: mount-nfs.sh
            path: mount-nfs.sh
      hostNetwork: true
      dnsPolicy: ClusterFirstWithHostNet
```
## run Job ou DaemonSet:

kubectl apply -f job-or-daemonset.yaml

## Manifest for NFS - PV - Persistent Volume
## Remoção de status: O campo status foi removido, pois é gerenciado pelo Kubernetes.
## Adição de labels e annotations: Adicionadas para facilitar o gerenciamento e fornecer uma descrição clara do propósito do PV.
## Manutenção de persistentVolumeReclaimPolicy: Adicionado como Retain para manter o volume mesmo após o PVC ser deletado. Pode ser ajustado conforme as necessidades (por exemplo, Recycle ou Delete).
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs1
  labels:
    app: my-backup
  annotations:
    description: "PV for NFS storage"
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  nfs:
    path: /opt/nfs1
    server: 192.168.100.100
  volumeMode: Filesystem
  persistentVolumeReclaimPolicy: Retain
```

## Manifest for NFS - PVC - Persistent Volume Claim
## Remoção de status: O campo status é gerenciado pelo Kubernetes e é removido do manifesto.
## Adição de labels e annotations: Usados para fornecer mais contexto sobre o PVC, facilitando o gerenciamento.
## Manutenção de volumeName: Se o PVC está sendo vinculado a um volume específico, volumeName é mantido. Se o vínculo com um volume específico não for necessário, volumeName pode ser removido, e o Kubernetes escolherá automaticamente um PersistentVolume (PV) adequado.
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-nfs1
  namespace: prod
  labels:
    app: my-backup
  annotations:
    description: "PVC for NFS storage"
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  volumeMode: Filesystem
  volumeName: pv-nfs1
```

## Manifest for CronJobs
## Especificar resource requests e limits: Isso ajuda na alocação de recursos e evita que o contêiner consuma mais do que deveria.
## Definir backoffLimit: Limita o número de vezes que o Kubernetes tenta reexecutar um job falhado.
## Definir serviceAccountName: Assegura que o job seja executado com as permissões mínimas necessárias.

```
apiVersion: batch/v1
kind: CronJob
metadata:
  name: job1
  namespace: prod
  labels:
    app: job1
  annotations:
    description: "This job runs a script every minute using a BusyBox container."
spec:
  schedule: '*/1 * * * *'
  concurrencyPolicy: Allow
  failedJobsHistoryLimit: 1
  successfulJobsHistoryLimit: 3
  suspend: false
  jobTemplate:
    spec:
      template:
        metadata:
          labels:
            app: job1
        spec:
          containers:
            - name: busybox1
              image: busybox
              imagePullPolicy: Always
              command:
                - /mnt/run.sh
              resources:
                requests:
                  cpu: "100m"
                  memory: "128Mi"
                limits:
                  cpu: "200m"
                  memory: "256Mi"
              volumeMounts:
                - mountPath: /mnt
                  name: vol-nfs1
          restartPolicy: Never
          dnsPolicy: ClusterFirst
          schedulerName: default-scheduler
          terminationGracePeriodSeconds: 30
          volumes:
            - name: vol-nfs1
              persistentVolumeClaim:
                claimName: pvc-nfs1
          serviceAccountName: default
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
