# RabbitMQ Setup

### Install RabbitMQ on Kubernetes using Operator

Follow the document below to install the RabbitMQ on Kubernetes using operator.
https://www.rabbitmq.com/kubernetes/operator/quickstart-operator.html

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
![Alt text](./images/1-managementui-login.PNG?raw=true "Title")

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
![Alt text](./images/3-user-creation.png?raw=true "Title")

3. Setup virtual host
Admin > Virtual Hosts > Add a new virtual host
Name: CPPS
![Alt text](./images/4-virtualhost.png?raw=true "Title")

4. Give virtual host permission to users
Click on the virtual host
![Alt text](./images/5-cpps-VH.png?raw=true "Title")

Under Permissions > Set permission, select the user and give it permission to the current virtual host
![Alt text](./images/6-VH-permission.png?raw=true "Title")


### Connecting to RabbitMQ
#### Connecting from within the same Kubernetes Cluster
https://www.rabbitmq.com/dotnet-api-guide.html#connecting

You can either connect through amqp
```
ConnectionFactory factory = new ConnectionFactory();
factory.Uri = new Uri("amqp://order-app:PAssw0rd@rabbitmq.rabbitmq:5672/cpps");

```
> Take note that for amqp connection, special characters need to be percent encoded 
https://en.wikipedia.org/wiki/Percent-encoding

or defining the variables separately
```
ConnectionFactory factory = new ConnectionFactory();
factory.UserName = order-app;
factory.Password = PAssw0rd;
factory.HostName = rabbitmq.rabbitmq;
factory.Port = 5672;
factory.VirtualHost = cpps;
```


#### Connecting from outside Kubernetes Cluster
This includes connectons from frontend angular services within kubernetes cluster

https://www.rabbitmq.com/dotnet-api-guide.html#connecting
You can either connect through amqp
```
ConnectionFactory factory = new ConnectionFactory();
factory.Uri = new Uri("amqp://order-app:PAssw0rd@172.20.105.213%2Frabbitmq%2Fserver/cpps");

```
> Take note that for amqp connection, special characters need to be percent encoded 
https://en.wikipedia.org/wiki/Percent-encoding

or defining the variables separately
```
ConnectionFactory factory = new ConnectionFactory();
factory.UserName = order-app;
factory.Password = PAssw0rd;
factory.HostName = 172.20.105.213/rabbitmq/server;
factory.VirtualHost = cpps;
```


