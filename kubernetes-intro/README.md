# Выполнено ДЗ №1
1. Основное ДЗ
2. Задание с *
## В процессе сделано
---
* Установлен и запущен minikube
* Создан Dockerfile с описанием образа:
   * web сервер запускается на порту 8000
   * в директории /app отдается файл homework.html и доступен по url http://localhost:8000/homework.html
   * сервер работает с пользователем nginx с UID 1001
   * собран образ контейнера из Dockerfile и запушен в публичную репу Docker Hub
```
docker build -t torwaldmalafsen/otus_kuber_nginx:1.0.8 .
docker push torwaldmalafsen/otus_kuber_nginx:1.0.8
```

* Написан манифест **web-pod.yaml** для создания pod **web** и содержащий контейнер с названием **web**
* Применен манифест, проверена его работоспособность

```
kubectl apply -f web-pod.yaml

kubectl get pods
NAME     READY   STATUS     RESTARTS       AGE
web       1/1    Running       0           58m
```

* В pod добавлен init-контейнер с исползованием volume **emptyDir** генерирующий страницу ***index.html***
* Запущен pod и проверена работоспособность web-сервера, команда с отслеживанием происходящего и port-forwarding:

```
kubectl apply -f web-pod.yaml && kubectl get pods -w
kubectl port-forward --address 0.0.0.0 pod/web 8000:8000
```
* ***Hipster Shop***. Клонировван репозиторий и собран обраp микросервиса **frontend** и запушен образ в Docker Hub
```
git clone https://github.com/GoogleCloudPlatform/microservices-demo.git
docker build -t torwaldmalafsen/otus_kuber_frontend:1.0.0 .
docker push torwaldmalafsen/otus_kuber_frontend:1.0.0
```
* Запущен **frontend** pod + сгенерирован манифест с помощью kubectl
```
kubectl run frontend --image torwaldmalafsen/otus_kuber_frontend:1.0.0 --restart=Never

kubectl run frontend --image torwaldmalafsen/otus_kuber_frontend:1.0.0 --restart=Never --dry-run -o yaml > frontend-pod.yaml
```
* Создан новый манифест **frontend-pod-healthy.yaml**, т.к. предыдущий запускался со статусом Error. Причины ошибки:
    * Не были установлены переменные окружения, например:
    ```
    panic: environment variable "PRODUCT_CATALOG_SERVICE_ADDR" not set

      - name: PRODUCT_CATALOG_SERVICE_ADDR
        value: "productcatalogservice:3550"
    ```

### **Ответ на первый вопрос**
---
Список подов и описание:
```
kubectl get pods -n kube-system
kubectl get pods -n kube-system -o yaml

coredns-787d4945fb-swz46
kind: ReplicaSet

kube-proxy-bwt5s
kind: DaemonSet
```
**ReplicaSet** - гарантирует, что определенное количество экземпляров подов всегда будет запущено в кластере

**DaemonSet** - это объект Kubernetes, который гарантирует, что копия модуля pod, определенного в конфигурации, всегда будет доступна на каждом рабочем узле в кластере