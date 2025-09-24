## **Задание 2: Настройка HTTPS с Secrets**  
### **Задача**  
Развернуть приложение с доступом по HTTPS, используя самоподписанный сертификат.

### **Шаги выполнения**  
1. **Сгенерировать SSL-сертификат**
```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt -subj "/CN=myapp.example.com"
```
2. **Создать Secret**
3. **Настроить Ingress**
4. **Проверить HTTPS-доступ**

### **Что сдать на проверку**  
- Манифесты:
  - `secret-tls.yaml`
  - `ingress-tls.yaml`
- Скриншот вывода `curl -k`

---

## Решение 2

### Создадим пространство имен
kubectl create ns app-demo

### применяем конфигмап, подымает деплоемент и сервис ноде порт
kubectl apply -f configmap.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service-nodeport.yaml




### работаем с сертификатом 
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt -subj "/CN=myapp.example.com"

преобразуем сертиффикаты в base64
cat tls.crt | base64 -w0
cat tls.key | base64 -w0

![рисунок 6](https://github.com/ysatii/kuber-homeworks2.3/blob/main/img/img_6.jpg)  
kubectl apply -f secret-tls.yaml  

### подымаем ingress-tls контроллер
kubectl apply -f ingress-tls.yaml
kubectl -n app-demo get ingress web-ing

 Проверка
NodePort (браузер напрямую):
curl http://$(minikube ip):30080/
![рисунок 7](https://github.com/ysatii/kuber-homeworks2.3/blob/main/img/img_7.jpg)  


 
### Настроить hosts
echo "$(minikube ip) myapp.example.com" | sudo tee -a /etc/hosts
curl -vk https://myapp.example.com/

Узнать IP Minikube:
minikube ip
Добавить в /etc/hosts (нужны root-права):
echo "$(minikube ip) myapp.example.com" | sudo tee -a /etc/hosts

![рисунок 8](https://github.com/ysatii/kuber-homeworks2.3/blob/main/img/img_8.jpg)  
![рисунок 9](https://github.com/ysatii/kuber-homeworks2.3/blob/main/img/img_9.jpg)  
![рисунок 10](https://github.com/ysatii/kuber-homeworks2.3/blob/main/img/img_10.jpg)  
![рисунок 11](https://github.com/ysatii/kuber-homeworks2.3/blob/main/img/img_11.jpg)  
![рисунок 12](https://github.com/ysatii/kuber-homeworks2.3/blob/main/img/img_12.jpg)  
![рисунок 13](https://github.com/ysatii/kuber-homeworks2.3/blob/main/img/img_13.jpg)  
![рисунок 14](https://github.com/ysatii/kuber-homeworks2.3/blob/main/img/img_14.jpg)  


Манифесты 
[configmap.yaml](https://github.com/ysatii/kuber-homeworks2.3/blob/main/task2/configmap.yaml)  
[deployment.yaml](https://github.com/ysatii/kuber-homeworks2.3/blob/main/task2/deployment.yaml)  
[ingress-tls.yaml](https://github.com/ysatii/kuber-homeworks2.3/blob/main/task2/ingress-tls.yaml)  
[ingress-tls.yaml](https://github.com/ysatii/kuber-homeworks2.3/blob/main/task2/ingress-tls.yaml)  
[service-nodeport.yaml](https://github.com/ysatii/kuber-homeworks2.3/blob/main/task2/service-nodeport.yaml) 
 
Сертификаты
[tls.crt](https://github.com/ysatii/kuber-homeworks2.3/blob/main/task2/tls.crt) 
[tls.key](https://github.com/ysatii/kuber-homeworks2.3/blob/main/task2/tls.key) 


kubectl delete ns app-demo
kubectl get ns