# kubernetes services usage
 * [PODS](#pods)
 * [Replication Controller](#replication-controller)
 * [ReplicaSet](#replicaset)
 * [Namespaces](#namespaces)
 * [Deployments](#deployments)
 * [Config Maps](#config-maps)
 * [Resource Quota](#resource-quota)
 * [Env variables in Kubernetes](#environment-variables)
 * [Secrets](#secrets)
 * [Encode and decode strings in Linux](#encode-and-decode-strings-in-linux)
 * [Security Contexts](#security-contexts)
 * [Service Accounts in Kubernetes](#service-accounts)
 * [Resource Requirements](#resource-requirements)
 
 ### PODs
 ----------
 ```YAML
apiVersion: v1
kind: Pod

metadata:
  name: myapp-pod
  labels:
    app:  myapp
    costcenter: us-east1
    location: NA
    type: frontend

spec:
  containers:
    - name: nginx-container
      image: nginx
 ```
 
  ### Replication Controller
 ----------
      Each container encapsulated with pod, multiple pods deployed using replica controller/ replicaset.
      kubectl get replicationcontroller - to get/list all replicationcontrollers
      kubectl create -f replicacontroller-definition-1.yml ( create replicacontroller)
 ```YAML
 apiVersion: v1
kind: ReplicationController

metadata:
  name: myapp-rc
  labels:
    app:  myapp
    costcenter: america
    location: NA
    type: frontend

spec:
  template:

    metadata:
     name: myapp-rc
     labels:
        app:  myapp
        costcenter: america
        location: NA
        type: frontend
    spec:
      containers:
      - name: nginx-container
        image: nginx
  replicas: 3
 ```
   ### Replicaset
 ----------
    Replicaset automatically creates pods
	Each container encapsulated with pod, multiple pods deployed using replica controller/ replicaset.
```YAML
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-1
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx
```
### Namespaces
----------
 ```YAML
apiVersion: v1
kind: Namespace
metadata:
  name: dev
 ```
### Deployments
----------
	Kubernetes deployment automatically creates replicasets
	Rolling update: Update applications/ deployments one by one without impacting users is called rolling update.
	Rollback changes
	pause changes
	resume changes
	Replicaset/ replicatecontroller is inside deployment.
```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-2
spec:
  replicas: 5
  selector:
    matchLabels:
      tier: nginx
  template:
    metadata:
      labels:
        tier: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
```
### Config Maps
----------
There are two ways to create config maps. Imperative way and Declarative way
#### Imperative 
	kubectl create configmap <<configmap-name>> --from-literal=<key>=<value>
	kubectl create configmap app-config --from-literal=APP_COLOR=blue
	kubectl create configmap app-config --from-literal=APP_COLOR=blue --from-literal=APP_NAME=sample
	kubectl create configmap app-config --from-file=<<file path>>
	kubectl create configmap app-config --from-file=app-config.properties
#### Declarative
```YAML
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: red
  APP_MODE: prod
```
	kubectl create -f config-map.yml
	kubectl describe configmap app-config
	kubectl edit configmap <<configmap Name>>
**When we have lot of pod definition data and environment data on pod is difficult. We can use configMap**
### Config Maps in POD
First create a config map and then assign it on all required pods
```YAML
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app:  myapp
    costcenter: america
    location: NA
    type: frontend
spec:
  containers:
    - name: nginx-container
      image: nginx
      ports:
        - containerPort: 8080
      envFrom:
       - configMapRef:
                 name: app-config
```
**We can also inject single env variables**
```YAML
apiVersion: v1
kind: Pod
metadata:
  name: mypod
  labels:
    type: frontend
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 8080
      env:
        - name: APP_COLOR
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_COLOR
```
### Resource Quota
To limit resource in namespaces we can use the resource quota.

compute-quota.yml
```YAML
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
      pods: "10"
      requests.cpu: "4"
      requests.memory: 5Gi
      limits.cpu: "10"
      limits.memory: 10Gi
```    
**Note:**
 * 1G is Gigabyte (1,000,000,000 bytes) and 1Gi means Gibibyte (1,073,741,824 bytes)
 * 1M is Megabyte (1,000,000 bytes) and 1Mi means Mebibyte (1,048,576 bytes)
 * 1k is Kilobyte (1,000 bytes) and 1Ki means Kibibyte (1,024 bytes)
### Environment variables
    docker run -e APP_COLOR=pink simple-webapp-color
#### Environment variables inside POD (key-value method)
```YAML
apiVersion: v1
kind: Pod
metadata:
  name: mypod
  labels:
    type: frontend
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 8080
      env:
        - name: APP_COLOR
          value: pink   
```
#### Environment variables inside POD (configMap method)
```YAML
apiVersion: v1
kind: Pod
metadata:
  name: mypod
  labels:
    type: frontend
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 8080
      env:
        - name: APP_COLOR
          valueFrom:
             configMapKeyRef: 
```
#### Environment variables inside POD (Secrets method)
```YAML
apiVersion: v1
kind: Pod
metadata:
  name: mypod
  labels:
    type: frontend
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 8080
      env:
        - name: APP_COLOR
          valueFrom:
             secretKeyRef: 
```
### Secrets
Secrets are used to store sensitive information like passwords and keys.
It is similar to configMap but it is in encoded/ hashed format. 
We can create the secrets in 2 ways. Imperative and Declarative way
#### Imperative way
     kubectl create secret generic <<secretName>>
     kubectl create secret generic <<secretName>> --from-literal=<key>=<value>
     kubectl create secret generic app-secret--from-literal=DB_HOST=mysql \
     kubectl create secret generic <<secretName>> --from-file=<path-to-file>
     kubectl create secret generic <<secretName>> --from-file=app_secret.properties
#### Declarative way
```YAML
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
   DB_HOST: mysql  #supports only encoded strings
   DB_NAME: root   #supports only encoded strings
   DB_PASS: passwd #supports only encoded strings
```
### Encode and decode strings in Linux
##### encode text 
     echo -n 'mysql' | base64
     echo -n 'root' | base64
     echo -n 'passwd' | base64
	
##### decode text 
     echo -n 'bXlzcWw=' | base64 --decode
     
### Security Contexts
    In kubernetes, Containers are encapsulated in pods
    docker run --user=1001 ubuntu sleep 3600
    docker run --cap-add MAC_ADMIN ubuntu
**Note: Capabilities are only supported at the container level and not at the POD level**
```YAML
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app:  myapp
    costcenter: america
    location: NA
    type: frontend
spec:
  containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep", "3600"]
      securityContext:
        runAsUser: 1000
        capabilities:
           add: ["MAC_ADMIN"]
```
### Service Accounts
	Two types of accounts in kubernetes
		- User accounts (admin, user) used by humans
		- service account used by machines/ applications (Jenkins, Flask and etc)
	Each namespace have default serviceaccount
         kubectl create serviceaccount dashboard-sa
	kubectl get serviceaccount
	kubectl describe serviceaccount dashboard-sa
	kubectl describe secret <<secret>>
**Create serviceaccount -> Assign role based permissions -> Export serviceaccount token -> use it on third party app**
What if third party app deployed on kubernetes itself, then we can simply mount serviceaccount on third party app
Create serviceaccount -> Assign role based permissions -> Mount serviceaccount on third party app
**Use serviceaccount in pod definition**
```YAML
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app:  myapp
    costcenter: america
    location: NA
    type: frontend

spec:
  containers:
    - name: nginx-container
      image: nginx
  serviceAccount: dashboard-sa
```
### Resource Requirements
    Minimum requirements for each container
    CPU | MEM   | Disk
    0.5 | 256Mi | 
```YAML
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app:  myapp
    costcenter: america
    location: NA
    type: frontend

spec:
  containers:
    - name: nginx-container
      image: nginx
          ports:
            - containerPort: 8080
          resources:
            requests:
                memory: "1Gi"
                cpu: 1  # 1 mean 1 vCPU in AWS and 1 core in GCP and Azure
          limits:   # we can set the resource usage in limits section
          memory: "2Gi"
          cpu: 2
```
**Note:**
 * 1G is Gigabyte (1,000,000,000 bytes) and 1Gi means Gibibyte (1,073,741,824 bytes)
 * 1M is Megabyte (1,000,000 bytes) and 1Mi means Mebibyte (1,048,576 bytes)
 * 1k is Kilobyte (1,000 bytes) and 1Ki means Kibibyte (1,024 bytes)
