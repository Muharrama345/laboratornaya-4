 Лабораторная 4. Replicated Service + Varnish Cache + Nginx SSL

 Архитектура системы
dictionary-server (3 реплики)
↓
varnish-cache (2 реплики)
↓
nginx-ssl (4 реплики + SSL)

Архитектура (https://i.imgur.com/example.png

| Файл | Описание |
|------|----------|
| `dictionary-deploy.yaml` | Deployment 3 реплик dictionary-server |
| `dictionary-service.yaml` | NodePort сервис для dictionary |
| `default.vcl` | Конфигурация Varnish |
| `varnish-deploy.yaml` | Deployment 2 реплик Varnish |
| `varnish-service.yaml` | ClusterIP сервис Varnish |
| `nginx.conf` | Nginx конфигурация с SSL |
| `nginx-deploy.yaml` | Deployment 4 реплик Nginx |
| `nginx-service.yaml` | LoadBalancer сервис Nginx SSL |

 Развертывание (по шагам)

 Часть 1: Dictionary Server
kubectl apply -f dictionary-deploy.yaml
kubectl get pods
kubectl apply -f dictionary-service.yaml
kubectl get services
kubectl get nodes -o wide
 curl http://<Node_IP>:<NodePort>/dog

 Часть 2: Varnish Cache
kubectl create configmap varnish-config --from-file=default.vcl
kubectl apply -f varnish-deploy.yaml
kubectl get pods
kubectl apply -f varnish-service.yaml
kubectl port-forward svc/varnish-service 8080:80
 curl localhost:8080/cat

 Часть 3: Nginx + SSL
openssl genrsa -out server.key 2048
openssl req -x509 -new -key server.key -days 365 -out server.crt
kubectl create secret tls ssl --cert=server.crt --key=server.key
kubectl create configmap nginx-conf --from-file=nginx.conf
kubectl apply -f nginx-deploy.yaml
kubectl get pods
kubectl apply -f nginx-service.yaml
kubectl port-forward svc/nginx-service 8443:443
 curl -k https://localhost:8443/horse
