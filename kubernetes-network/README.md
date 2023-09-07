# Выполнено ДЗ №2
- [x] Основное ДЗ
- [x] Задание coredns со *
## В процессе сделано
---
* Создан манифест **web-pod** и добавлены проверки Readiness Probe и Liveness Probe
* Создан и применен манфиест deployment для web
* Исправлена проблема с available deployment увеличением реплик до 3 и пропиской порта 8000 в readinessProbe
* Добавлен блок **startegy** в манифест web-deploy
* Протестированы варианты деплоя с разными значениями **maxUnavailable** и **maxSurge**
* Наблюдение за процессом осуществлялось с помощью **kubspy**
* Создан сервис Cluster IP
* Включен IPVS  в kube-proxy через minikube dashboard
* Очищены iptables в ВМ с minikube, установлен ipvsadm в ВМ
* Установлен MetalLB и настроен балансировщик с помощью **ConfigMap**
* Создан сервис **LoadBalancer** и применен манифест
* Просмотрены логи пода-контроллера MetalLB, определен назначенный IP-адрес
* Добавлен route через IP-адрес кластера
* Ссоздан сервис LoadBalancer, который позволил получать записи через внешний IP
* Создан Ingress c конфигом LoadBalancer
* Использован headless-сервис для веб-приложения
* Настроен ingress-прокси, справлен синтаксис в web-ingress
 

## Как запустить проект
---
* Создание pod с проверками readinessProbe и livenessProbe
 ```
  kubectl apply -f web-pod.yaml
  kubectl apply -f web-deploy.yaml

    readinessProbe:
        httpGet:
            path: /index.html
            port: 8000
    livenessProbe:
        tcpSocket: { port: 8000 }
```
* Добавлен блок startegy в манифест web-deploy с наблюдением за процессом
 ```
 kubectl get events --watch  

   strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 100%
      maxSurge: 0
 ```
* Создан сервис ClusterIP, найден кластерный IP
```
kubectl apply -f web-svc-cip.yaml
minikube ssh
sudo -i
iptables --list -nv -t nat
```
* Включен IPVS для kube-proxy, исправлен configmap, применена новая конфигурация
```
kubectl --namespace kube-system edit configmap/kube-proxy
minikube dashboard

kube-system - Configs - ConfigMaps:
    ipvs:
        strictARP: true
    mode: "ipvs"

kubectl --namespace kube-system delete pod --selector='k8s-app=kube-proxy'
```
* Очищены настройки iptables в ВМ Minikube, установлен ipvsadm
```
minikube ssh
touch /tmp/iptables.cleanup
iptables-restore /tmp/iptables.cleanup
apt update
apt install ipvsadm
ipvsadm --list -n
```
* Включен аддон MetalLB в minikube
```
minikube addons enable metallb
kubectl --namespace metallb-system get all
```
* Настроен **Load Balancer** в режиме L2 c созданием пула IP-адресов 172.17.255.1 - 172.17.255.255 + проверка конфига
```
kubectl apply -f metallb-config.yaml
kubectl apply -f web-svc-lb.yaml
kubectl --namespace metallb-system logs pod/controller-77868b5f54-dbgxf
```
* Добавлен статический маршрут в ВМ minikube, т.к. сеть кластера изолирована от основной ОС
```
minikube ssh
ip addr show eth0
ip route add 172.17.255.0/24 via 192.168.49.2
```
* Создан сервис LoadBalancer, открывающий доступ к **CoreDNS** снаружи кластера
```
kubectl apply -f coredns-svc-lb.yaml
nslookup web.default.cluster.local 172.17.255.2
```
* Включен ingress в minikube, создан манфиест nginx с конифгом Load Balancer
```
minikube addons enable ingress
kubectl apply -f nginx-lb.yaml
```
* Подключено веб-приложение к igress-контроллеру
```
cp web-svc-cip.yaml web-svc-headless.yaml

    clusterIP: None, 
    name: web-svc
```
* Настроен ingress-прокси, создан манифест с ресурсом Ingress + применен манифест и проверена корректность **Address** и **Backends**
```
kubectl apply -f web-ingress.yaml
kubectl describe ingress/web
```


## Как проверить работоспособность
---
* перейти по url - http://172.17.255.2
* curl http://172.17.255.2/web/index.html
* nslookup web.default.cluster.local 172.17.255.2

## PR checklist:
---
 - [x] Выставлен label с темой домашнего задания
