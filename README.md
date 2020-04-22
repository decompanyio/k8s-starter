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
ssh-keygen -y -f ~/polarishare/ko-decompany.pem > ko-decompany.pub 
```

## Create Cluster

```bash
CLUSTER_NAME=frontend-cluster-asem
AWS_REGION=ap-northeast-2 # write cluster's region

echo "CLUSETER_NAME :" $CLUSTER_NAME
echo "REGION        :" $REGION
echo ""

eksctl create cluster \
--name $CLUSTER_NAME \
--version 1.14 \
--region $AWS_REGION \
--nodegroup-name standard-workers \
--node-type c5.large \
--nodes 2 \
--nodes-min 1 \
--nodes-max 4 \
--ssh-access \
--ssh-public-key ./ko-decompany.pub \
--managed
```

## ALB Ingress Controller 설치

```
wget https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.3/docs/examples/iam-policy.json

echo '>>> CREATE ALBIngressControllerIAMPolicy '
aws iam create-policy \
--policy-name ALBIngressControllerIAMPolicy \
--policy-document file://./iam-policy.json
echo ''

echo '>>> Connecting ALBIngressControllerIAMPolicy To WorkerNode Role'
NG_ROLE=`kubectl -n kube-system describe configmap aws-auth | grep rolearn`
ACCOUNT=${NG_ROLE:24:12}
WN_ROLE=${NG_ROLE:42}
echo "ACCOUNT          :" $ACCOUNT
echo "WORKER NODE ROLE :" $WN_ROLE
echo "NODE GROUP ROLE  :" $NG_ROLE

aws iam attach-role-policy \
--policy-arn arn:aws:iam::${ACCOUNT}:policy/ALBIngressControllerIAMPolicy \
--role-name ${WN_ROLE}
echo ''

echo '>>> Create ClusterRole for ALB Ingress Controller' 
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.3/docs/examples/rbac-role.yaml
echo ''

echo '>>> Create ALB Ingress Controller'

VPC_ID=`eksctl get cluster --name ${CLUSTER_NAME} --region ${AWS_REGION} --output json | jq -r '.[0].ResourcesVpcConfig.VpcId'`
echo "CLUSTER NAME : " $CLUSTER_NAME
echo "VPC ID       : " $VPC_ID
echo "AWS REGION   : " $AWS_REGION
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
> https://docs.aws.amazon.com/ko_kr/AmazonCloudWatch/latest/monitoring/Container-Insights-setup-logs.html
```
STACK_NAME=$(eksctl get nodegroup --region=$AWS_REGION --cluster $CLUSTER_NAME -o json | jq -r '.[].StackName')
ROLE_NAME=$(aws cloudformation describe-stack-resources --region $AWS_REGION --stack-name $STACK_NAME | jq -r '.StackResources[] | select(.ResourceType=="AWS::IAM::Role") | .PhysicalResourceId')
aws iam put-role-policy --region us-west-2 --role-name $ROLE_NAME --policy-name Logs-Policy-For-Worker --policy-document file://k8s-logs-policy.json
```

## Deploy Fluentd in kube-system

fluentd.yml 다운로드(기본설정)
> https://banzaicloud.com/docs/one-eye/logging-operator/plugins/outputs/cloudwatch/

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

## Fluentd 설치하기
```
kubectl apply -f fluentd.yml
```

## CLUSTER 확인하기

```
kubectl get pods -w --namespace=kube-system
```



## Deploy

```
kubectl apply -f frontend/ps-frontend-deployment.yaml
kubectl apply -f frontend/ps-frontend-service.yaml
kubectl apply -f frontend/ps-frontend-ingress.yaml
```

```
kubectl delete -f frontend/ps-frontend-deployment.yaml
kubectl delete -f frontend/ps-frontend-service.yaml
kubectl delete -f frontend/ps-frontend-ingress.yaml
```

## Check Deploy status

```
kubectl get deployments
kubectl get service ps-frontend -o wide
echo $(kubectl get service ps-frontend -o json | jq -r '.status.loadBalancer.ingress[].hostname')
```

## 도메인 확인

```
kubectl get ingress -n frontend -o wide
```

## ETC

## 잘못된 삭제로 kubectl의 권한 오류가 발생할경우 복구하기


[가이드 : https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/create-kubeconfig.html](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/create-kubeconfig.html)
```
aws eks --region $AWS_REGION update-kubeconfig --name $CLUSTER_NAME

```

## Cluster 삭제하기

```
eksctl delete cluster --name {cluster_name} --region {region}
```

> Cluster 삭제시 자동으로 생성되는 Policy, Role, pod, service등 수동으로 생성된것들이 있으면 제거해야 삭제가 가능함
> VPC는 수동으로 지워야함
> 운영이 오래되면 그냥 수동으로 지울수 밖에 없음