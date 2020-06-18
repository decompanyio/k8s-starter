# node-group 마이그레이션하기

> 참조 Migrating to a new worker node group [https://docs.aws.amazon.com/eks/latest/userguide/migrate-stack.html]
```



CLUSTER_NAME=frontend-cluster-asem
AWS_REGION=ap-northeast-2

echo "CLUSETER_NAME :" $CLUSTER_NAME
echo "REGION        :" $AWS_REGION
echo ""

eksctl get nodegroups --cluster=$CLUSTER_NAME --region $AWS_REGION

eksctl create nodegroup \
--cluster $CLUSTER_NAME \
--version 1.14 \
--region $AWS_REGION \
--name standard-workers-new \
--node-type c5.large \
--nodes 2 \
--nodes-min 1 \
--nodes-max 4 \
--ssh-public-key ./ko-decompany.pub \
--node-ami auto
```

### nodegroup 삭제하기
```
eksctl delete nodegroup --cluster $CLUSTER_NAME --region $AWS_REGION --name standard-workers-new
eksctl get nodegroups --cluster=$CLUSTER_NAME --region $AWS_REGION
```