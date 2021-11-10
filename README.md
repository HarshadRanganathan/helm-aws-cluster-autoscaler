# helm-aws-cluster-autoscaler

Helm chart for setting up Cluster Autoscaler in your EKS cluster.

Chart Reference - https://github.com/kubernetes/autoscaler

Table of Contents
=================

   * [helm-aws-cluster-autoscaler](#helm-aws-cluster-autoscaler)
      * [Pre-requisites](#pre-requisites)
         * [IAM](#iam)
      * [Install/Upgrade Chart](#installupgrade-chart)

## Pre-requisites

### IAM

We will be using IRSA (IAM Roles for Service Accounts) to give the required permissions to the Cluster Autoscaler pod.

`Note: You need to create an OIDC provider for your cluster to make use of IRSA. Refer - https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html`

1. Create a new IAM policy `aws-cluster-autoscaler-pol` with the policy document at `iam/policy.json`

2. Create a new IAM role `aws-cluster-autoscaler-rol` and attach the IAM policy `aws-cluster-autoscaler-pol`

3. Update the trust relationship of the IAM role `aws-cluster-autoscaler-rol` as below replacing the `account_id`, `eks_cluster_id` and `region` with the appropriate values.

This trust relationship allows pods with serviceaccount `cluster-autoscaler` in `kube-system` namespace to assume the role.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "",
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::<account_id>:oidc-provider/oidc.eks.us-east-1.amazonaws.com/id/<eks_cluster_id>"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "oidc.eks.<region>.amazonaws.com/id/<eks_cluster_id>:sub": "system:serviceaccount:kube-system:cluster-autoscaler"
        }
      }
    }
  ]
}
```

### Config Updates

1. Update `stages/prod/prod-values.yaml` file with EKS cluster and IAM Role ARN.

```yaml
autoDiscovery:
  clusterName: # eks cluster name
 
 rbac:
  serviceAccount:
    annotations:
      eks.amazonaws.com/role-arn: # iam role arn
```

## Install/Upgrade Chart

Run below helm command to install/upgrade the helm chart by providing shared and stage specific values.

```bash
helm upgrade -i cluster-autoscaler . -n platform --values=stages/shared-values.yaml --values=stages/prod/prod-values.yaml
```
