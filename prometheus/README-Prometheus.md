
# Prometheus 설치하기

## Kube-prometheus 설치하기

[설치 가이드](https://github.com/coreos/kube-prometheus#quickstart)

release-0.3 branch clone
```
git clone -b release-0.3 --single-branch {저장소 URL} https://github.com/coreos/kube-prometheus.git
```

Install
```
kubectl create -f manifests/setup
until kubectl get servicemonitors --all-namespaces ; do date; sleep 1; echo ""; done
kubectl create -f manifests/
```


## 외부에서 접속하기 위한 Dashborad Ingress 설정하기

 - ingress.kubernetes.io/whitelist-source-range : 접속 가능한 IP 입력
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: prometheus-ui
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/scheme: internet-facing
    ingress.kubernetes.io/whitelist-source-range: 118.37.116.13/32 # change this range to admin IPs
spec:
  rules:
  - http:
      paths:
      - backend:
          serviceName: prometheus-k8s
          servicePort: 9090
        path: /*
```
 - 설치하기

```bash
kubectl apply -f prometheus-ingress.yaml -n monitoring
```


## grafana ingress 설치하기

 - 접속 가능한 IP설정하기
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: grafana
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/scheme: internet-facing
    ingress.kubernetes.io/whitelist-source-range: 118.37.116.13/32 # change this range to admin IPs
spec:
  rules:
  - http:
      paths:
      - backend:
          serviceName: grafana
          servicePort: 3000
        path: /*
```

- ingress 생성하기

```
kubectl apply -f prometheus-grafana-ingress.yaml -n monitoring
```
