# Complete Application Deployment using Kubernetes Components

[Complete Application Deployment using Kubernetes Components | Kubernetes Tutorial 20](https://www.youtube.com/watch?v=EQNO_kM96Mo&t=411s)

# Overview Diagram of Kubernetes Components we create

---

![Untitled](Complete%20Application%20Deployment%20using%20Kubernetes%20C%209d35aab90f5c46cd8611fb76b893ce4a/Untitled.png)

# Browser Request Flow

![Untitled](Complete%20Application%20Deployment%20using%20Kubernetes%20C%209d35aab90f5c46cd8611fb76b893ce4a/Untitled%201.png)

# MongoDB Deployment

---

### mongo.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb-deployment
  labels:
    app: mongodb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongodb
        image: mongo
        ports:
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          value:
        - name: MONGO_INITDB_ROOT_PASSWORD
          value:
```

create secret for deployment.yaml

![Untitled](Complete%20Application%20Deployment%20using%20Kubernetes%20C%209d35aab90f5c46cd8611fb76b893ce4a/Untitled%202.png)

# Secret

---

### mongo-secret.yaml

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mongodb-secret
type: Opaque
data:
  mongo-root-username: dXNlcm5hbWU=
  mongo-root-password: cGFzc3dvcmQ=
```

Before the referring secret from mongo-secret.yaml, we need to apply secret.ymal

```bash
kubectl apply -f mongo-secret.yaml
$ secret/mongodb-secret created
```

```bash
kubectl get secret

AME             TYPE     DATA   AGE
mongodb-secret   Opaque   2      38s
```

Secret can be referenced now in Deployment

### mongo.yaml with secret

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb-deployment
  labels:
    app: mongodb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongodb
        image: mongo
        ports:
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: mongo-root-username
        - name: MONGO_INITDB_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: mongo-root-password
```

```bash
% kubectl apply -f mongo.yaml
% kubectl describe pod mongodb-deployment-844789cd64-w58cm

% kubectl get pod -o wide
NAME                                  READY   STATUS    RESTARTS       AGE    IP           NODE       NOMINATED NODE   READINESS GATES
hello-minikube-65dc654df9-p6hhq       1/1     Running   4 (176m ago)   17h    172.17.0.5   minikube   <none>           <none>
mongodb-deployment-844789cd64-w58cm   1/1     Running   0              74m    172.17.0.6   minikube   <none>           <none>
python-webapp-59758d56b7-6n5j8        1/1     Running   0              177m   172.17.0.4   minikube   <none>           <none>
python-webapp-59758d56b7-rgz6d        1/1     Running   0              177m   172.17.0.2   minikube   <none>           <none>
```

```bash
Name:             mongodb-deployment-844789cd64-w58cm
Namespace:        default
Priority:         0
Service Account:  default
Node:             minikube/192.168.49.2
Start Time:       Sat, 08 Oct 2022 14:32:32 +0800
Labels:           app=mongodb
                  pod-template-hash=844789cd64
Annotations:      <none>
Status:           Running
IP:               172.17.0.6
IPs:
  IP:           172.17.0.6
Controlled By:  ReplicaSet/mongodb-deployment-844789cd64
Containers:
  mongodb:
    Container ID:   docker://2502c1366fd89db4a7c00a2f44347c4381385e36e13a701a311a199f6c3cb58b
    Image:          mongo
    Image ID:       docker-pullable://mongo@sha256:2ca8fb22c9522b49fd1f5490dee3e7026a4331b9f904d5acf10a9638c1d1539d
    Port:           27017/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sat, 08 Oct 2022 14:32:52 +0800
    Ready:          True
    Restart Count:  0
    Environment:
      MONGO_INITDB_ROOT_USERNAME:  <set to the key 'mongo-root-username' in secret 'mongodb-secret'>  Optional: false
      MONGO_INITDB_ROOT_PASSWORD:  <set to the key 'mongo-root-password' in secret 'mongodb-secret'>  Optional: false
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-g6rtl (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-g6rtl:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:                      <none>
```

# Internal Service for MongoDB

---

![Untitled](Complete%20Application%20Deployment%20using%20Kubernetes%20C%209d35aab90f5c46cd8611fb76b893ce4a/Untitled%203.png)

![Untitled](Complete%20Application%20Deployment%20using%20Kubernetes%20C%209d35aab90f5c46cd8611fb76b893ce4a/Untitled%204.png)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb-deployment
  labels:
    app: mongodb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongodb
        image: mongo
        ports:
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: mongo-root-username
        - name: MONGO_INITDB_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: mongo-root-password

---
apiVersion: v1
kind: Service
metadata: 
  name: mongodb-service
spec:
  selector:
    app: mongodb
  ports:
    - protocol: TCP
      port: 27017
      targetPort: 27017
```

```bash
% kubectl apply -f mongo.yaml

% kubectl get service
NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
face-detection-by-opencv   NodePort    10.107.165.112   <none>        5500:30490/TCP   162m
hello-minikube             NodePort    10.109.79.71     <none>        80:32560/TCP     17h
kubernetes                 ClusterIP   10.96.0.1        <none>        443/TCP          18h
mongodb-service            ClusterIP   10.109.118.255   <none>        27017/TCP        87s
```

```bash
% kubectl describe service mongodb-service
Name:              mongodb-service
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=mongodb
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.109.118.255
IPs:               10.109.118.255
Port:              <unset>  27017/TCP
TargetPort:        27017/TCP
Endpoints:         172.17.0.6:27017
Session Affinity:  None
Events:            <none>
```

```bash
% kubectl get pod -o wide 
NAME                                  READY   STATUS    RESTARTS       AGE    IP           NODE       NOMINATED NODE   READINESS GATES
hello-minikube-65dc654df9-p6hhq       1/1     Running   4 (3h6m ago)   17h    172.17.0.5   minikube   <none>           <none>
mongodb-deployment-844789cd64-w58cm   1/1     Running   0              84m    172.17.0.6   minikube   <none>           <none>
```

```bash
% kubectl get all | grep mongodb
pod/mongodb-deployment-844789cd64-w58cm   1/1     Running   0              88m
service/mongodb-service            ClusterIP   10.109.118.255   <none>        27017/TCP        6m34s
deployment.apps/mongodb-deployment   1/1     1            1           88m
replicaset.apps/mongodb-deployment-844789cd64   1         1         1       88m
```

# MongoExpress Deployment

---

![Untitled](Complete%20Application%20Deployment%20using%20Kubernetes%20C%209d35aab90f5c46cd8611fb76b893ce4a/Untitled%205.png)

**What is mongo-express?**

mongo-express is a web-based MongoDB admin interface written in Node.js, Express.js, and Bootstrap3.

[mongo-express - Official Image | Docker Hub](https://hub.docker.com/_/mongo-express)

![Untitled](Complete%20Application%20Deployment%20using%20Kubernetes%20C%209d35aab90f5c46cd8611fb76b893ce4a/Untitled%206.png)

### mongo-express.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-express
  labels:
    app: mongo-express
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongo-express
  template:
    metadata:
      labels:
        app: mongo-express
    spec:
      containers:
      - name: mongo-express
        image: mongo-express
        ports:
        - containerPort: 8081
        env:
        - name: ME_CONFIG_MONGODB_ADMINUSERNAME
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: mongo-root-username
        - name: ME_CONFIG_MONGODB_ADMINPASSWORD
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: mongo-root-password
        - name: ME_CONFIG_MONGODB_SERVER
          value: pending
```

![Untitled](Complete%20Application%20Deployment%20using%20Kubernetes%20C%209d35aab90f5c46cd8611fb76b893ce4a/Untitled%207.png)

### ConfigMap for mongo-express.yaml deployment

![Untitled](Complete%20Application%20Deployment%20using%20Kubernetes%20C%209d35aab90f5c46cd8611fb76b893ce4a/Untitled%208.png)

### mongo-configmap.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mongodb-configmap
data:
  database_url: mongodb-service
```

![Untitled](Complete%20Application%20Deployment%20using%20Kubernetes%20C%209d35aab90f5c46cd8611fb76b893ce4a/Untitled%209.png)

```bash
% kubectl apply -f mongo-configmap.yaml
% kubectl apply -f mongo-express.yaml
```

```bash
% kubectl logs mongo-express-5bf4b56f47-zpqnm
Welcome to mongo-express
------------------------

(node:7) [MONGODB DRIVER] Warning: Current Server Discovery and Monitoring engine is deprecated, and will be removed in a future version. To use the new Server Discover and Monitoring engine, pass option { useUnifiedTopology: true } to the MongoClient constructor.
Mongo Express server listening at http://0.0.0.0:8081
Server is open to allow connections from anyone (0.0.0.0)
basicAuth credentials are "admin:pass", it is recommended you change this in your config.js!
```

# MongoExpress External Service

Make mogo-express accessible by browser, we need an external service for mogo-express

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-express
  labels:
    app: mongo-express
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongo-express
  template:
    metadata:
      labels:
        app: mongo-express
    spec:
      containers:
      - name: mongo-express
        image: mongo-express
        ports:
        - containerPort: 8081
        env:
        - name: ME_CONFIG_MONGODB_ADMINUSERNAME
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: mongo-root-username
        - name: ME_CONFIG_MONGODB_ADMINPASSWORD
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: mongo-root-password
        - name: ME_CONFIG_MONGODB_SERVER
          valueFrom:
            configMapKeyRef:
              name: mongodb-configmap
              key: database_url
---
apiVersion: v1
kind: Service
metadata:
  name: mongo-expression-service
spec:
  selector:
    app: mongo-express
  type: LoadBalancer
  ports:
    - protocol: TCP
      port: 8081
      targetPort: 8081
      nodePort: 30000
```

![Untitled](Complete%20Application%20Deployment%20using%20Kubernetes%20C%209d35aab90f5c46cd8611fb76b893ce4a/Untitled%2010.png)

```bash
% kubectl apply -f mongo-express.yaml
```

![Untitled](Complete%20Application%20Deployment%20using%20Kubernetes%20C%209d35aab90f5c46cd8611fb76b893ce4a/Untitled%2011.png)

we didn’t define mongodb-service as cluster IP type becasue cluster IP which is the same as an internal service type is default. You don’t have to define when you’re creating internal service.

The difference is that cluster IP will  give the service an internal IP address. Shown as above, 10.96.86.105 is the external IP address, and the load banlancer will also give services an internal IP address but in addition to that it will also give the service an external IP address where the request will be coming from.

![Untitled](Complete%20Application%20Deployment%20using%20Kubernetes%20C%209d35aab90f5c46cd8611fb76b893ce4a/Untitled%2012.png)

Ask minikube to assign an external IP for us:

```bash
minikube service mongo-expression-service
```

```bash
(mlp) nelsonlin@macbook devops-mongo-db-k8s-tutorial % minikube service mongo-expression-service      
|-----------|--------------------------|-------------|---------------------------|
| NAMESPACE |           NAME           | TARGET PORT |            URL            |
|-----------|--------------------------|-------------|---------------------------|
| default   | mongo-expression-service |        8081 | http://192.168.49.2:30000 |
|-----------|--------------------------|-------------|---------------------------|
🏃  Starting tunnel for service mongo-expression-service.
|-----------|--------------------------|-------------|------------------------|
| NAMESPACE |           NAME           | TARGET PORT |          URL           |
|-----------|--------------------------|-------------|------------------------|
| default   | mongo-expression-service |             | http://127.0.0.1:62480 |
|-----------|--------------------------|-------------|------------------------|
🎉  Opening service default/mongo-expression-service in default browser...
❗  Because you are using a Docker driver on darwin, the terminal needs to be open to run it.
```

![Untitled](Complete%20Application%20Deployment%20using%20Kubernetes%20C%209d35aab90f5c46cd8611fb76b893ce4a/Untitled%2013.png)