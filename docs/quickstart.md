# QuickStart Guide

## Table of Contents

<!-- mdformat-toc start --slug=github --no-anchors --maxlevel=6 --minlevel=1 -->

- [QuickStart Guide](#quickstart-guide)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [Install](#install)
    - [Pre-Requisites](#pre-requisites)
    - [Slurm Operator](#slurm-operator)
    - [Slurm Cluster](#slurm-cluster)
    - [Testing](#testing)

<!-- mdformat-toc end -->

## Overview

This quickstart guide will help you get the slurm-operator running and deploy
Slurm clusters to Kubernetes.

## Install

### Pre-Requisites

Install the pre-requisite helm charts.

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager jetstack/cert-manager \
	--namespace cert-manager --create-namespace --set crds.enabled=true
helm install prometheus prometheus-community/kube-prometheus-stack \
	--namespace prometheus --create-namespace --set installCRDs=true
```

### Slurm Operator

Download values and install the slurm-operator from OCI package.

```bash
curl -L https://raw.githubusercontent.com/SlinkyProject/slurm-operator/refs/tags/v0.1.0/helm/slurm-operator/values.yaml \
  -o values-operator.yaml

cd /home/cloudshell-user/slurm-operator/helm/slurm-operator
helm install slurm-operator oci://ghcr.io/slinkyproject/charts/slurm-operator \
  --values=values.yaml --version=0.1.0 --namespace=slinky --create-namespace
```

Make sure the cluster deployed successfully with:

```sh
kubectl --namespace=slinky get pods
```

Output should be similar to:

```sh
NAME                                      READY   STATUS    RESTARTS   AGE
slurm-operator-7444c844d5-dpr5h           1/1     Running   0          5m00s
slurm-operator-webhook-6fd8d7857d-zcvqh   1/1     Running   0          5m00s
```

### Slurm Cluster
Provision FSx for Lustre (CLI)


```bash
aws fsx create-file-system \
    --file-system-type LUSTRE \
    --storage-capacity 1200 \
    --subnet-ids subnet-066e85c88cb30fc47 \
    --lustre-configuration DeploymentType=PERSISTENT_2,PerUnitStorageThroughput=250 \
    --tags Key=Name,Value=slurm-fsx
```
After creation, you can check its status with:
```bash
aws fsx create-file-system \
    --file-system-type LUSTRE \
    --storage-capacity 1200 \
    --subnet-ids subnet-066e85c88cb30fc47 \
    --lustre-configuration DeploymentType=PERSISTENT_2,PerUnitStorageThroughput=250 \
    --tags Key=Name,Value=slurm-fsx
```

Download values and install a Slurm cluster from OCI package.
    
```bash
curl -L https://raw.githubusercontent.com/SlinkyProject/slurm-operator/refs/tags/v0.1.0/helm/slurm/values.yaml \
  -o values-slurm.yaml

cd /home/cloudshell-user/slurm-operator/helm/slurm

helm install slurm oci://ghcr.io/slinkyproject/charts/slurm \
  --values=values.yaml --version=0.1.0 --namespace=slurm --create-namespace
```

Make sure the slurm cluster deployed successfully with:

```sh
kubectl --namespace=slurm get pods
```

Output should be similar to:

```sh
NAME                              READY   STATUS    RESTARTS       AGE
slurm-accounting-0                1/1     Running   0              5m00s
slurm-compute-debug-0             1/1     Running   0              5m00s
slurm-controller-0                2/2     Running   0              5m00s
slurm-exporter-7b44b6d856-d86q5   1/1     Running   0              5m00s
slurm-mariadb-0                   1/1     Running   0              5m00s
slurm-restapi-5f75db85d9-67gpl    1/1     Running   0              5m00s
```

### Testing

To test Slurm functionality, connect to the controller to use Slurm client
commands:

```sh
kubectl --namespace=slurm exec -it statefulsets/slurm-controller -- bash --login
```

On the controller pod (e.g. host `slurm@slurm-controller-0`), run the following
commands to quickly test Slurm is functioning:

```sh
sinfo
srun hostname
sbatch --wrap="sleep 60"
squeue
```

See [Slurm Commands][slurm-commands] for more details on how to interact with
Slurm.

<!-- Links -->

[slurm-commands]: https://slurm.schedmd.com/quickstart.html#commands
