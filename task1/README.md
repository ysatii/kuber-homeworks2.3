
## **Задание 1: Работа с ConfigMaps**
### **Задача**
Развернуть приложение (nginx + multitool), решить проблему конфигурации через ConfigMap и подключить веб-страницу.

### **Шаги выполнения**
1. **Создать Deployment** с двумя контейнерами
   - `nginx`
   - `multitool`
3. **Подключить веб-страницу** через ConfigMap
4. **Проверить доступность**

### **Что сдать на проверку**
- Манифесты:
  - `deployment.yaml`
  - `configmap-web.yaml`
- Скриншот вывода `curl` или браузера

---

## Решение 1

### Создадим пространство имен app-demo
kubectl create ns app-demo

### применим configmap
kubectl apply -f configmap.yaml
kubectl -n app-demo get configmap web-content -o yaml

### создадим под с контейнерами nginx и multitool
kubectl apply -f deployment.yaml
kubectl -n app-demo rollout status deploy/web-app
kubectl -n app-demo get pods -o wide

### проверим что под nginx в работе
POD=$(kubectl -n app-demo get pod -l app=web-app -o jsonpath='{.items[0].metadata.name}')
kubectl -n app-demo exec -it "$POD" -c multitool -- sh -lc '
  curl -sI http://127.0.0.1:80/ | head -n1;
  echo "----";
  curl -s http://127.0.0.1:80/
'

получили ответ от контейнера nginx! все в работе!

### создадим сервис для nginx
kubectl apply -f service.yaml
kubectl -n app-demo get svc web-svc

### создадим порт форвард что ты проверить ответ в браузере
kubectl -n app-demo port-forward svc/web-svc 8080:80
# в браузере
localhost:8080/

### можем провести проверку создав сервис типа nodeport
kubectl apply -f service-nodeport.yaml
kubectl -n app-demo get svc web-svc-nodeport
kubectl get nodes -o wide
192.168.49.2

curl http://192.168.49.2:30080/



kubectl delete ns app-demo
kubectl get ns