# Выполнено ДЗ №4
- [x] Основное ДЗ
- [x] Задание secrets со *
## В процессе сделано
---
* Установлен kind
* Применена конфигурация **Stateful Set**
* Создан **Headless Service** для minio
* Проверена работа minio
* Создан манифест secrets для безопасного хранения секретов
* Дополнена конфигурация **Stateful Set**

## Как запустить проект
---
* Установим и запустим kind
```
brew install kind
kind create cluster
export KUBECONFIG="$(kind get kubeconfig-path --name="kind")"
```
* Применим конфиг Stateful Set с Minio - локальным S3 хранилищем
```
kubectl apply -f minio-statefulset.yaml
```
* Применим Headless Service для доступности statful set изнутри кластера
```
kubectl apply -f minio-headless-service.yaml
```
* Проверим работу Minio
```
kubectl get statefulsets
kubectl get pvc
kubectl get pods
kubectl describe pod/minio-0
```
* Применен манифест minio-secrets с секретами для minio + дополнен манифест minio-statefulset
```
kubectl apply -f minio-secrets.yaml
kubectl get secrets
kubectl apply -f minio-statefulset.yaml
```

## PR checklist:
---
 - [x] Выставлен label с темой домашнего задания