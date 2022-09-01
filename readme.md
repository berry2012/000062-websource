
![Create Infrastructure](./architecture.png) 


## Prerequisites

1. Installed Kubernetes Tool
2. IAM Role Configured
3. Installed eksctl Tool
4. EKS Cluster
5. ALB Controller plugin deployed in the EKS Cluster
6. Github Authorization token from your Github account
7. Create S3 bucket
8. Create CodePipeline Service Role
9. Create Code Buid Service Role
10. Allow role eks-CodeBuildServiceRole permissions in RBAC of EKS cluster

## Tech Stack Involved
- Github
- CodePipeline
- CodeBuild
- ECR
- EKS


## Create existing Kubernetes Workload
`Deployment --> Service --> Ingress (ALB)`

```
kubectl create -f infrastructure/workload.yml
```

## Kubernetes Namespace
- persportfolio

## How New Releases are rolledout
New Docker Image is built and existing deployment is set with the latest image.


## Create the CICD Pipeline with CloudFormation Template 
`infrastructure/cloudformation.yml`

## Create service role for CodePipeline service and Create service role for CodeBuild service
```
wget https://raw.githubusercontent.com/First-Cloud-Journey/000062-EKSCICD/main/codepipeline/cpAssumeRolePolicyDocument.json
aws iam create-role --role-name eks-CodePipelineServiceRole --assume-role-policy-document file://cpAssumeRolePolicyDocument.json
wget https://raw.githubusercontent.com/First-Cloud-Journey/000062-EKSCICD/main/codepipeline/cpPolicyDocument.json
aws iam put-role-policy --role-name eks-CodePipelineServiceRole --policy-name codepipeline-access --policy-document file://cpPolicyDocument.json
wget https://raw.githubusercontent.com/First-Cloud-Journey/000062-EKSCICD/main/codebuild/cbAssumeRolePolicyDocument.json
aws iam create-role --role-name eks-CodeBuildServiceRole --assume-role-policy-document file://cbAssumeRolePolicyDocument.json
wget https://raw.githubusercontent.com/First-Cloud-Journey/000062-EKSCICD/main/codebuild/cbPolicyDocument.json
aws iam put-role-policy --role-name eks-CodeBuildServiceRole --policy-name codebuild-access --policy-document file://cbPolicyDocument.json
```


## Configure RBAC and add the CodeBuild Service role
`kubectl get configmaps aws-auth -n kube-system -o yaml > aws-auth.yaml`


```
- groups:
  - system:masters
  rolearn: arn:aws:iam::{$AWS_ACCOUNT_ID}}:role/eks-CodeBuildServiceRole
  username: codebuild-eks
```

`kubectl apply -f aws-auth.yaml`
