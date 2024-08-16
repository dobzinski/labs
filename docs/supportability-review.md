
# Supportability Review

## Download Script

```
$ curl -OLs https://raw.githubusercontent.com/rancherlabs/support-tools/master/collection/rancher/v2.x/supportability-review/collect.sh​
$ chmod +x collect.sh​
```

## Config Vars

```
$ export RANCHER_URL="https://rancher.example.com"​
$ export RANCHER_TOKEN="token-a1b2c:hp7nxfs25w5g7rlc6gkasddhzpphfjbgmcqg6g2kpv52gxg7tl2fgpq2q"​
```

# Select Upstream Cluster
```
$ export KUBECONFIG="/path/to/upstream.yaml"
```

## Run Downstream Cluster

```
$ ./collect.sh --upstream
```

# Select Downstream Cluster
```
$ export KUBECONFIG="/path/to/downstream.yaml"
```

## Run Downstream Cluster

```
$ ./collect.sh --downstream
```
