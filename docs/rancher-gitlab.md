# Install GitLab on Rancher Using Deployment

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
    - Path on the Node: /disk1/git/etc
    - Click Create button
  - Name: pv-gitlab-opt
    - Volume Plugin: HostPath
    - Capacity 10GiB
    - Path on the Node: /disk1/git/opt
    - Click Create button
  - Name: pv-gitlab-log
    - Volume Plugin: HostPath
    - Capacity 10GiB
    - Path on the Node: /disk1/git/log
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

11 - Click the Create button and inform:
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
       - Networking:
         - Service Type: Cluster IP
         - Name: http
         - Private Container: 80
         - Protocol: TCP

12 - Click the Create button, and wait!

13 - Go to Service Discovery / Ingress

14 - Click the Create button and inform:
   - Name: gitlab-http
   - Request Host: gitlab.example.com
   - Path
     -  Prefix: /
     -  Target Service: gitlab
     -  Port: 80
   - Click Create button

15 - Open your Web Browser and insert link http://gitlab.example.com, you will see the Sign in form.

16 - Back to Rancher, and go to Workloads / Pods, and Click on Execute Shell for gitlab Pod.

17 - Run the command for reset password to the root user (minimum is 8 characters):
```
gitlab-rake "gitlab:password:reset[root]"
```
