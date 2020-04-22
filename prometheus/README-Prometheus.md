
# Prometheus 설치하기

## Namespace
  - monitoring 로 생성됨
  - ./kube-prometheue/manifests/setup/0namespace-namespace.ymal 참조

## Kube-prometheus 이용하기

[설치 가이드](https://github.com/coreos/kube-prometheus#quickstart)

```
git clone https://github.com/coreos/kube-prometheus.git
kubectl create -f kube-prometheus/manifests/setup
kubectl create -f kube-prometheus/manifests
```


## Ingress 설정하기

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
