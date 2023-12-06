---
title: Using AWS EFS as a Persistent Volume
description: Guide on utilizing EFS as a persistent volume
layout: ../../layouts/MainLayout.astro
---

Mount an EFS volume in RWX mode by using a filesystem ID and mountpoints across all subnets in all node groups. Ensure you have an EFS storage class name. Follow the [instructions below](/en/efs#tldr---configure-an-efs-volume) for provisioning.

## Prerequisites

- EFS storage class name: `<efs-storage-class-name>`
- Filesystem ID: `<filesystem-id>`

## Use in Kubeshark

Configure the following to utilize the persistent volume:

```yaml
tap:
  persistentStorage:    true
  storageClass:         <efs-storage-class-name>  # prerequisite
  storageLimit:         5Gi                       # An example
  fileSystemIdAndPath:  <filesystem-id>           # Prerequisite
```

## TL;DR - Configure an EFS Volume

### Prerequisites

Prepare the following information:
- Cluster region: `<cluster-region>`
- Node group subnets: `<subnet-id>`
- EKS cluster VPC-ID: `<cluster-vpc-id>`

### Create a Dedicated Security Group 

```yaml
aws ec2 create-security-group \
--query GroupId \
--output text \
--group-name MyEfsMountableFromEverywhereSecurityGroup \
--description "Opens inbound EFS/NFS port to be accessible from every host in subnet" \
--region <cluster-region> \      # Prerequisite
--vpc-id <cluster-vpc-id>        # Prerequisite
```
Save the group ID for the following command: `<security-group-id>`.

### Open Inbound (Ingress) NFS/EFS port `2049`

Authorize ingress on port 2049 for the security group created above in your cluster region.

```yaml
aws ec2 authorize-security-group-ingress \
--group-id <security-group-id>  \   # From the previous command
--protocol tcp \
--port 2049 \
--cidr 0.0.0.0/0 \
--region <cluster-region>           # Prerequisite
```

### Create Filesystem

Create a file system in your cluster region and note the filesystem ID.

```yaml
aws efs create-file-system  \
--query "FileSystemId" \
--output text \
--region <cluster-region>          # Prerequisite  
```
Save the filesystem ID for the following command: `<filesystem-id>`.

### Create Mount-points in Each Subnet

For each subnet across all node groups, create mount targets to provide all pods access to the file system. Use the filesystem ID and security group ID from the previous steps.

```yaml
aws efs create-mount-target \
--query "MountTargetId" \
--output text 
--file-system-id <filesystem-id> \      # From previous command
--subnet-id <subnet-id> \               # Prerequisite
--security-groups <security-group-id> \ # From previous command
--region <cluster-region>               # Prerequisite
```

### Deploy the Amazon EFS CSI driver

Deploy the Amazon EFS CSI driver using the stable ECR release.

```yaml
kubectl apply -k "github.com/kubernetes-sigs/aws-efs-csi-driver/deploy/kubernetes/overlays/stable/ecr/?ref=release-1.3"
```

### Create a `StorageClass`

Create a file named `efs-sc.yaml` with either of below content (depending from your infrastructure and cluster configuration/requirements)  and deploy it using kubectl.

#### Dynamically provisioned

Simpler and suitable in most cases. Kubehark and all other persistent volume claims with specified below storage class will be provisioned to the automatically created unique directory on the EFS with specified below file system ID 

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass         
metadata:
  name: efs-sc # Any arbitrary name can be chosen
provisioner: efs.csi.aws.com
parameters:   
  provisioningMode: efs-ap
  fileSystemId: <FileSystemId> # From previous command
  directoryPerms: "700"
```

#### Statically provisioned

In this case EFS file system ID should be specified during Kubeshark deployment (e.g. via `--set` in Helm). Can be helpful e.g. if some infrastructure has requirement (e.g. defined by organziation) to use only some directrory pre-created specially for Kubeshark with specific permissions on the EFS file system which is used for other EFS persistent volume claims as well

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: efs-sc
provisioner: efs.csi.aws.com
```
Deploy it:

```yaml
kubectl apply -f efs-sc.yaml
```