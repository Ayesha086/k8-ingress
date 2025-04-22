# Ingress Controller
*** Setup IAM for ServiceAccount ****
REGION_CODE=us-east-1
CLUSTER_NAME=expense
1.OIDC provider
```
eksctl utils associate-iam-oidc-provider \
    --region $REGION_CODE \
    --cluster $CLUSTER_NAME \
    --approve
```
2. IAM Policy 
```
curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json

3. create IAM Role
```
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy.json
```

4.Provide access toEKS through IAM Policy
```
eksctl create iamserviceaccount \
--cluster=$CLUSTER_NAME \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--attach-policy-arn=arn:aws:iam::<AWS_ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy \
--overrride-existing-serviceaccounts \
--region $REGION_CODE
--approve
```

### INSTALL DRIVERS 
USING HELM

1.Add the EKS repository to Helm:
```
helm repo add eks https://aws.github.io/eks-charts
```
2.Install the AWS Load Balancer controller, if using iamserviceaccount
```
 helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=$CLUSTER_NAME --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
```
Install the AWS Load Balancer controller, if not using iamserviceaccount
```
helm upgrade -i aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=<k8s-cluster-name>
```
