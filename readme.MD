# Ingress Controller
## OIDC provider
trust and authecation provider

REGION_CODE=us-east-1
CLUSTER_NAME=roboshop
ACC_ID=203981192510
``` 
eksctl utils associate-iam-oidc-provider \
    --region $REGION_CODE \
    --cluster $CLUSTER_NAME \
    --approve
```
dowload IAM policy 

```
curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.13.4/docs/install/iam_policy.json
``` 
create IAM policy
``` 
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy.json
``` 
create IAM Role and k8 ServiceAccount. mapping between IAM roles and k8 service account

```
 eksctl create iamserviceaccount \
  --cluster=$CLUSTER_NAME \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --attach-policy-arn=arn:aws:iam::$ACC_ID:policy/AWSLoadBalancerControllerIAMPolicy \
  --override-existing-serviceaccounts \
  --region $REGION_CODE \
  --approve

 eksctl create iamserviceaccount \
  --cluster=roboshop \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --attach-policy-arn=arn:aws:iam::203981192510:policy/AWSLoadBalancerControllerIAMPolicy \
  --override-existing-serviceaccounts \
  --region us-east-1 \
  --approve
```
### install load balancer controller driers
```
helm repo add eks https://aws.github.io/eks-charts
```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=$CLUSTER_NAME --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller