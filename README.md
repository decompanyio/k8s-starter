# k8s-starter
EKS Cluster 설치하기

## 주의

us-west-2(오레곤)에 설치하는 가이드임
us-west-1(캘리포니아)에는 EKS가 없음

## eksctl 설치하기 및 EKS 가이드

> [가이드(https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/getting-started-eksctl.html)](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/getting-started-eksctl.html)


## Key Pair 

```
ssh-keygen
```

## Export public key from pem file

```
ssh-keygen -y -f /Users/jay/Documents/infraware/decompany.oregon.pem > decompany.oregon.pub
```

## Create Cluster

```bash
eksctl create cluster \
--name dev \
--version 1.14 \
--region us-west-2 \
--nodegroup-name standard-workers \
--node-type t3.medium \
--nodes 3 \
--nodes-min 1 \
--nodes-max 4 \
--ssh-access \
--ssh-public-key {ssh-keygen을 통하여 export된 public 파일} \
--managed
```

## ALB Ingress Controller 설치

```
echo '>>> CREATE ALBIngressControllerIAMPolicy '
aws iam create-policy \
--policy-name ALBIngressControllerIAMPolicy \
--policy-document https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.3/docs/examples/iam-policy.json
echo ''

echo '>>> Connecting ALBIngressControllerIAMPolicy To WorkerNode Role'
NG_ROLE=`kubectl -n kube-system describe configmap aws-auth | grep rolearn`
ACCOUNT=${NG_ROLE:24:12}
WN_ROLE=${NG_ROLE:42}
echo "ACCOUNT          : $ACCOUNT"
echo "WORKER NODE ROLE : $WN_ROLE"
echo "NODE GROUP ROLE  : $NG_ROLE"

aws iam attach-role-policy \
--policy-arn arn:aws:iam::${ACCOUNT}:policy/ALBIngressControllerIAMPolicy \
--role-name ${WN_ROLE}
echo ''

echo '>>> Create ClusterRole for ALB Ingress Controller' 
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.3/docs/examples/rbac-role.yaml
echo ''

echo '>>> Create ALB Ingress Controller'
CLUSTER_NAME='dev' # write your's cluster name
AWS_REGION='us-west-2' # write cluster's region
VPC_ID=`eksctl get cluster --name ${CLUSTER_NAME} --region ${AWS_REGION} --output json | jq -r '.[0].ResourcesVpcConfig.VpcId'`
echo "CLUSTER NAME : $CLUSTER_NAME"
echo "VPC ID       : $VPC_ID"
echo "AWS REGION   : $AWS_REGION"
echo ''

echo '>>> Remove Old alb-ingress-controller.yaml file && New alb-ingress-controller.yaml file Download'
rm -rf alb-ingress-controller.yaml* &&
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.3/docs/examples/alb-ingress-controller.yaml &&  
# alb-ingress-controller.yaml
sed -i -e "s/# - --cluster-name=devCluster/- --cluster-name=$CLUSTER_NAME/g" alb-ingress-controller.yaml &&
sed -i -e "s/# - --aws-vpc-id=vpc-xxxxxx/- --aws-vpc-id=$VPC_ID/g" alb-ingress-controller.yaml &&
sed -i -e "s/# - --aws-region=us-west-1/- --aws-region=$AWS_REGION/g" alb-ingress-controller.yaml


kubectl apply -f ./alb-ingress-controller.yaml  
echo '>>> FINISH'
sleep 5

echo '>>> Checking Create ALB Ingress Controller'
kubectl get pods -n kube-system | grep alb
```

## Log 수집을 위한 Policy 추가

```
STACK_NAME=$(eksctl get nodegroup --region=us-west-2 --cluster dev -o json | jq -r '.[].StackName')
ROLE_NAME=$(aws cloudformation describe-stack-resources --region us-west-2 --stack-name $STACK_NAME | jq -r '.StackResources[] | select(.ResourceType=="AWS::IAM::Role") | .PhysicalResourceId')
aws iam put-role-policy --region us-west-2 --role-name $ROLE_NAME --policy-name Logs-Policy-For-Worker --policy-document file://k8s-logs-policy.json
```

## Deploy Fluentd in kube-system

fluentd.yml 다운로드(기본설정)

```
wget https://eksworkshop.com/intermediate/230_logging/deploy.files/fluentd.yml
```

아래를 올바르게 수정

```
env:
    - name: REGION
    value: us-west-2
    - name: CLUSTER_NAME
    value: dev
```

Fluentd 설치하기
```
kubectl apply -f fluentd.yml
```

확인하기

```
kubectl get pods -w --namespace=kube-system
```



## Deploy

```
kubectl apply -f frontend/ps-frontend-deployment.yaml
kubectl apply -f frontend/ps-frontend-service.yaml
kubectl apply -f frontend/ps-frontend-ingress.yaml
```

## Check Deploy status

```
kubectl get deployments
kubectl get service ps-frontend -o wide
echo $(kubectl get service ps-frontend -o json | jq -r '.status.loadBalancer.ingress[].hostname')
```
