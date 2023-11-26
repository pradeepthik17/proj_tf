# Prerequisites:
Install kubectl, eksctl, AWS cli, helm on MacOS/Windows/Linux

# Install EKS : 
# Install using Fargate :

eksctl create cluster --name <cluster_name> --region us-east-1 --fargate

Check the cluster and services created on AWS console.

# Run below command to update the kube config:

aws eks update-kubeconfig --name <cluster_name> --region us-east-1

# Create 2048 App:
# Create Fargate profile:

eksctl create fargateprofile --cluster demo-cluster --region us-east-1 --name alb-sample-app --namespace game-2048

# Deploy the deployment, service and Ingress:
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml

# Check if there is an IAM OIDC provider configured already, If not, run the below command

eksctl utils associate-iam-oidc-provider --cluster <cluster_name> --approve

# Setup alb:
# Download IAM policy:

curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json

# When you are trying to run curl from powershell, if it will throw error please run below command to fix 
cmd: remove-item alias:\curl

# Create IAM Policy:

aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json

# Create IAM Role:

eksctl create iamserviceaccount --cluster=<your-cluster-name> --namespace=kube-system --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy --approve

# Deploy ALB controller:
# Add helm repo:

helm repo add eks https://aws.github.io/eks-charts

# Update the repo:

helm repo update eks

# Install alb:

helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=<your-cluster-name> --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller --set region=<region> --set vpcId=<your-vpc-id>
  
# Verify that the deployments are running:

kubectl get deployment -n kube-system aws-load-balancer-controller

# Kubectl commands used in this project to check the status of pods and services

- kubectl get pods -n game-2048
- kubectl get svc -n game-2048
- kubectl get ingress -n game-2048
- kubectl get deploy -n kube-system

# Copy the DNS name and run it on browser, you will see the application
# Delete the cluster
eksctl delete cluster --name <cluster_name> --region us-east-1  #If you face any issue, please delete manually from Aws console



