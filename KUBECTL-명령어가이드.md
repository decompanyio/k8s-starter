# kubectl 간단한 명령어들~

## Cluster Info

```
kubectl cluster-info
```

## Cluster Info Dump Export

```
kubectl cluster-info dump > dump.log
```

## Object 정보조회

```
kubectl get [service, pod, ingress, service] -o wide
```

## app=ps-frontend 인 모든 Pod 정보기 label 정보 포함 (모든 namespace)

```
kubectl get pod -o wide --all-namespaces --show-labels=true -l app=ps-frontend
```


## 전체 namespace에서 app=ps-frontend를 가지고 있는 Pod를 찾아 삭제

```
kubectl delete pod --all-namespaces -l app=ps-frontend 
```

## Pod 정보보기

```
kubectl describe pod {Pod Name}
kubectl describe pod ps-frontend-59b6b898db-869xg
```

## 현재 Service(LB)에서 선택된 deployment의 version(blue/green) 가져오기 

```
VERSION=$(kubectl get service -n frontend -l app=ps-frontend -o json | jq -r '.items[].spec.selector.version')
echo $
```