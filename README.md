# AWS IAM Operator

[![Join the chat at https://gitter.im/aws-iam-operator/community](https://badges.gitter.im/aws-iam-operator/community.svg)](https://gitter.im/aws-iam-operator/community?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

An operator that enables AWS IAM management via Kubernetes custom resources.

## Installation

**CRD**

The CRDs can easily be applied to the cluster with kubectl:
```shell script
kubectl kustomize 'github.com/redradrat/aws-iam-operator/config/crd?ref=master' | kubectl apply -f -
```
or for a specific GITREF (e.g. branch, tag) with:
```shell script
kubectl kustomize 'github.com/redradrat/aws-iam-operator/config/crd?ref=GITREF' | kubectl apply -f -
```

**Controllers**

The controller deployment can be applied to the cluster with kuebctl:
```shell script
kubectl kustomize github.com/redradrat/aws-iam-operator/config/manager?ref=master | kubectl apply -f -
```

## Custom Resources

* [Role](#Role)
* [AssumeRolePolicy](#AssumeRolePolicy)
* [Policy](#Policy)
* [PolicyAttachment](#PolicyAttachment)

### Role

The Role resource abstracts an AWS IAM Role. 

Setting an `assumeRolePolicy` or an `assumeRolePolicyRef` is **mandatory**.
Creating a `ServiceAccount` resource is possible via `createServiceAccount`. The created ServiceAccount includes the EKS OIDC support annotation.

```yaml
apiVersion: aws-iam.redradrat.xyz/v1beta1
kind: Role
metadata:
  name: role-sample
  namespace: default
spec:
  assumeRolePolicy:
    - effect: "Allow"
      principal:
        "Federated": "blabla"
      actions:
        - "sts:AssumeRoleWithWebIdentity"
      conditions:
        "StringEquals":
          "blablabla": "system:serviceaccount:kube-system:aws-cluster-autoscaler"
  createServiceAccount: true
```

Resulting `ServiceAccount`:
```yaml
❯ k get sa role-sample -o yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::0000000000:role/role-sample
  creationTimestamp: "2020-02-30T00:25:61Z"
  name: role-sample
  namespace: default
  ownerReferences:
  - apiVersion: aws-iam.redradrat.xyz/v1beta1
    blockOwnerDeletion: true
    controller: true
    kind: Role
    name: role-sample
    uid: ...
```

### AssumeRolePolicy

The AssumeRolePolicy is an auxiliary resource for the `Role` resource. It provides a way to define a single trust policy for multiple roles.

```yaml
apiVersion: aws-iam.redradrat.xyz/v1beta1
kind: AssumeRolePolicy
metadata:
  name: assumerolepolicy-sample
spec:
  statement:
    - sid: someid
      effect: "Allow"
      principal:
        "Federated": "blabla"
      actions:
        - "xxxx:DescribeSomething"
      resources:
        - "*"
      conditions:
        "StringEquals":
          "aws:SourceIp": "172.0.0.1"
```

### Policy

The Policy resource abstracts an AWS IAM Policy.

For `conditions`, please check https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition_operators.html for valid Operators. For the comparison, only single String-type values are allowed as comparison values. For keys please check out https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html

```yaml
apiVersion: aws-iam.redradrat.xyz/v1beta1
kind: Policy
metadata:
  name: policy-sample
spec:
  statement:
    - sid: someid
      effect: "Allow"
      actions:
        - "xxxx:DescribeSomething"
      resources:
        - "*"
      conditions:
        "StringEquals":
          "aws:SourceIp": "172.0.0.1"
```

### PolicyAttachment

The Policy resource abstracts the attachment of an AWS IAM Policy to another AWS IAM Resource e.g. Role (in future maybe User, Groups, etc.).

```yaml
apiVersion: aws-iam.redradrat.xyz/v1beta1
kind: PolicyAttachment
metadata:
  name: policyattachment-sample
spec:
  policy:
    name: policy-sample
    namespace: default
  target:
    type: Role
    name: role-sample
    namespace: default
```