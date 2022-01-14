<img src="https://kubernetes.io/images/kubernetes-horizontal-color.png" width="500px"/><br/>

# **📝 Certified Kubernetes Application Developer (CKAD) Practice Exams**
### 📖 **Author:** **`Ken Yew Piong`**
### 📆 **Last Modified:**  <img src="https://img.shields.io/badge/dynamic/json?style=flat-square&labelColor=0039A9&color=027DFF&label=UTC&query=currentDateTime&url=http%3A%2F%2Fworldclockapi.com%2Fapi%2Fjson%2Futc%2Fnow&logo=AzureDevOps&logoColor=3399FF"/>
<a href="https://github.com/KenYew">
  <img src="https://img.shields.io/badge/GitHub-black?style=social&logo=GitHub"/>
</a>
<a href="https://gitlab.com/KenYew">
  <img src="https://img.shields.io/badge/GitLab-black?style=social&logo=GitLab"/>
</a>

---
# <div id='toc'/> 📋 **Table of Contents** 
1. ### [📓 **Exam Syllabus**](#syllabus)
0. ### [🧬 **Lightning Lab 1**](#lab1)
0. ### [🧪 **Lightning Lab 2**](#lab2)
0. ### [📕 **Mock Exam 1**](#exam1)
0. ### [📗 **Mock Exam 2**](#exam2)

---
# <div id='syllabus'/> **📓 Certified Kubernetes Application Developer (CKAD)  Exam Syllabus**

## **20% - 📱 Application Design and Build**
- Define, build and modify container images
- Understand Jobs and CronJobs
- Understand multi-container Pod design patterns (e.g. sidecar, init and others)
- Utilize persistent and ephemeral volumes

## **20% - 📲 Application Deployment**
- Use Kubernetes primitives to implement common deployment strategies (e.g. blue/ green or canary)
- Understand Deployments and how to perform rolling updates
- Use the Helm package manager to deploy existing packages

## **25% - 🌥 Application Environment, Configuration and Security**
- Discover and use resources that extend Kubernetes (CRD)
- Understand authentication, authorization and admission control
- Understanding and defining resource requirements, limits and quotas
- Understand ConfigMaps
- Create & consume Secrets
- Understand ServiceAccounts
- Understand SecurityContexts
## **15% - 🖥 Application Observability and Maintenance**
- Understand API deprecations
- Implement probes and health checks
- Use provided tools to monitor Kubernetes applications
- Utilize container logs
- Debugging in Kubernetes
## **20% - 🌍 Services & Networking**
- Demonstrate basic understanding of NetworkPolicies
- Provide and troubleshoot access to applications via services
- Use Ingress rules to expose applications

---
# <div id='lab1'/> 🧬 **Lightning Lab 1**
### **Question 1: Persistent Volumes and Persistent Volume Claims**
- [x] Create a Persistent Volume called log-volume. 
- ✅ Storage class name: `manual`
- ✅ Access mode: `RWX`
- ✅ Size: `1Gi` 
- ✅ hostPath: `/opt/volume/nginx`
- [x] Create a PVC called log-claim. 
- ✅ Requesting a minimum of `200Mi` of storage
- ✅ PVC should bind to `log-volume`
- [x] Mount volume in a pod called logger.
- ✅ MountPath: `/var/www/nginx` 
- ✅ Pod image: `nginx:alpine`

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: log-volume
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  storageClassName: manual
  hostPath:
    path: /opt/volume/nginx
then create a Persistent Volume Claim called log-claim as follows:-

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: log-claim
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 200Mi
  storageClassName: manual

# Check the bind status of PV and PVC by running the following command:-
root@controlplane:~$ kubectl get pv,pvc

# Now, create a new pod called logger with nginx:alpine image as follows:-
---
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: logger
# pod name
  name: logger
spec:
  containers:
  - image: nginx:alpine
    name: logger
    volumeMounts:
    - name: log
      mountPath: /var/www/nginx
  volumes:
  - name: log
    persistentVolumeClaim:
        claimName: log-claim
```
---
### **Question 2: Troubleshooting Network Policies**
- [x] We have deployed a new pod called secure-pod and a service called secure-service. Incoming or Outgoing connections to this pod are not working.
- [x] Troubleshoot why this is happening.
- [x] Make sure that incoming connection from the pod webapp-color are successful.
Important: Don't delete any current objects deployed.

```yaml
# Incoming or outgoing connections are not working because of network policy. In the default namespace, we deployed a default-deny network policy which is interrupting the incoming or outgoing connections.

# Now, create a network policy called test-network-policy to allow the connections, as follows:-
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: secure-pod
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: webapp-color
    ports:
    - protocol: TCP
      port: 80

# then check the connectivity from the webapp-color pod to the secure-pod:-
root@controlplane:~$ kubectl exec -it webapp-color -- sh
/opt # nc -v -z -w 5 secure-service 80
```
---

### **Question 3: Executing Commands in Pods**
- [x] Create a pod called time-check in the `dvl1987` namespace. This pod should run a container called time-check that uses the busybox image.
- [x] Create a config map called time-config with the data `TIME_FREQ=10` in the same namespace.
- [x] The time-check container should run the command: `while true; do date; sleep $TIME_FREQ;done` and write the result to the location `/opt/time/time-check.log`
- [x] The path `/opt/time` on the pod should mount a volume that lasts the lifetime of this pod.

```yaml
# Create a namespace called dvl1987 by using the below command:-
$ kubectl create namespace dvl1987

# Solution manifest file to create a configMap called time-config in the given namespace as follows:-
apiVersion: v1
data:
  TIME_FREQ: "10"
kind: ConfigMap
metadata:
  name: time-config
  namespace: dvl1987

# Now, create a pod called time-check in the same namespace as follows:-
---
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: time-check
  name: time-check
  namespace: dvl1987
spec:
  volumes:
  - name: log-volume
    emptyDir: {}
  containers:
  - image: busybox
    name: time-check
    env:
    - name: TIME_FREQ
      valueFrom:
            configMapKeyRef:
              name: time-config
              key: TIME_FREQ
    volumeMounts:
    - mountPath: /opt/time
      name: log-volume
    command:
    - "/bin/sh"
    - "-c"
    - "while true; do date; sleep $TIME_FREQ;done > /opt/time/time-check.log"
```
---
### **Question 4: Deployments, Rollouts and Rollbacks**
- [x] Create a new deployment called `nginx-deploy`, with one single container called `nginx`, image `nginx:1.16` and `4 replicas`. 
- [x] The deployment should use `RollingUpdate` strategy with `maxSurge=1`, and `maxUnavailable=2`.
- [x] Next, upgrade the deployment to `version 1.17`.
- [x] Finally, once all pods are updated, undo the update and go back to the previous version.

```yaml
# Run the following command to create a manifest for deployment nginx-deploy and save it into a file:-
kubectl create deployment nginx-deploy --image=nginx:1.16 --replicas=4 --dry-run=client -oyaml > nginx-deploy.yaml

# and add the strategy field under the spec section as follows:-
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 2

# So final manifest file for deployment called nginx-deploy should looks like below:-
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deploy
  name: nginx-deploy
  namespace: default
spec:
  replicas: 4
  selector:
    matchLabels:
      app: nginx-deploy
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 2
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: nginx-deploy
    spec:
      containers:
      - image: nginx:1.16
        imagePullPolicy: IfNotPresent
        name: nginx

# then run the kubectl apply -f nginx-deploy.yaml to create a deployment resource.
# Now, upgrade the deployment's image version using the kubectl set image command:-
kubectl set image deployment nginx-deploy nginx=nginx:1.17

# Run the kubectl rollout command to undo the update and go back to the previous version:-
kubectl rollout undo deployment nginx-deploy
```
✅ Deployment created correctly? </br>
✅ Was the deployment created with `nginx:1.16`? </br>
✅ Was it upgraded to `1.17`? </br>
✅ Deployment rolled back to `1.16`? </br>

---

### **Question 5: Deployment with Ephemeral and ConfigMap Volumes**
- [x] Create a redis deployment with the following parameters:
- ✅ Name: `redis`
- ✅ Image: `redis:alpine image`
- ✅ Replicas: `1`
- ✅ CPU Request: `0.2` 
- ✅ Label: `app=redis`
- [x] Mount exactly 2 volumes.
- ✅ An Empty directory volume called data at path `/redis-master-data`
- ✅ A configmap volume called redis-config at path `/redis-master`
- [x] Container should expose the port `6379`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: redis
  name: redis
spec:
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      volumes:
      - name: data
        emptyDir: {}
      - name: config
        configMap:
          name: redis-config
      containers:
      - image: redis:alpine
        name: redis
        volumeMounts:
        - mountPath: /redis-master-data
          name: data
        - mountPath: /redis-master
          name: config
        ports:
        - containerPort: 6379
        resources:
          requests:
            cpu: "0.2"
```
---
# <div id='lab2'/> 🧪 **Lightning Lab 2**

### **Question 1: Readiness and Liveness Probes**
- [x] We have deployed a few pods in this cluster in various namespaces. 
- [x] Inspect them and identify the pod which is not in a Ready state. Troubleshoot and fix the issue.
- [x] Next, add a check to restart the container on the same pod if the command `ls /var/www/html/file_check` fails. 
- [x] This check should start after a `delay of 10 seconds` and `run every 60 seconds`.

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
  name: nginx1401
  namespace: dev1401
spec:
  containers:
  - image: kodekloud/nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    ports:
    - containerPort: 9080
      protocol: TCP
    readinessProbe:
      httpGet:
        path: /
        port: 9080 # readinessProbe port must be the same as containerPort!
    livenessProbe:
      exec:
        command:
        - ls
        - /var/www/html/file_check
      initialDelaySeconds: 10
      periodSeconds: 60
```
---
### **Question 2: Cronjobs**
- [x] Create a cronjob called dice that runs every one minute. 
- [x] Use the Pod template located at `/root/throw-a-dice`. 
- [x] The image throw-dice randomly returns a value between 1 and 6. 
- [x] The result of 6 is considered success and all others are failure.
- [x] The job should be `non-parallel` and complete the task `once`. Use a `backoffLimit of 25`.
- [x] If the task is not completed within `20 seconds` the job should fail and pods should be terminated.

```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: dice
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      completions: 1
      backoffLimit: 25 # This is so the job does not quit before it succeeds.
      activeDeadlineSeconds: 20
      template:
        spec:
          containers:
          - name: dice
            image: kodekloud/throw-dice
          restartPolicy: Never
```
---

### **Question 3: Secret Volume and NodeSelector**
- [x] Create a pod called `my-busybox` in the `dev2406` namespace using the `busybox` image. 
- [x] The container should be called `secret` and should `sleep for 3600 seconds`.
- [x] The container should mount a `read-only secret volume` called `secret-volume` at the path `/etc/secret-volume`. 
- [x] The secret being mounted has already been created for you and is called `dotfile-secret`.
- [x] Make sure that the pod is `scheduled on controlplane` and no other node in the cluster.

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: my-busybox
  name: my-busybox
  namespace: dev2406
spec:
  volumes:
  - name: secret-volume
    secret:
      secretName: dotfile-secret
  nodeSelector:
    kubernetes.io/hostname: controlplane
  tolerations:
  - key: "node-role.kubernetes.io/master"
    operator: "Exists"
    effect: "NoSchedule"
  containers:
  - command:
    - sleep
    args:
    - "3600"
    image: busybox
    name: secret
    volumeMounts:
    - name: secret-volume
      readOnly: true
      mountPath: "/etc/secret-volume"
```
---
### **Question 4: Ingress**
- [x] Create a single ingress resource called ingress-vh-routing. 
- [x] The resource should route HTTP traffic to multiple hostnames as specified below:
- ✅ The service video-service should be accessible on http://watch.ecom-store.com:30093/video
- ✅ The service apparels-service should be accessible on http://apparels.ecom-store.com:30093/wear
- ✅ 30093 is the port used by the Ingress Controller

```yaml
kind: Ingress
apiVersion: networking.k8s.io/v1
metadata:
  name: ingress-vh-routing
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: watch.ecom-store.com
    http:
      paths:
      - pathType: Prefix
        path: "/video"
        backend:
          service:
            name: video-service
            port:
              number: 8080
  - host: apparels.ecom-store.com
    http:
      paths:
      - pathType: Prefix
        path: "/wear"
        backend:
          service:
            name: apparels-service
            port:
              number: 8080
```
---
### **Question 5: Logs**
- [x] A pod called dev-pod-dind-878516 has been deployed in the default namespace. 
- [x] Inspect the logs for the container called log-x and redirect the warnings to /opt/dind-878516_logs.txt on the controlplane node
```yaml
kubectl logs dev-pod-dind-878516 -c log-x | grep WARNING > /opt/dind-878516_logs.txt
```
---

# <div id='exam1'/> 📕 **Mock Exam 1**
### **Question 1: Pods**
- [x] Deploy a pod named nginx-448839 using the nginx:alpine image.
```yaml
kubectl run nginx-448839 --image=nginx:alpine
```
✅ Name: `nginx-448839` </br>
✅ Image: `nginx:alpine` </br>

---
### **Question 2: Namespace**
- [x] Create a namespace named apx-z993845
```yaml
kubectl create namespace apx-z993845
```
✅ Namespace: `apx-z993845`</br>

---
### **Question 3: Deployments**
- [x] Create a new Deployment named httpd-frontend with 3 replicas using image httpd:2.4-alpine
```yaml
kubectl create deployment httpd-frontend --image=httpd:2.4-alpine --replicas=3
```
✅ Name: `httpd-frontend` </br>
✅ Replicas: `3` </br>
✅ Image: `httpd:2.4-alpine` </br>

---

### **Question 4: Pods with Labels**
- [x] Deploy a messaging pod using the redis:alpine image with the labels set to tier=msg.
```yaml
kubectl run messaging --image=redis:alpine -l tier=msg
```
✅ Pod Name: `messaging` </br>
✅ Image: `redis:alpine` </br>
✅ Labels: `tier=msg` </br>

---

### **Question 5: Troubleshooting ReplicaSets**
- [x] A replicaset rs-d33393 is created. However the pods are not coming up. Identify and fix the issue.
- [x] Once fixed, ensure the ReplicaSet has 4 Ready replicas.
```yaml
# The image used for the replicaset should be busybox instead of busyboxXXXXXXX. 
# Use kubectl edit rs rs-d33393 to fix the image. 
# Then delete all PODs to provision new ones with the new image.
```
---

### **Question 6: Services**
- [x] Create a service messaging-service to expose the `redis deployment` in the `marketing namespace` within the cluster on `port 6379`.
- [x] Use imperative commands

```yaml
kubectl expose deployment redis --port=6379 --name messaging-service --namespace marketing
```
✅ Service: `messaging-service` </br>
✅ Port: `6379` </br>


---

### **Question 7: Environment Variables**
- [x] Update the environment variable on the pod webapp-color to use a green background.
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2021-07-24T04:54:05Z"
  labels:
    name: webapp-color
  name: webapp-color
  namespace: default
spec:
  containers:
  - env:
    - name: APP_COLOR
      value: green # Set environment variable APP_COLOR to green
    image: kodekloud/webapp-color
    imagePullPolicy: Always
    name: webapp-color

# Recreate the existing pod with the above YAML file. You can make use of the replace command like this:
kubectl replace -f <above yaml file> --force. 

# This will delete the old pod and replace it with the new one with the configuration defined in the YAML file.
```
✅ Pod Name: `webapp-color` </br>
✅ Label Name: `webapp-color` </br>
✅ Env: `APP_COLOR=green` </br>

---

### **Question 8: ConfigMaps**
- [x] Create a new ConfigMap named cm-3392845. Use the spec given on the below.
```yaml
kubectl create configmap cm-3392845 --from-literal=DB_NAME=SQL3322 --from-literal=DB_HOST=sql322.mycompany.com --from-literal=DB_PORT=330
```
✅ ConfigName Name: `cm-3392845` </br>
✅ Data: `DB_NAME=SQL3322` </br>
✅ Data: `DB_HOST=sql322.mycompany.com` </br>
✅ Data: `DB_PORT=3306` </br>

---

### **Question 9: Secrets**
- [x] Create a new Secret named db-secret-xxdf with the data given (on the below).
```yaml
kubectl create secret generic db-secret-xxdf --from-literal=DB_Host=sql01 --from-literal=DB_User=root --from-literal=DB_Password=password123
```
✅ Secret Name: `db-secret-xxdf` </br>
✅ Secret 1: `DB_Host=sql01` </br>
✅ Secret 2: `DB_User=root` </br>
✅ Secret 3: `DB_Password=password123` </br>

---

### **Question 10: System Capabilities**
- [x] Update pod app-sec-kff3345 to run as Root user and with the SYS_TIME capability.
```yaml
# Add SYS_TIME capability to the container's securityContext.
apiVersion: v1
kind: Pod
metadata:
  name: app-sec-kff3345
  namespace: default
spec:
  securityContext:
    runAsUser: 0
  containers:
  - command:
    - sleep
    - "4800"
    image: ubuntu
    name: ubuntu
    securityContext:
     capabilities:
        add: ["SYS_TIME"]
```
✅ Pod Name: `app-sec-kff3345` </br>
✅ Image Name: `ubuntu` </br>
✅ SecurityContext: `Capability SYS_TIME` </br>

---

### **Question 11: Logs**
- [x] Export the logs of the e-com-1123 pod to the file /opt/outputs/e-com-1123.logs
```yaml
kubectl logs e-com-1123 --namespace e-commerce > /opt/outputs/e-com-1123.logs
```
---

### **Question 12: Persistent Volume**
- [x] Create a Persistent Volume with the given specification.
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-analytics
spec:
  capacity:
    storage: 100Mi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteMany
  hostPath:
      path: /pv/data-analytics
```
✅ Volume Name: `pv-analytics` </br>
✅ Storage: `100Mi` </br>
✅ Access modes: `ReadWriteMany` </br>
✅ Host Path: `/pv/data-analytics` </br>

---

### **Question 13: Network Policy**
- [x] Create a redis deployment using the image redis:alpine with 1 replica and label app=redis.
- [x] Expose it via a ClusterIP service called redis on port 6379. 
- [x] Create a new Ingress Type NetworkPolicy called redis-access which allows only the pods with label access=redis to access the deployment.
```yaml
# To create deployment:
kubectl create deployment redis --image=redis:alpine --replicas=1

# To expose the deployment using ClusterIP:
kubectl expose deployment redis --name=redis --port=6379 --target-port=6379

# To create ingress rule:
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: redis-access
  namespace: default
spec:
  podSelector:
    matchLabels:
       app: redis
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          access: redis
    ports:
     - protocol: TCP
       port: 6379
```

---

### **Question 14: Multi-Container Pods**
- [x] Create a Pod called sega with two containers
- ✅ Container 1: Name tails with image busybox and command: sleep 3600.
- ✅ Container 2: Name sonic with image nginx and Environment variable: NGINX_PORT with the value 8080.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sega
spec:
  containers:
  - image: busybox
    name: tails
    command:
    - sleep
    - "3600"
  - image: nginx
    name: sonic
    env:
    - name: NGINX_PORT
      value: "8080"
```

---
# <div id='exam2'/> 📗 **Mock Exam 2**
### **Question 1: Deployments and NodePorts**
- [x] Create a deployment called my-webapp with image: nginx, label tier:frontend and 2 replicas.
- [x] Expose the deployment as a NodePort service with name front-end-service, port: 80 and NodePort: 30083
```yaml
# Create a deployment manifest file called my-webapp using the imperative command
kubectl create deployment my-webapp --image=nginx --replicas=2 --dry-run=client -oyaml > my-webapp.yaml

# now, add given labels which is tier: frontend
vi my-webapp.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    tier: frontend
    app: my-webapp
  name: my-webapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-webapp
  template:
    metadata:
      labels:
        app: my-webapp
    spec:
      containers:
      - image: nginx
        name: nginx

kubectl create -f my-webapp.yaml to create a deployment.

# To expose the deployment as a node port service using the imperative command
kubectl expose deployment my-webapp --name front-end-service --type NodePort --port 80 --dry-run=client -oyaml > front-end-service.yaml

# now, add the nodePort field under the ports section as follows
ports:
 - port: 80
   protocol: TCP
   targetPort: 80
   nodePort: 30083

kubectl apply -f front-end-service.yaml
kubectl get service # check service
```

---

### **Question 2: Taints and Tolerations**
- [x] Add a taint to the node node01 of the cluster. Use the specification below:
- ✅ key: `app_type`, value: `alpha` and effect: `NoSchedule`
- [x] Create a pod called `alpha`, image: `redis` with `toleration to node01`.

```yaml
# To add a taint to the node node01 using the imperative command:-
kubectl taint node node01 app_type=alpha:NoSchedule

# Verify it using the kubectl describe command:-
root@controlplane:~$ kubectl describe node node01 | grep -i app_type

# Now, create a pod manifest file called alpha using the imperative command:-
kubectl run alpha --image=redis --dry-run=client -oyaml > alpha.yaml

# and add rules under the spec.tolerations field as follows:
tolerations:
  - effect: NoSchedule
    key: app_type
    value: alpha

# So final manifest file to create a pod called alpha with tolerations rules as follows:-
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: alpha
  name: alpha
spec:
  tolerations:
  - effect: NoSchedule
    key: app_type
    value: alpha
  containers:
  - image: redis
    name: alpha
  dnsPolicy: ClusterFirst
  restartPolicy: Always

# To check the pod has been successfully deployed on the node node01, using -o wide flag:-
kubectl get pods -o wide
```
---

### **Question 3: Node Affinity**
- [x] Apply a label `app_type=beta` to node `controlplane`. 
- [x] Create a new deployment called `beta-apps` with image: `nginx` and replicas: `3`. 
- [x] Set Node Affinity to the deployment to place the PODs on controlplane only.
- ✅ NodeAffinity: `requiredDuringSchedulingIgnoredDuringExecution`
```yaml
# Add a label to node controlplane:-
kubectl label node controlplane app_type=beta

# then create a deployment called beta-apps as follows:-
kubectl create deploy beta-apps --image=nginx --replicas=3 --dry-run=client -oyaml > beta.yaml

# Add affinity field under the spec.template.spec section, as follows:-
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: beta-apps
  name: beta-apps
spec:
  replicas: 3
  selector:
    matchLabels:
      app: beta-apps
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: beta-apps
    spec:
      affinity:
        nodeAffinity:
         requiredDuringSchedulingIgnoredDuringExecution:
           nodeSelectorTerms:
           - matchExpressions:
             - key: app_type
               values: ["beta"]
               operator: In
      containers:
      - image: nginx
        name: nginx
kubectl create -f beta.yaml
```
---

### **Question 4: Ingress**
- [x] Create a new Ingress Resource for the service: `my-video-service` to be made available at the URL: `http://ckad-mock-exam-solution.com:30093/video`.
- [x] Create an ingress resource with host: `ckad-mock-exam-solution.com`
- [x] Service prefix path: `/video`
- [x] Once set up, curl test of the URL from the nodes should be successful / HTTP 200
```yaml
# Create an ingress resource manifest file using the imperative command:-
kubectl create ingress ingress --rule="ckad-mock-exam-solution.com/video*=my-video-service:8080" --dry-run=client -oyaml > ingress.yaml
kubectl create -f ingress.yaml
```
✅ http://ckad-mock-exam-solution.com:30093/video accessible? </br>

---

### **Question 5: Readiness Probes**
- [x] We have deployed a new pod called pod-with-rprobe. This Pod has an initial delay before it is Ready.
- [x] Update the newly created pod pod-with-rprobe with a readinessProbe using the given spec
- ✅ httpGet path: `/ready`
- ✅ httpGet port: `8080`
```yaml
# Get the manifest of already running pod pod-with-rprobe using the imperative command:-
kubectl get po pod-with-rprobe -oyaml > pod.yaml

# With help of vi editor, Add the readinessProbe under the spec.containers section:-
readinessProbe:
  httpGet:
    path: /ready
    port: 8080

# then run the kubectl replace command, it will delete the existing pod and recreate a new pod with changes:-
kubectl replace -f pod.yaml --force
```

---

### **Question 6: Liveness Probes**
- [x] Create a new pod called nginx1401 in the default namespace with the image nginx. 
- [x] Add a livenessProbe to the container to restart it if the command ls /var/www/html/probe fails. 
- [x] This check should start after a delay of 10 seconds and run every 60 seconds.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx1401
  namespace: default
spec:
  containers:
    - name: nginx1401
      image: nginx
      livenessProbe:
        exec:
          command: ["ls /var/www/html/probe"]
        initialDelaySeconds: 10
        periodSeconds: 60
```

---

### **Question 7: Jobs**
- [x] Create a job called whalesay with image docker/whalesay and command "cowsay I am going to ace CKAD!".

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: whalesay
spec:
  completions: 10
  backoffLimit: 6
  template:
    metadata:
      creationTimestamp: null
    spec:
      containers:
      - command:
        - sh 
        - -c
        - "cowsay I am going to ace CKAD!"
        image: docker/whalesay
        name: whalesay
      restartPolicy`: Never
```
✅ completions: `10` </br>
✅ backoffLimit: `6` </br>
✅ restartPolicy: `Never` </br>

---
### **Question 8: Multi-Container Pods**
- [x] Create a pod called multi-pod with two containers with the below specs:
```yaml
# Name and Images
Container 1:
name: jupiter, image: nginx
Container 2:
name: europa, image: busybox
command: sleep 4800

# Environment Variables
Container 1:
type: planet
Container 2:
type: moon
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: multi-pod
  name: multi-pod
spec:
  containers:
  - image: nginx
    name: jupiter
    env:
    - name: type
      value: planet
  - image: busybox
    name: europa
    command: ["/bin/sh","-c","sleep 4800"]
    env:
     - name: type
       value: moon
```

---
### **Question 9: Persistent Volumes**
- [x] Create a PersistentVolume called custom-volume with size: `50MiB` reclaim policy: `retain`, Access Modes: `ReadWriteMany` and hostPath: `/opt/data`
  
```yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: custom-volume
spec:
  accessModes: ["ReadWriteMany"]
  capacity:
    storage: 50Mi
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /opt/data
```

---
### [📋 **Back to Table of Contents**](#toc)
---