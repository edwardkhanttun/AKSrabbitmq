# RabbitMQ Setup

### Install RabbitMQ on Kubernetes using Operator

Follow the document below to install the Kong Gateway Enterprise version on Kubernetes using helm.

https://docs.konghq.com/gateway/2.6.x/install-and-run/helm/
1. Install RabbitMQ Cluster Operator
```
kubectl apply -f "https://github.com/rabbitmq/cluster-operator/releases/latest/download/cluster-operator.yml"
```
2. Create namespace and rabbitmq cluster
```
kubectl create ns rabbitmq
kubectl apply -f rabbitmq.yaml -n rabbitmq
```
### Verify RabbitMQ is installed successfully
```
kubectl get all -n rabbitmq

NAME                    READY   STATUS    RESTARTS   AGE
pod/rabbitmq-server-0   1/1     Running   0          119s

NAME                     TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)                        AGE
service/rabbitmq         ClusterIP   10.97.4.122   <none>        5672/TCP,15672/TCP,15692/TCP   119s
service/rabbitmq-nodes   ClusterIP   None          <none>        4369/TCP,25672/TCP             119s

NAME                               READY   AGE
statefulset.apps/rabbitmq-server   1/1     119s

NAME                                    ALLREPLICASREADY   RECONCILESUCCESS   AGE
rabbitmqcluster.rabbitmq.com/rabbitmq   True               True               119s
```

### Login into the RabbitMQ Management UI
1. Obtain RabbitMQ Management login details
```
username="$(kubectl get secret rabbitmq-default-user -n rabbitmq -o jsonpath='{.data.username}'| base64 --decode)"

password="$(kubectl get secret rabbitmq-default-user -n rabbitmq -o jsonpath='{.data.password}'| base64 --decode)"

echo "$username" 
echo "$password" 
```
2. Access RabbitMQ Management UI (Nodeport)
```
kubectl apply -f rabbitmq-management-svc.yaml -n rabbitmq
kubectl get svc -n rabbitmq
NAME                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                        AGE
rabbitmq              ClusterIP   10.97.4.122      <none>        5672/TCP,15672/TCP,15692/TCP   10m
rabbitmq-management   NodePort    10.109.254.168   <none>        15672:31408/TCP                89s
rabbitmq-nodes        ClusterIP   None             <none>        4369/TCP,25672/TCP             10m
```
Access the management UI @ 172.20.105.64:31408
![Alt text](./images/1-managementui-login.png?raw=true "Title")

3. Access RabbitMQ Management UI (Ingress)
```
kubectl apply -f rabbitmq-ingress.yaml -n rabbitmq
```
Access the management UI @ 172.20.105.213/rabbitmq/

### RabbitMQ Configuration
Access the management UI @ 172.20.105.64:31408 OR 172.20.105.213/rabbitmq/
1. Create admin account
Admin > User > Create new user 
Username: mctadmin
Password: CloudEnabled
Tags: Admin
![Alt text](./images/2-admin-user-creation.png?raw=true "Title")

2. Create user accounts
Admin > User > Create new user 
Username: order-app
Password: PAssw0rd
Tags: Management
![Alt text](./images/2-user-creation.png?raw=true "Title")

3. Setup virtual host
Admin > Virtual Hosts > Add a new virtual host
Name: CPPS
![Alt text](./images/4-virtualhost.png?raw=true "Title")
