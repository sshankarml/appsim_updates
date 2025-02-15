---
id: arcloud-deployment-aws
title: AR Cloud AWS Deployment
sidebar_position: 2
sidebar_label: AWS
date: 02/07/2023
tags: [ARCloud, Cloud, Kubernetes, Istio, Helm, AWS]
keywords: [ARCloud, Cloud, Kubernetes, Istio, Helm, AWS]
description: "Enterprise deployment to Amazon Web Services (AWS)"
---
import DownloadArcloud from './_download_arcloud.md';
import InstallIstio from './_install_istio.md';
import InstallIstioAws from './_install_istio_aws.md';
import InstallArcloud from './_install_arcloud.md';
import DeploymentVerification from './_deployment_verification.md';

This deployment strategy will provide a production-ready system using Amazon Web Services.

## Download

<DownloadArcloud />

```shell
export DOMAIN="arcloud.domain.tld"
```

### Tools

Make sure that the following tools are installed and configured:

- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html)
- [eksctl](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html)
- SSH client - a key pair is required and can be generated using:

```shell
ssh-keygen -t rsa -b 4096
```

## Infrastructure Setup

### Kubernetes System Recommendations

- Version `1.23.x`, `1.24.x`, `1.25.x`
- 8 Nodes (each with):
  - 8 CPUs
  - 32 GB memory

Example [instance types in AWS](https://aws.amazon.com/ec2/instance-types/):

- 8 * **t3.medium**
- 4 * **m5.large**
- 2 * **m5.xlarge**

### Environment Settings

In your terminal configure the following variables per your environment:

```shell
export AWS_PROFILE="your-profile"
export AWS_ACCOUNT_ID="your-account-id"
export AWS_REGION="your-region"
export AWS_CLUSTER_NAME="your-cluster-name"
```

### Sample cluster configurations

The two options below are alternatives that can be used depending on your preferences:

- Option 1 - an unmanaged node group is used, manual installation of add-ons is required
- Option 2 - an managed node group is used, add-ons and service accounts are installed automatically

#### Option 1: Bare-bones cluster with non-managed node group

Adjust the `./setup/eks-cluster.yaml` file to your needs and create the cluster:

```shell
cat ./setup/eks-cluster.yaml | envsubst | eksctl create cluster -f -
```

Wait until the command finishes and verify the results in [CloudFormation](https://console.aws.amazon.com/cloudformation).

Confirm `kubectl` is directed at the correct context:

```shell
kubectl config current-context
```

:::info Expected response
`{your-email}@{your-cluster}.{your-region}.eksctl.io`
:::

Complete the following guides to install additional required cluster components:

- [Amazon EBS CSI driver](https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html):
  - [Creating the Amazon EBS CSI driver IAM role for service accounts](https://docs.aws.amazon.com/eks/latest/userguide/csi-iam-role.html)
  - [Managing the Amazon EBS CSI driver as an Amazon EKS add-on](https://docs.aws.amazon.com/eks/latest/userguide/managing-ebs-csi.html)
- [Installing the AWS Load Balancer Controller add-on](https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html)
- [Managing the Amazon VPC CNI plugin for Kubernetes add-on](https://docs.aws.amazon.com/eks/latest/userguide/managing-vpc-cni.html)

:::note
In case of problems installing the VPC CNI plugin, do *not* provide the version, so the default one is used instead.
:::

#### Option 2: Pre-configured cluster with managed node group and preinstalled add-ons

Adjust the `./setup/eks-cluster-managed-with-addons.yaml` file to your needs and create
the cluster:

```shell
cat ./setup/eks-cluster-managed-with-addons.yaml | envsubst | eksctl create cluster -f -
```

Wait until the command finishes and verify the results in [CloudFormation](https://console.aws.amazon.com/cloudformation).

Confirm `kubectl` is directed at the correct context:

```shell
kubectl config current-context
```

:::info Expected response
`{your-email}@{your-cluster}.{your-region}.eksctl.io`
:::

Install the AWS Load Balancer Controller (use the image repository for the selected region based on this
[list](https://docs.aws.amazon.com/eks/latest/userguide/add-ons-images.html)), e.g.:

```shell showLineNumbers
helm repo add eks https://aws.github.io/eks-charts
helm repo update
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
    -n kube-system \
    --set clusterName=$AWS_CLUSTER_NAME \
    --set serviceAccount.create=false \
    --set serviceAccount.name=aws-load-balancer-controller \
    --set image.repository=602401143452.dkr.ecr.eu-west-3.amazonaws.com/amazon/aws-load-balancer-controller
```

## Cluster verification

To make sure the cluster is correctly configured you can run the following commands:

1. Check if your cluster is accessible using eksctl:

```shell
eksctl get cluster --region $AWS_REGION --name $AWS_CLUSTER_NAME -o yaml
```

:::info
The cluster status should be ACTIVE.
:::

2. Verify that the **OIDC** issuer is configured, e.g.:

```yaml
Identity:
  Oidc:
    Issuer: https://oidc.eks.eu-west-3.amazonaws.com/id/0A6729247C19177211F7EE71E85F9F50
```

3. Check if the add-ons are installed on your cluster:

```shell
eksctl get addons --region $AWS_REGION --cluster $AWS_CLUSTER_NAME -o yaml
```

:::info
There should be 2 add-ons and their status should be ACTIVE.
:::

## Install Istio

:::note Istio
Minimum Requirements

- AR Cloud requires Istio version `1.16.x`
- DNS Pre-configured with corresponding certificate for TLS
- Configure Istio Gateway
- Open the MQTT Port (8883)
:::

<InstallIstio />

<InstallIstioAws />

## Install ARCloud

<InstallArcloud />

## Verify Installation

<DeploymentVerification />
