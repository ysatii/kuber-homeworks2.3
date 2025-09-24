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



### преобразуем сертиффикаты в base64
cat tls.crt | base64 -w0
cat tls.key | base64 -w0

![рисунок 6](https://github.com/ysatii/kuber-homeworks2.3/blob/main/img/img_6.jpg)
kubectl apply -f secret-tls.yaml


kubectl apply -f ingress-tls.yaml
kubectl -n app-demo get ingress web-ing

 Проверка

NodePort (браузер напрямую):

curl http://$(minikube ip):30080/





Ingress (HTTPS):

echo "$(minikube ip) myapp.example.com" | sudo tee -a /etc/hosts
curl -vk https://myapp.example.com/

Настроить hosts

Узнать IP Minikube:

minikube ip


Добавить в /etc/hosts (нужны root-права):
echo "$(minikube ip) myapp.example.com" | sudo tee -a /etc/hosts