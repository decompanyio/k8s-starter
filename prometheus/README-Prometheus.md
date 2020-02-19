
# Prometheus 설치하기

## Kube-prometheus 이용하기

[설치 가이드](https://github.com/coreos/kube-prometheus#quickstart)

```
git clone https://github.com/coreos/kube-prometheus.git
kubectl create -f kube-prometheus/manifests/setup
kubectl create -f kube-prometheus/manifests
```


Ingress 설정하기

```
kubectl apply -f prometheus-ingress.yaml -n monitoring
kubectl apply -f prometheus-grafana-ingress.yaml -n monitoring
```
