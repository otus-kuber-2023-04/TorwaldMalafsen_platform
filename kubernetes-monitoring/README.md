# Выполнено ДЗ №6
- [x] Основное ДЗ
- [x] Задание со *
## В процессе сделано
---
* Написан deployment, service, servicemonotor для nginx
* В конфиге указан location /basic-status
* Установлены и настроены nginx-exporter, prometheus и grafana
* В конфигурации nginx-exporter в аргументах указан **-nginx.scrape-uri**

## Как запустить проект
---
* Создадим новую сборку образа nginx с отдачей метрик в /basic_status 
```
docker build -t torwaldmalafsen/otus_kuber_nginx:1.1.9 .
docker push torwaldmalafsen/otus_kuber_nginx:1.1.9
```
* Применим deploy и service манифесты nginx

```
kubectl apply -f nginx-deployment.yaml 
kubectl apply -f nginx-service.yaml
kubectl get services
NAME          TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S) 
kubernetes    ClusterIP      10.96.0.1        <none>        443/TCP
minio         ClusterIP      None             <none>        9000/TCP
nginx-app     ClusterIP      10.100.170.219   <none>        80/TCP 
```
* Деплоим nginx-exporter

```
kubectl apply -f nginx-exporter-deployment.yaml 
kubectl apply -f kubectl nginx-exporter-service.yaml 
NAME             TYPE           CLUSTER-IP       EXTERNAL-IP   PORT
nginx-exporter   ClusterIP      10.109.221.195   <none>        80/TCP
```
* Устанавливаем оператор prometheus и деплоим его

```
LATEST=$(curl -s https://api.github.com/repos/prometheus-operator/prometheus-operator/releases/latest | jq -cr .tag_name)

curl -sL https://github.com/prometheus-operator/prometheus-operator/releases/download/${LATEST}/bundle.yaml | kubectl create -f -

kubectl wait --for=condition=Ready pods -l  app.kubernetes.io/name=prometheus-operator -n default
```
* Деплоим prometheus

```
kubectl apply -f servicemonitor.yaml
kubectl apply -f prometheus-deployment.yaml
kubectl get pods

NAME                                   READY   STATUS    RESTARTS       AGE
minio-0                                1/1     Running   3 (117m ago)   52d
nginx-app-9b7d55956-4qcg2              1/1     Running   0              14m
nginx-app-9b7d55956-72j24              1/1     Running   0              14m
nginx-app-9b7d55956-9j9ql              1/1     Running   0              14m
nginx-exporter-5d5f785fbc-tj4dw        1/1     Running   0              7m57s
prometheus-operator-744c6bb8f9-4nqsq   1/1     Running   0              4m1s
prometheus-prometheus-0                2/2     Running   0              2m56s
```
*  Деплоим Grafana

```
kubectl apply -f grafana-deployment.yaml
```
* Прокидываем порты для nginx, prometheus и графаны. Проверяем доступность на локалхосте

```
kubectl port-forward svc/nginx-app 8080:80
kubectl port-forward service/grafana 3000:3000
kubectl port-forward --address 0.0.0.0 svc/prometheus-operated 9090:9090
```
* Заходим в Графану, указываем в Data Source источник - http://prometheus-operated:9090/. Создадим дашборд:
![Grafana Dashboard](/kubernetes-monitoring/pics/grafana_dashboards.png)


* Метрики доступны в Prometheus
![Prometheus](/kubernetes-monitoring/pics/prometheus.png)

* Путь /basic_status доступен в nginx
![Nginx](/kubernetes-monitoring/pics/nginx_status.png)

## PR checklist:
---
 - [x] Выставлен label с темой домашнего задания
