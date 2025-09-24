
## **Задание 3: Настройка RBAC**  
### **Задача**  
Создать пользователя с ограниченными правами (только просмотр логов и описания подов).

### **Шаги выполнения**  
1. **Включите RBAC в microk8s**
```bash
microk8s enable rbac
```
2. **Создать SSL-сертификат для пользователя**
```bash
openssl genrsa -out developer.key 2048
openssl req -new -key developer.key -out developer.csr -subj "/CN={ИМЯ ПОЛЬЗОВАТЕЛЯ}"
openssl x509 -req -in developer.csr -CA {CA серт вашего кластера} -CAkey {CA ключ вашего кластера} -CAcreateserial -out developer.crt -days 365
```
3. **Создать Role (только просмотр логов и описания подов) и RoleBinding**
4. **Проверить доступ**

### **Что сдать на проверку**  
- Манифесты:
  - `role-pod-reader.yaml`
  - `rolebinding-developer.yaml`
- Команды генерации сертификатов
- Скриншот проверки прав (`kubectl get pods --as=developer`)

---

## Решение 3

## Подготовка MicroK8s (без ingress)
### (один раз) добавить себя в группу и перелогиниться
sudo usermod -aG microk8s $USER
sudo chown -R $USER ~/.kube || true
newgrp microk8s

# старт и проверка
microk8s start
microk8s status --wait-ready

# нужны только dns, storage, rbac (ingress НЕ включаем)
microk8s enable dns storage rbac


## Базовое приложение: ConfigMap + Deployment + NodePort
### namespace
microk8s kubectl create ns app-demo

### применить манифесты
microk8s kubectl apply -f configmap.yaml
microk8s kubectl apply -f deployment.yaml
microk8s kubectl apply -f service-nodeport.yaml

### дождаться готовности
microk8s kubectl -n app-demo rollout status deploy/web-app

### посмотреть состояние
microk8s kubectl -n app-demo get all -o wide

## Проверка через NodePort (в браузере/через curl)
### IP ноды
NODE_IP=$(microk8s kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
echo "$NODE_IP"
ответ 10.0.85.1

### внешний порт сервиса (в манифесте 30080 — на всякий случай вытащим из кластера)
PORT=$(microk8s kubectl -n app-demo get svc web-svc -o jsonpath='{.spec.ports[0].nodePort}')
echo "$PORT"
ответ 30080


### проверка работоспобности
curl "http://$NODE_IP:$PORT/"
curl "http://10.0.85.1:30080/"

ответ 
```
<html>
  <head><title>K8s demo</title></head>
  <body>
    <h1>Hello from nginx on Kubernetes!</h1>
  </body>
</html>
```


## RBAC только просмотр Pod’ов и логов
## ключ и CSR для пользователя developer
openssl genrsa -out developer.key 2048
openssl req -new -key developer.key -out developer.csr -subj "/CN=developer"

# подписываем кластерным CA MicroK8s
sudo openssl x509 -req -in developer.csr \
  -CA /var/snap/microk8s/current/certs/ca.crt \
  -CAkey /var/snap/microk8s/current/certs/ca.key \
  -CAcreateserial -out developer.crt -days 365

# применить роли
microk8s kubectl apply -f role-pod-reader.yaml
microk8s kubectl apply -f rolebinding-developer.yaml

# быстрая проверка (имперсонация)
microk8s kubectl -n app-demo get pods --as=developer
microk8s kubectl -n app-demo logs deploy/web-app -c nginx --as=developer
microk8s kubectl -n app-demo auth can-i delete pods --as=developer   # должно быть "no"