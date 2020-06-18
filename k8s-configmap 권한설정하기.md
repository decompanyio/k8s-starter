region=$(curl http://169.254.169.254/latest/dynamic/instance-identity/document|grep region|awk -F\" '{print $4}')
echo "[default]" > ~/.aws/config
echo "region = ${region}" >> ~/.aws/config



## eks권한 오류 관련 

> https://aws.amazon.com/ko/premiumsupport/knowledge-center/eks-api-server-unauthorized-error/

```
aws eks update-kubeconfig --name frontend-cluster-asem --region ap-northeast-2
```


## configmap 수정하기(권한 유저/롤 추가하기)

```bash
kubectl edit configmap aws-auth -n kube-system
```
> configmap aws-auth 내용
```
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
data:
  mapRoles: |
    - groups:
      - system:bootstrappers
      - system:nodes
      rolearn: arn:aws:iam::197966029048:role/eksctl-frontend-cluster-asem-node-NodeInstanceRole-JOXDQI0RQJJC
      username: system:node:{{EC2PrivateDNSName}}
    - groups:
      - system:masters
      rolearn: arn:aws:iam::197966029048:role/RoleEKSBuilder
      username: user:ec2:{{EC2PrivateDNSName}}
  mapUsers: |
    - groups:
      - system:masters
      userarn: arn:aws:iam::197966029048:user/jay
      username: jay
kind: ConfigMap
metadata:
  creationTimestamp: "2020-04-02T09:31:03Z"
  name: aws-auth
  namespace: kube-system
  resourceVersion: "7840309"
  selfLink: /api/v1/namespaces/kube-system/configmaps/aws-auth
  uid: abac2c5c-74c4-11ea-9963-0a50135df14c
                                             
```


## EKS API 서버 확인하기
```
kubectl get apiservices 
kubectl get apiservices v1beta1.metrics.k8s.io
kubectl get apiservice v1beta1.metrics.k8s.io -o yaml
kubectl get apiservices | egrep metrics


```


aws --region ap-northeast-2 eks update-kubeconfig --name frontend-cluster-asem




## Cluster Role 확인하기
```
kubectl get clusterroles -o wide 
kubectl get clusterroles system:discovery -o yaml
```

결과
```
NAME                                                                   AGE
admin                                                                  42d
alb-ingress-controller                                                 42d
aws-node                                                               42d
cluster-admin                                                          42d
edit                                                                   42d
eks:fargate-manager                                                    42d
eks:node-bootstrapper                                                  42d
eks:node-manager                                                       42d
eks:podsecuritypolicy:privileged                                       42d
fluentd                                                                42d
kube-state-metrics                                                     23d
node-exporter                                                          23d
prometheus-adapter                                                     23d
prometheus-k8s                                                         23d
prometheus-operator                                                    23d
resource-metrics-server-resources                                      23d
system:aggregate-to-admin                                              42d
system:aggregate-to-edit                                               42d
system:aggregate-to-view                                               42d
system:aggregated-metrics-reader                                       23d
system:auth-delegator                                                  42d
system:aws-cloud-provider                                              42d
system:basic-user                                                      42d
system:certificates.k8s.io:certificatesigningrequests:nodeclient       42d
system:certificates.k8s.io:certificatesigningrequests:selfnodeclient   42d
system:controller:attachdetach-controller                              42d
system:controller:certificate-controller                               42d
system:controller:clusterrole-aggregation-controller                   42d
system:controller:cronjob-controller                                   42d
system:controller:daemon-set-controller                                42d
system:controller:deployment-controller                                42d
system:controller:disruption-controller                                42d
system:controller:endpoint-controller                                  42d
system:controller:expand-controller                                    42d
system:controller:generic-garbage-collector                            42d
system:controller:horizontal-pod-autoscaler                            42d
system:controller:job-controller                                       42d
system:controller:namespace-controller                                 42d
system:controller:node-controller                                      42d
system:controller:persistent-volume-binder                             42d
system:controller:pod-garbage-collector                                42d
system:controller:pv-protection-controller                             42d
system:controller:pvc-protection-controller                            42d
system:controller:replicaset-controller                                42d
system:controller:replication-controller                               42d
system:controller:resourcequota-controller                             42d
system:controller:route-controller                                     42d
system:controller:service-account-controller                           42d
system:controller:service-controller                                   42d
system:controller:statefulset-controller                               42d
system:controller:ttl-controller                                       42d
system:coredns                                                         42d
system:csi-external-attacher                                           42d
system:csi-external-provisioner                                        42d
system:discovery                                                       42d
system:heapster                                                        42d
system:kube-aggregator                                                 42d
system:kube-controller-manager                                         42d
system:kube-dns                                                        42d
system:kube-scheduler                                                  42d
system:kubelet-api-admin                                               42d
system:node                                                            42d
system:node-bootstrapper                                               42d
system:node-problem-detector                                           42d
system:node-proxier                                                    42d
system:persistent-volume-provisioner                                   42d
system:public-info-viewer                                              42d
system:volume-scheduler                                                42d
view                                                                   42d
```