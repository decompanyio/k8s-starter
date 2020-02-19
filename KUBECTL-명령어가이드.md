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
kubectl [service, pod, ingress, service] get -o wide
```