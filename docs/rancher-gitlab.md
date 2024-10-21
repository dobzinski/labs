# Install GitLab on Rancher

## Steps

1 - Open the Downstream Cluster

2 - Go to Cluster / Projects/Namespaces

3 - Click on Create Project button

4 - Inform:
  - Name: Infrastructure
  - Click Create button

5 - On Infrastructure tab, click Create Namespace button
  - Name: infra
  - Click Create button

6 - Go to Storage / PersistentVolumes

7 - Gitlab it's need the three PVs, repeat three times to click on the Create button:
  - Name: pv-gitlab-etc
    - Volume Plugin: HostPath
    - Capacity 1GiB
    - Path on the Node: /disk1/github/etc
    - Click Create button
  - Name: pv-gitlab-opt
    - Volume Plugin: HostPath
    - Capacity 10GiB
    - Path on the Node: /disk1/github/opt
    - Click Create button
  - Name: pv-gitlab-log
    - Volume Plugin: HostPath
    - Capacity 10GiB
    - Path on the Node: /disk1/github/log
    - Click Create button

8 - Go to Storage / PersistentVolumeClaims

9 - Repeat three times for create the PVCs, then click Create button:
  - Name: pvc-gitlab-etc
    - Namespace: infra
    - Source: Mark the "Use an existing Persistent Volume"
    - Select in Persistent Volume: pv-gitlab-etc
    - Click Create button
  - Name: pvc-gitlab-opt
    - Namespace: infra
    - Source: Mark the "Use an existing Persistent Volume"
    - Select in Persistent Volume: pv-gitlab-opt
    - Click Create button
  - Name: pvc-gitlab-log
    - Namespace: infra
    - Source: Mark the "Use an existing Persistent Volume"
    - Select in Persistent Volume: pv-gitlab-log
    - Click Create button

10 - Go to Workloads / Deployments

11 - Click in Create button and inform:
   - Name: gitlab
   - Namespace: infra
   - Image: gitlab/gitlab-ce:latest
   - Change to Pod tab, Go to Storage:
     - Add Volume : Persistent Volume Claim
       - Volume Name: vol-gitlab-etc
       - Persistent Volume Claim: pvc-gitlab-etc
     - Add Volume : Persistent Volume Claim
       - Volume Name: vol-gitlab-opt
       - Persistent Volume Claim: pvc-gitlab-opt
     - Add Volume : Persistent Volume Claim
       - Volume Name: vol-gitlab-log
       - Persistent Volume Claim: pvc-gitlab-log
     - Go to Networking, and inform the Hostname: git
     - Back to container tab, Go to Storage:
       - Select Volume : vol-gitlab-etc
         - Mount Point: /etc/gitlab
       - Select Volume : vol-gitlab-opt
         - Mount Point: /var/opt/gitlab
       - Select Volume : vol-gitlab-log
         - Mount Point: /var/log/gitlab

12 - Click Create button, and wait!


