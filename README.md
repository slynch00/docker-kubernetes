# docker-kubernetes

## Installation using KOPS

If needed, generate some user access keys in iam

## AWS access to install kops
```
AmazonEC2FullAccess
AmazonRoute53FullAccess
AmazonS3FullAccess
IAMFullAccess
AmazonVPCFullAccess
```

### install homebrew if mac os
With Homebrew installed, on osx you just need to run:
`brew update && brew install kops`

### install on Linux
`curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64`
`chmod +x kops-linux-amd64`
`sudo mv kops-linux-amd64 /usr/local/bin/kops`

### install awscli
[MacOS](https://docs.aws.amazon.com/cli/latest/userguide/cli-install-macos.html)

[Linux](https://docs.aws.amazon.com/cli/latest/userguide/awscli-install-linux.html)

Install the amazon aws cli tools. If you don't have python installed, use the bundled installer option

[Bundle Installer](https://docs.aws.amazon.com/cli/latest/userguide/awscli-install-bundle.html)

### create s3 bucket
`aws s3api create-bucket --bucket <bucket-name> --region <region> --create-bucket-configuration LocationConstraint=<region>`
You might also want to attach a policy
	`aws s3api put-bucket-policy --bucket <bucket-name> --policy file://$PATH_TO_POLICY/s3_policy.json [--profile $AWS_PROFILE]`

### environment var for s3 bucket
`export KOPS_STATE_STORE=s3://<bucket-name>`
This creates an environment variable for kops to determine where to store state files

### create cluster
`kops create cluster <k8s-cluster-name> --zones <region><a/b/c> --yes`
ex. kops create cluster q-k8s-cluster --zones us-east-1c --yes
  For HA Master:
  `kops create cluster q-k8s-cluster --zones us-east-1a,us-east-1b,us-east-1c --node-count 3 --master-zones us-east-1a,us-east-1b,us-east-1c --yes`
  
  #### note for existing vpc
  Existing VPC use --vpc flag

(may require you to run ssh keygen if you get an error when you run kops)

### validation
`kops validate cluster`
If it says cannot get nodes, it may be still building. Could take 5-10 minutes to build

`kubectl get nodes --show-labels`

### resolving the conflicting subnet error
When implementing on an existing VPC, the potential for conflicting subnets may occur. To resolve, follow these steps. 
  `kops get clusters <cluster-name> --state s3://<bucket-name> -o yaml > cluster-config.yaml`
  Then modify the subnets to resolve the conflicts by replacing the conflicting ip ranges
  `vi  cluster-config.yaml`
  
### delete cluster
`kops delete cluster --name <name> --yes`
#### get cluster names
`kops get clusters`
