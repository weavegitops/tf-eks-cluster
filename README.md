# GitOpsify an EKS Cluster using TF-Controller from Weaveworks

This repo explains how to apply the GitOps methodology to the Terraform config repo:  [Provision an EKS Cluster learn guide](https://learn.hashicorp.com/terraform/kubernetes/provision-eks-cluster), containing
- Terraform configuration files to provision an EKS cluster on AWS.
- Terraform custom resource to manage the tf resources defined in the above configuration files.

Here are the steps to make this run properly:
1. I did use a **KinD Cluster** but feel free to use another type of k8s cluster (changes may apply though)
2. Deploy **FluxCD** on your cluster in one of the way described [here](https://fluxcd.io/docs/installation/)
3. Deploy the **Terraform controller** to your FluxCD namespace. Steps are [here](https://weaveworks.github.io/tf-controller/getting_started/)!
4. Create a **secret** on your cluster for AWS Authentication purposes:
```yaml
cat <<EOF | kubectl apply -f -
---
apiVersion: v1
kind: Secret
metadata:
  name: aws-credentials
  namespace: flux-system
data:
  access_key: <<64bEncoded-AWS-Access-key-ID>>
  secret_key: <<64bEncoded-AWS-Secret-access-key>>
EOF
```

5. Create a **GitRepo** and **Kustomization** objects to deploy your Terraform custom resource stored in the folder /tf-controller-files: 
```yaml
cat <<EOF | kubectl apply -f -
---
apiVersion: source.toolkit.fluxcd.io/v1beta1
kind: GitRepository
metadata:
  name: eks-cluster-gitrepo
  namespace: flux-system
spec:
  interval: 30s
  url: https://github.com/talaverant/tf-eks-cluster
  ref:
    branch: main
---
apiVersion: kustomize.toolkit.fluxcd.io/v1beta2
kind: Kustomization
metadata:
  name: eks-cluster-ks
  namespace: flux-system
spec:
  prune: true
  interval: 2m
  destroyResourcesOnDeletion: true
  path: "./tf-controller-files"
  sourceRef:
    kind: GitRepository
    name: eks-cluster-gitrepo
  timeout: 3m
EOF
```

Note: If you want to change the name of the **cluster** or **AWS Region**, you can do that on the vpc.tf file!
