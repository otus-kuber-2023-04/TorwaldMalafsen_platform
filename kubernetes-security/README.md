# Выполнено ДЗ №5
- [x] Основное ДЗ

## В процессе сделано
---
* Создан Service Account **bob**, выдана ему роль admin в рамках всего кластера
* Создан Service Account dave без доступа к кластеру
* Создан Namespace prometheus и создан Service Account **carol** в этом Namespace
* Выдана всем Service Account в Namespace prometheus возможность делать get, list, watch в отношении Pods всего кластера
* Создан Namespace dev и создан Service Account **jane** в Namespace dev
* Выдана **jane** роль admin в рамках Namespace dev
* Создан Service Account **ken** в Namespace dev
* Выдана **ken** роль view в рамках Namespace dev

## Как запустить проект
---
* Применим все манифесты из task01, task02 и task03
```
kubectl apply -f task01/
kubectl apply -f task02/
kubectl apply -f task03/
```
* Проверим создание сервисных аккаунтов, нэймспэйсов и ролей
```
task01:
kubectl get sa
kubectl describe sa bob
kubectl describe clusterrolebinding bob-admin
kubectl describe sa dave

task02:
kubectl get sa --namespace=prometheus
kubectl describe role/prom-pod-reader --namespace=prometheus

task03:
kubectl describe sa/ken --namespace=dev
kubectl describe rolebinding ken-view --namespace=dev
```

## PR checklist:
---
 - [x] Выставлен label с темой домашнего задания