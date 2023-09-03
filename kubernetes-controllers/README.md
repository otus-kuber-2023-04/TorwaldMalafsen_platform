# Выполнено ДЗ №2
1. Основное ДЗ
2. Задание со *
## В процессе сделано
---
* Устаовлен kind и создан кластер из 1 мастер ноды и 3 воркер нод
* Применен манфиест frontend-replicaset и запущена одна реплика микросервиса frontend
* Запущено обновление ReplicaSet на версию образа 0.0.2
* Написан и применен Deployment манифест для сервиса PaymentService 
* Обновлен Deployment на версию образа 0.0.2
* Исследована работа Rollbacl Deployment
* Написан манифест Deployment с параметрами maxSurge и maxUnavailable
* Изучена работа механизма Probes
* Изучена работа DaemonSet контроллеров kubernetes
* Написан манифест node-exporter для развертывания DaemonSet с Node Exporter

## Как запустить проект
---
* создадим кластер из конфига и проверим создание master и worker нод
```
kind create cluster --config kind-config.yaml
kubectl get nodes
```

* запустим одну реплику микросервиса **frontend** и проверим работу + затем увеличим количество реплик до 3
```
kubectl apply -f frontend-replicaset.yaml
kubectl get pods -l app=frontend
kubectl scale replicaset frontend --replicas=3
kubectl get rs frontend
```

* Проверим, что благодаря контроллеру pods восстанавливаются после их ручного удаления
```
kubectl delete pods -l app=frontend | kubectl get pods -l app=frontend -w
```

* Обновим ReplicaSet на версию образа 0.0.2 + параллельно отслеживаем происходящее + проверим образ, запущенный в **replicaSet**
```
kubectl apply -f frontend-replicaset.yaml | kubectl get pods -l app=frontend -w
kubectl get replicaset frontend -o=jsonpath='{.spec.template.spec.containers[0].image}'
```
*  Напишем и применим Deployment  манифест для микросервиса **paymentSerivce**
```
kubectl delete pods -l app=frontend | kubectl get pods -l app=frontend -w
```
* Обновим Deployment на версию образа 0.0.2 + запустим отслеживание происходящего
```
kubectl apply -f paymentservice-deployment.yaml | kubectl get pods -l app=paymentservice -w
```
* Посмотрим историю версий Deployment

```
kubectl rollout history deployment paymentservice
```
* Откатить обновление на предыдущую версию микросервиса можно командой

```
kubectl rollout undo deployment paymentservice --to-revision=1 | kubectl get rs -l app=paymentservice -w
```
* Посмотрим историю версий Deployment

```
kubectl rollout history deployment paymentservice
```
* Изучим работу DaemonSet на примере Node Exporter

```
kubectl apply -f node-exporter-daemonset.yaml
kubectl port-forward node-exporter 9100:9100
```
## Как проверить работоспособность
---
Для просмотра метрик node exporter перейти по ссылке - http://localhost:9100/metrics