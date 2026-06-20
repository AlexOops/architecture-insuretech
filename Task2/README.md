# Task2 — Динамическое масштабирование контейнеров

## Цель

Настроить динамическое масштабирование тестового приложения в Kubernetes с помощью Horizontal Pod Autoscaler.

## Что реализовано

- Deployment для `scaletestapp`.
- Service для доступа к приложению.
- HPA по memory utilization.
- Locust-сценарий для генерации нагрузки.
- Логи/скриншоты проверки масштабирования.

## Используемый образ

```
ghcr.io/yandex-practicum/scaletestapp:latest
```

## Параметры Deployment
- начальное количество реплик: 1
- memory request: 20Mi
- memory limit: 30Mi
- container port: 8080

## Параметры HPA
- minReplicas: 1
- maxReplicas: 10
- metric: memory
- target averageUtilization: 80%

## Запуск

```bash 
minikube start --driver=docker --memory=4096 --cpus=2
minikube addons enable metrics-server

kubectl apply -f Task2/manifests/deployment.yaml
kubectl apply -f Task2/manifests/service.yaml
kubectl apply -f Task2/manifests/hpa.yaml
```

## Получение URL приложения
```bash 
minikube service scaletestapp-service --url
```

## Проверка
```bash 
kubectl get pods
kubectl get deployment scaletestapp
kubectl get hpa scaletestapp-hpa
kubectl top pods
```

## Запуск нагрузки
```bash 
locust -f Task2/locustfile.py
```
После запуска открыть Locust UI:
```bash 
http://localhost:8089
```
В поле Host указать URL из команды:
```bash 
minikube service scaletestapp-service --url
```
## Результат
После генерации нагрузки количество реплик должно увеличиться автоматически. Для подтверждения используются:

- kubectl get hpa
- kubectl get pods
- kubectl describe hpa scaletestapp-hpa
- скриншоты или текстовые логи в директории Task2/logs
