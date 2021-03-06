# Kubernetes Setup for Prometheus and Grafana

## Quick start

To quickly start all the things just do this:
```bash
kubectl create namespace monitoring
kubectl --namespace monitoring create \
  --filename https://raw.githubusercontent.com/xiaoping378/k8s-monitor/master/manifests-all.yaml
```

To shut down all components again:
```bash
kubectl delete namespace monitoring
```
本项目依赖的所有docker镜像已打包放在百度云上,所有镜像均可以在docker hub上找到 [下载镜像tar包](https://pan.baidu.com/s/1hskbi6o)

## More Details

Alternatively follow these steps to get a feeling for the different components of this setup:

```bash
kubectl create --filename manifests/prometheus-core-configmap.yaml
# kubectl get configmaps
# kubectl delete configmaps/prometheus

kubectl create --filename manifests/prometheus-core-service.yaml
# kubectl get services/prometheus
# minikube service prometheus

kubectl create --filename manifests/prometheus-core-deployment.yaml
# kubectl get --all-namespaces --output wide pods
# kubectl logs prometheus-2556266794-sd260
# kubectl delete pods/prometheus-2556266794-sd260

kubectl create --filename manifests/node-exporter-service.yaml
kubectl create --filename manifests/node-exporter-daemonset.yaml

# create Alertmanager
kubectl create --filename manifests/prometheus-alert-configmap.yaml
kubectl create --filename manifests/prometheus-alert-service.yaml
kubectl create --filename manifests/prometheus-alert-deployment.yaml

kubectl create --filename manifests/grafana-service.yaml
# kubectl get services/grafana
# minikube service grafana

kubectl create --filename manifests/grafana-deployment.yaml
# kubectl get --all-namespaces --output wide pods
```

See grafana.net for some example [dashboards](https://grafana.net/dashboards) and [plugins](https://grafana.net/plugins).

- Configure [Prometheus](https://grafana.net/plugins/prometheus) data source for Grafana.<br/>
`Grafana UI / Data Sources / Add data source`
  - `Name`: `prometheus`
  - `Type`: `Prometheus`
  - `Url`: `http://prometheus:9090`
  - `Add`

- Import [Prometheus Stats](https://grafana.net/dashboards/2):<br/>
  `Grafana UI / Dashboards / Import`
  - `Grafana.net Dashboard`: `https://grafana.net/dashboards/2`
  - `Load`
  - `Prometheus`: `prometheus`
  - `Save & Open`

- Import [Kubernetes cluster monitoring](https://grafana.net/dashboards/162):<br/>
  `Grafana UI / Dashboards / Import`
  - `Grafana.net Dashboard`: `https://grafana.net/dashboards/162`
  - `Load`
  - `Prometheus`: `prometheus`
  - `Save & Open`

Instead of manually configuring the datasource and dashboards you can run the following job. It uses the API to configure Grafana to a state similar to when you manually go through the steps described above.

```bash
kubectl create --filename manifests/grafana-import-dashboards-job.yaml
```


## Create one single manifest file

```bash
target="./manifests-all.yaml"
rm "$target"
printf -- "# Derived from ./manifests/*.yaml\n---\n" >> "$target"
for file in ./manifests/*.yaml ; do
  if [ -e "$file" ] ; then
     cat "$file" >> "$target"
     printf -- "---\n" >> "$target"
  fi
done
```

## create configmap file

```bash
kubectl create configmap prometheus-core --from-file=manifests/prometheus-core-configmap --output yaml --dry-run > manifests/prometheus-core-configmap.yaml
kubectl create configmap grafana-import-dashboards --from-file=manifests/grafana-import-dashboards-configmap --output yaml --dry-run > manifests/grafana-import-dashboards-configmap.yaml
kubectl create configmap prometheus-alert --from-file=manifests/prometheus-alert-configmap --output yaml --dry-run > manifests/prometheus-alert-configmap.yaml
```

## Credits

Based on
```
https://github.com/giantswarm/kubernetes-prometheus
```
