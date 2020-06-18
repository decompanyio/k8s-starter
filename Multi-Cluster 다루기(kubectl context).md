# Multi-Cluster 다루기

## 현재 kubectl에 등록된 클러스터 목록 조회 

설정파일 보기
```
vi ~/.kube/config
```

kubectl 이요
```
kubectl config get-contexts
```

결과
```
CURRENT   NAME                                                                    CLUSTER                                                                 AUTHINFO                                                                NAMESPACE
          arn:aws:eks:ap-northeast-2:197966029048:cluster/frontend-cluster-asem   arn:aws:eks:ap-northeast-2:197966029048:cluster/frontend-cluster-asem   arn:aws:eks:ap-northeast-2:197966029048:cluster/frontend-cluster-asem   
          jay@frontend-cluster-asem.ap-northeast-2.eksctl.io                      frontend-cluster-asem.ap-northeast-2.eksctl.io                          jay@frontend-cluster-asem.ap-northeast-2.eksctl.io                      
*         jay@prod-cluster.ap-northeast-2.eksctl.io                               prod-cluster.ap-northeast-2.eksctl.io                                   jay@prod-cluster.ap-northeast-2.eksctl.io    kub
```

## context 변경

```
kubectl config use-context {context name}
kubectl config use-context jay@prod-cluster.ap-northeast-2.eksctl.io
```