# k8s-starter
EKS Cluster 설치하기

## 주의
eks 1.16기준으로 작성된 문서입니다.

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
## set env

```bash
CLUSTER_NAME=prod-cluster
AWS_REGION=ap-northeast-2
EKS_VERSION=1.16

echo "CLUSETER_NAME :" $CLUSTER_NAME
echo "AWS_REGION        :" $AWS_REGION
echo "EKS_VERSION       :" $EKS_VERSION
echo ""

```

## Create Cluster

```bash

eksctl create cluster \
--name $CLUSTER_NAME \
--version $EKS_VERSION \
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
> 참조 : https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/alb-ingress.html

1) IAM OIDC 공급자를 생성하여 클러스터와 연결합니다. eksctl 버전 0.20.0 이상을 설치하지 않은 경우 eksctl 설치 또는 업그레이드의 지침을 완료하여 설치하거나 업그레이드합니다. eksctl version을 사용하여 설치된 버전을 확인할 수 있습니다.

```bash
eksctl utils associate-iam-oidc-provider \
    --region $AWS_REGION \
    --cluster $CLUSTER_NAME \
    --approve
```

2) ALB 수신 컨트롤러 포드에 대해 사용자를 대신하여 AWS API를 호출할 수 있도록 하는 IAM 정책을 다운로드합니다. GitHub에서 정책 문서를 볼 수 있습니다.

```
curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/iam-policy.json

echo '>>> CREATE ALBIngressControllerIAMPolicy '
aws iam create-policy \
--policy-name ALBIngressControllerIAMPolicy \
--policy-document file://./iam-policy.json
echo ''
반환된 정책 ARN을 기록합니다.

3) 다음 명령과 함께 사용하도록 kube-system 네임스페이스의 Kubernetes 서비스 계정(alb-ingress-controller), 클러스터 역할 및 ALB 수신 컨트롤러에 대한 클러스터 역할 바인딩을 생성합니다. kubectl을 설치하지 않은 경우 kubectl 설치의 지침을 완료하여 설치합니다

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/rbac-role.yaml
```

4) ALB 수신 컨트롤러에 대한 IAM 역할을 생성하고 이전 단계에서 생성한 서비스 계정에 역할을 연결합니다. eksctl을 사용하여 클러스터를 생성하지 않은 경우 AWS Management 콘솔 또는 AWS CLI 탭의 지침을 따릅니다.
생성된 cluster의 정보 알기

```
eksctl create iamserviceaccount \
    --region $AWS_REGION \
    --name alb-ingress-controller \
    --namespace kube-system \
    --cluster $CLUSTER_NAME \
    --attach-policy-arn arn:aws:iam::197966029048:policy/ALBIngressControllerIAMPolicy \
    --override-existing-serviceaccounts \
    --approve
```

5) 다음 명령으로 ALB 수신 컨트롤러를 배포합니다.
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/alb-ingress-controller.yaml
```

6) 다음 명령과 편집하기 위해 ALB 수신 컨트롤러 배포 매니페스트를 엽니다.

```bash
kubectl edit deployment.apps/alb-ingress-controller -n kube-system
```

```yaml
    spec:
      containers:
      - args:
        - --ingress-class=alb
        - --cluster-name=prod
        - --aws-vpc-id=vpc-03468a8157edca5bd
        - --aws-region=region-code
```

## Logging을 위한 fluentd 설정
> 참조 : https://docs.aws.amazon.com/ko_kr/AmazonCloudWatch/latest/monitoring/Container-Insights-setup-logs.html

1) CloudWatch에 대한 네임스페이스를 생성

```
kubectl apply -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/cloudwatch-namespace.yaml
```

2) fluentd 설치

클러스터 이름과 로그가 전송될 AWS 리전을 포함하여 cluster-info이라는 ConfigMap을 생성합니다. 다음 명령을 실행하여 클러스터와 리전 이름으로 자리표시자를 업데이트합니다.
```
kubectl create configmap cluster-info \
--from-literal=cluster.name=$CLUSTER_NAME \
--from-literal=logs.region=$AWS_REGION -n amazon-cloudwatch
```

fluentd 배포
```
kubectl apply -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/fluentd/fluentd.yaml
```

확인
```
kubectl get pods -n amazon-cloudwatch
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
eksctl delete cluster --name $CLUSTER_NAME --region $AWS_REGION
```

> Cluster 삭제시 자동으로 생성되는 Policy, Role, pod, service등 수동으로 생성된것들이 있으면 제거해야 삭제가 가능함
> VPC는 수동으로 지워야함
> 운영이 오래되면 그냥 수동으로 지울수 밖에 없음