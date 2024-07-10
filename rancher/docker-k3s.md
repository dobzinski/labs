# docker-rancher
Steps for configuring lab Rancher (K3s) using Docker

$\color{red}{\textsf{VERY IMPORTANT! DO NOT USE FOR PRODUCTION!}}$

Topology
===
   - 192.168.0.9  - Infra: DNS (2gb/1vcpu)
   - 192.168.0.10 - Docker
   - 192.168.0.11 - Node1: Server Etcd + Control Plane (8gb/4vcpu)
   - 192.168.0.12 - Node2: Server Worker (8gb/4vcpu)

_* Note: You can install Docker localy enviroment (Linux/Windows) and create Virtual Machines with openSUSE Linux to Infra, Node1 and Node2._

DNS Server
===

1. Install DNS service
```
zypper install bind
```

2. Edit the named.conf file.
```
vi /etc/named.conf
```

3. Enable the forwarders to Google (uncomment the line and add IPs address).
```
forwarders { 8.8.8.8; 8.8.4.4; };
```

4. Configure the new zone "myrancher.lab" (add those lines to the end).
```
zone "myrancher.lab" in {
        type master;
        file "myrancher.lab.zone";
        allow-update { none; };
        allow-query { any; };
};
```

5. Edit a new zone file.
```
vi /var/lib/named/myrancher.lab.zone
```

6. Add those lines to myrancher.lab.zone file.
```
$TTL 86400
@   IN  SOA     ns.myrancher.lab root.myrancher.lab. (
        2024071001      ;Serial
        3600            ;Refresh
        1800            ;Retry
        604800          ;Expire
        86400           ;Minimum TTL
)
         IN  NS         ns.myrancher.lab.
         IN  A          192.168.0.10
ns       IN  NS         192.168.0.9
docker   IN  A          192.168.0.10
node1    IN  A          192.168.0.11
node2    IN  A          192.168.0.12
rancher1 IN  CNAME      docker
```

7. Disable Firewall and start the DNS service
```
systemctl disable --now firewalld
```
```
systemctl enable --now named
```

Self signed certificates
===

1. Use the "Openssl" commands to generate all certificates.
```
openssl genrsa -out ca.key 4096
```
```
openssl req -new -x509 -days 3650 -key ca.key -subj "/C=CN/ST=GD/L=SZ/O=Acme, Inc./CN=Acme Root CA" -out ca.crt
```
```
openssl req -newkey rsa:2048 -nodes -keyout rancher.key -subj "/C=CN/ST=GD/L=SZ/O=Acme, Inc./CN=*.myrancher.lab" -out rancher.csr
```
```
openssl x509 -req -extfile <(printf "subjectAltName=DNS:myrancher.lab,DNS:rancher1.myrancher.lab") -days 365 -in rancher.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out rancher.crt
```
2. Transfer the ca.crt and rancher.crt certificates to Node1 and Node2.

3. Login SSH to Node1, and run commands to disable Firewall and copy certificates, repeat for Node2.
```
systemctl disable --now firewalld
```
```
mv *.crt /etc/pki/trust/anchors/
```
```
update-ca-certificates
```

Deploy Rancher
===

1. Install Docker and pull image rancher/rancher:latest.

2. Run the command to start container.
```
docker run -d -p 80:80 -p 443:443 -p 2375:2375 -v {PATH-SSL}/rancher.crt:/etc/rancher/ssl/cert.pem -v {PATH-SSL}/rancher.key:/etc/rancher/ssl/key.pem -v {PATH-SSL}/ca.crt:/etc/rancher/ssl/cacerts.pem --privileged rancher/rancher:latest
```
3. Open the web browser and paste url: https://rancher1.myrancher.lab/

4. Copy the command in Rancher page, go to the container terminal of Rancher and paste the command for showing the secret hash.
```
kubectl get secret --namespace cattle-system bootstrap-secret -o go-template='{{.data.bootstrapPassword|base64decode}}{{"\n"}}'
```

5. Copy the hash, back to web browser, paste in "Password" and click the button.

6. In next page:
   - Mark to "Set a specific password to use", use 12 chars for password "admin"
   - Check the Server URL: https://rancher1.myrancher.lab
   - Mark the checkbox to agree terms and click "Continue" button

Create the Downstream Cluster
===

1. In Rancher. go to "Cluster Management", and click the "Create" button.

2. Check the RKE2/K3s is enable and click the "Custom".

3. Inform those values:
   - Type the cluster name, e.g. "mycluster"
   - Check the Kubernetes Version
   - Use the "canal" in "Container Network"
   - Click in "Create" button

4. In Registration Tab:
   - Mark only "etcd" and "Control Plane"
   - Mark the "Insecure" checkbox to skip
   - Click in the box script to copy (verify params: --etcd --controlplane)

5. Back to Node1 Terminal and paste the script for etcd and Control Plane.

6. Back to Registration Tab:
   - Mark only "Worker"
   - Mark the "Insecure" checkbox to skip
   - Click in the box script to copy (verify param: --worker)

7. Back to Node2 Terminal and paste the script for Worker.

8. Back to Web browser, and click the "Machines" tab, wait for both nodes change status to "Active".

9. For debugs, check logs in Rancher Container or use the command in Node1:
```
journalctl -u rke2-server -f
```
