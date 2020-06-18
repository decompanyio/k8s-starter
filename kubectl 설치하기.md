region=$(curl http://169.254.169.254/latest/dynamic/instance-identity/document|grep region|awk -F\" '{print $4}')
echo "[default]" > ~/.aws/config
echo "region = ${region}" >> ~/.aws/config



INSTANCE_PROFILE_NAME=`basename $(aws ec2 describe-instances | jq -r '.Reservations[0].Instances[0].IamInstanceProfile.Arn' | awk -F "/" "{print $2}")`
aws iam get-instance-profile --instance-profile-name $INSTANCE_PROFILE_NAME --query "InstanceProfile.Roles[0].RoleName" --output text


## eks권한 오류 관련 

> https://aws.amazon.com/ko/premiumsupport/knowledge-center/eks-api-server-unauthorized-error/
aws eks update-kubeconfig --name frontend-cluster-asem --role-arn arn:aws:iam::197966029048:role/RoleEKSBuilder --region ap-northeast-2
aws eks update-kubeconfig --name frontend-cluster-asem --region ap-northeast-2


```
CURRENT   NAME                                                                    CLUSTER                                                                 AUTHINFO                                                                NAMESPACE
*         arn:aws:eks:ap-northeast-2:197966029048:cluster/frontend-cluster-asem   arn:aws:eks:ap-northeast-2:197966029048:cluster/frontend-cluster-asem   arn:aws:eks:ap-northeast-2:197966029048:cluster/frontend-cluster-asem   
          jay@frontend-cluster-asem.ap-northeast-2.eksctl.io                      frontend-cluster-asem.ap-northeast-2.eksctl.io                          jay@frontend-cluster-asem.ap-northeast-2.eksctl.io   
```

## EKS API 서버 확인하기
```
kubectl get apiservices 
kubectl get apiservices v1beta1.metrics.k8s.io
kubectl get apiservice v1beta1.metrics.k8s.io -o yaml
kubectl get apiservices | egrep metrics


```


aws --region ap-northeast-2 eks update-kubeconfig --name frontend-cluster-asem

