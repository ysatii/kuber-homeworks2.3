
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

### Подготовка MicroK8s (на основе задания 2 без ingress)
### (один раз) добавить себя в группу и перелогиниться
sudo usermod -aG microk8s $USER
sudo chown -R $USER ~/.kube || true
newgrp microk8s

### старт и проверка
microk8s start  
microk8s status --wait-ready  
![рисунок 15](https://github.com/ysatii/kuber-homeworks2.3/blob/main/img/img_15.jpg)  
### нужны только dns, storage, rbac (ingress НЕ включаем)
microk8s enable dns storage rbac


### Базовое приложение: ConfigMap + Deployment + NodePort
### namespace
microk8s kubectl create ns app-demo

### применим configmap, deployment, service-nodeport.
microk8s kubectl apply -f configmap.yaml  
microk8s kubectl apply -f deployment.yaml  
microk8s kubectl apply -f service-nodeport.yaml  

### нужно дождаться готовности
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
![рисунок 16](https://github.com/ysatii/kuber-homeworks2.3/blob/main/img/img_16.jpg)  

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
```
sudo openssl x509 -req -in developer.csr \
  -CA /var/snap/microk8s/current/certs/ca.crt \
  -CAkey /var/snap/microk8s/current/certs/ca.key \
  -CAcreateserial -out developer.crt -days 365
```

# применить роли
microk8s kubectl apply -f role-pod-reader.yaml
microk8s kubectl apply -f rolebinding-developer.yaml

# быстрая проверка (имперсонация)
microk8s kubectl -n app-demo get pods --as=developer
microk8s kubectl -n app-demo logs deploy/web-app -c nginx --as=developer
microk8s kubectl -n app-demo auth can-i delete pods --as=developer   # должно быть "no"


## Посмотреть логи nginx по Pod’у (без доступа к Deployment)
### получить имя Pod по метке (разрешено ролью)
microk8s kubectl -n app-demo get pods -l app=web-app --as=developer

![рисунок 17](https://github.com/ysatii/kuber-homeworks2.3/blob/main/img/img_17.jpg)  
### допустим, имя пода web-app-64c58d7d74-h5qlb:
microk8s kubectl -n app-demo logs pod/web-app-64c58d7d74-h5qlb -c nginx --as=developer

### Или сразу по селектору (тоже не трогает Deployment)
microk8s kubectl -n app-demo logs -l app=web-app -c nginx --as=developer --tail=100 --prefix=true
![рисунок 18](https://github.com/ysatii/kuber-homeworks2.3/blob/main/img/img_18.jpg)  





### Быстрая проверка прав (должно быть «yes» на pods и pods/log)
microk8s kubectl -n app-demo auth can-i get pods --as=developer
microk8s kubectl -n app-demo auth can-i get pods/log --as=developer
microk8s kubectl -n app-demo auth can-i delete pods --as=developer   # ожидаемо: no


### Листинги манифестов с сервера  
microk8s kubectl -n app-demo get all -o wide  

microk8s kubectl -n app-demo get configmap web-content -o yaml > _out_configmap.yaml  
microk8s kubectl -n app-demo get deploy web-app -o yaml > _out_deployment.yaml  
microk8s kubectl -n app-demo get svc web-svc -o yaml > _out_service.yaml  
microk8s kubectl -n app-demo get role pod-viewer -o yaml > _out_role.yaml  
microk8s kubectl -n app-demo get rolebinding developer-pod-view -o yaml > _out_rolebinding.yaml  
![рисунок 19](https://github.com/ysatii/kuber-homeworks2.3/blob/main/img/img_19.jpg)  


## Удаляем все что сделано  
microk8s kubectl delete ns app-demo  
microk8s kubectl get ns  
![рисунок 20](https://github.com/ysatii/kuber-homeworks2.3/blob/main/img/img_20.jpg)  

##Манифесты
### Ссылки на манифесты с сервера  
[_out_configmap.yaml](https://github.com/ysatii/kuber-homeworks2.3/blob/main/task3/_out_configmap.yaml)
[_out_deployment.yaml](https://github.com/ysatii/kuber-homeworks2.3/blob/main/task3/_out_deployment.yaml)
[_out_role.yaml](https://github.com/ysatii/kuber-homeworks2.3/blob/main/task3/_out_role.yaml)
[_out_rolebinding.yaml](https://github.com/ysatii/kuber-homeworks2.3/blob/main/task3/_out_rolebinding.yaml)
[_out_service.yaml](https://github.com/ysatii/kuber-homeworks2.3/blob/main/task3/_out_service.yaml)

### configmap
[configmap.yaml](https://github.com/ysatii/kuber-homeworks2.3/blob/main/task3/configmap.yaml)

### deployment
[deployment.yaml](https://github.com/ysatii/kuber-homeworks2.3/blob/main/task3/deployment.yaml)

## Сертификат
### developer.crt
[developer.crt](https://github.com/ysatii/kuber-homeworks2.3/blob/main/task3/developer.crt)
### developer.csr
[developer.csr](https://github.com/ysatii/kuber-homeworks2.3/blob/main/task3/developer.csr)
### developer.key
[developer.key](https://github.com/ysatii/kuber-homeworks2.3/blob/main/task3/developer.key)
    		
## Роли
### читает информацию про поды и логи  
[role-pod-reader.yaml](https://github.com/ysatii/kuber-homeworks2.3/blob/main/task3/role-pod-reader.yaml)  
[rolebinding-developer.yaml](https://github.com/ysatii/kuber-homeworks2.3/blob/main/task3/rolebinding-developer.yaml)  

### сервис nodeport
[service-nodeport.yaml](https://github.com/ysatii/kuber-homeworks2.3/blob/main/task3/service-nodeport.yaml)

 

 

	
 

 

	
 

	
 
 

	
 

	
 

	
 
l
	
 

	
 