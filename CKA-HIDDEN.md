<style>
details > summary {
  padding: 4px;
  width: 100px;
  background-color: #3c3d3c;
  border: none;
  box-shadow: 1px 1px 2px #101010;
  cursor: pointer;
}
</style>

<img src="https://kubernetes.io/images/kubernetes-horizontal-color.png" width="500px"/><br/>

# **📝 Certified Kubernetes Administrator (CKA) Practice Exams**
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
0. ### [🧬 **Lightning Lab**](#lightning)
0. ### [📕 **Mock Exam 1**](#exam1)
0. ### [📘 **Mock Exam 2**](#exam2)
0. ### [📗 **Mock Exam 3**](#exam3)

---
# <div id='syllabus'/> **📓 Certified Kubernetes Administrator (CKA) Exam Syllabus**
## **25% - ⎈ Cluster Architecture, Installation & Configuration**
- Manage role based access control (RBAC)
- Use Kubeadm to install a basic cluster
- Manage a highly-available Kubernetes cluster
- Provision underlying infrastructure to deploy a Kubernetes cluster
- Perform a version upgrade on a Kubernetes cluster using Kubeadm
- Implement etcd backup and restore

## **15% - ⚡️ Workloads & Scheduling**
- Understand deployments and how to perform rolling update and rollbacks
- Use ConfigMaps and Secrets to configure applications
- Know how to scale applications
- Understand the primitives used to create robust, self-healing, application deployments
- Understand how resource limits can affect Pod scheduling
- Awareness of manifest management and common templating tools

## **20% - 🌍 Services & Networking**
- Understand host networking configuration on the cluster nodes
- Understand ClusterIP, NodePort, LoadBalancer service types and endpoints
- Know how to use Ingress controllers and Ingress resources
- Know how to configure and use CoreDNS
- Choose an appropriate container network interface plugin

## **10% - 📀 Storage**
- Understand storage classes, persistent volumes
- Understand volume mode, access modes and reclaim policies for volumes
- Understand persistent volume claims primitive
- Know how to configure applications with persistent storage

## **30% - 👩‍🔧 Troubleshooting**
- Evaluate cluster and node logging
- Understand how to monitor applications
- Manage container stdout & stderr logs
- Troubleshoot application failure
- Troubleshoot cluster component failure
- Troubleshoot networking
---
# <div id='lightning'/> 🧬 **Lightning Lab**
### **Question 1: Kubeadm Cluster Upgrade**
- [x] Upgrade the current version of kubernetes from 1.19 to 1.20.0 exactly using the kubeadm utility. 
- [x] Make sure that the upgrade is carried out one node at a time starting with the master node. 
- [x] To minimize downtime, the deployment gold-nginx should be rescheduled on an alternate node before upgrading each node.
- [x] Upgrade controlplane node first and drain node node01 before upgrading it. Pods for gold-nginx should run on the controlplane node subsequently.
<details><summary><b>Solution</b></summary>
<p>

```yaml
# On the Control Plane: 
root@controlplane:~# kubectl drain controlplane --ignore-daemonsets
root@controlplane:~# apt update
root@controlplane:~# apt-get install kubeadm=1.20.0-00
root@controlplane:~# kubeadm upgrade plan v1.20.0
root@controlplane:~# kubeadm upgrade apply v1.20.0
root@controlplane:~# apt-get install kubelet=1.20.0-00
root@controlplane:~# systemctl daemon-reload
root@controlplane:~# systemctl restart kubelet
root@controlplane:~# kubectl uncordon controlplane 
root@controlplane:~# kubectl drain node01 --ignore-daemonsets

# On the Worker Node (node01): 
root@node01:~# apt update
root@node01:~# apt-get install kubeadm=1.20.0-00
root@node01:~# kubeadm upgrade node
root@node01:~# apt-get install kubelet=1.20.0-00
root@node01:~# systemctl daemon-reload
root@node01:~# systemctl restart kubelet

# Back to the Control Plane:
root@controlplane:~# kubectl uncordon node01
root@controlplane:~# kubectl get pods -o wide | grep gold (make sure this is scheduled on node)
```
</p>
</details>

---
### **Question 2: Deployments**
- [x] Print the names of all deployments in the admin2406 namespace in the following format:
  ```
  DEPLOYMENT        CONTAINER_IMAGE        READY_REPLICAS        NAMESPACE
  <deployment name> <container image used> <ready replica count> <Namespace>
  ```
- [x] The data should be sorted by the increasing order of the deployment name.
  ```
  DEPLOYMENT CONTAINER_IMAGE READY_REPLICAS NAMESPACE
  deploy0    nginx:alpine    1              admin2406
  ```
- [x] Write the result to the file /opt/admin2406_data.

<details><summary><b>Solution</b></summary>
<p>

```yaml
$ kubectl -n admin2406 get deployment -o custom-columns=DEPLOYMENT:.metadata.name,CONTAINER_IMAGE:.spec.template.spec.containers[].image,READY_REPLICAS:.status.readyReplicas,NAMESPACE:.metadata.namespace --sort-by=.metadata.name > /opt/admin2406_data
```
</p>
</details>

---
### **Question 3: Troubleshooting KubeConfig**
- [x] A kubeconfig file called admin.kubeconfig has been created in /root/CKA. 
- [x] There is something wrong with the configuration. Troubleshoot and fix it.

<details><summary><b>Solution</b></summary>
<p>

```yaml 
# Make sure the port for the kube-apiserver is correct. 
Run the below command to know the cluster information:
$ kubectl cluster-info --kubeconfig /root/CKA/admin.kubeconfig
$ vi /root/CKA/admin.kubeconfig # Change port from 4380 to 6443.
```
</p>
</details>

---
### **Question 4: Deployments and Rollouts**
- [x] Create a new deployment called nginx-deploy, with image nginx:1.16 and 1 replica. 
- [x] Next, upgrade the deployment to version 1.17 using rolling update. 
- [x] Make sure that the version upgrade is recorded in the resource annotation.

<details><summary><b>Solution</b></summary>
<p>

```yaml
# Make use of the kubectl create command to create the deployment and explore the --record option while upgrading the deployment image.
Run the below command to create a deployment nginx-deploy:
$ kubectl create deployment  nginx-deploy --image=nginx:1.16
Run the below command to update the new image for nginx-deploy deployment and to record the version:
$ kubectl set image deployment/nginx-deploy nginx=nginx:1.17 --record
```
</p>
</details>

---

### **Question 5: Persistent Volume Claims**
- [x] A new deployment called alpha-mysql has been deployed in the alpha namespace. 
- [x] However, the pods are not running. Troubleshoot and fix the issue. 
- [x] The deployment should make use of the persistent volume alpha-pv to be mounted at /var/lib/mysql and should use the environment variable MYSQL_ALLOW_EMPTY_PASSWORD=1 to make use of an empty root password.
- [x] Important: Do not alter the persistent volume.

<details><summary><b>Solution</b></summary>
<p>

```yaml
Use the command kubectl describe and try to fix the issue.
Solution manifest file to create a pvc called mysql-alpha-pvc as follows:

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-alpha-pvc
  namespace: alpha
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: slow
```
</p>
</details>

--- 

### **Question 6: ETCD Backup**
- [x] Take the backup of ETCD at the location /opt/etcd-backup.db on the controlplane node.

<details><summary><b>Solution</b></summary>
<p>

```yaml
# Make use of help from the etcdctl snapshot save commend
etcdctl snapshot save --help

# Get PKI file path information using the following command
kubectl describe pod etcd-controlplane -n kube-system

# Replace PKI file paths in the below command and execute it to save an ETCD snapshot in volume
export ETCDCTL_API=3
etcdctl snapshot save --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key --endpoints=127.0.0.1:2379 /opt/etcd-backup.db
```
</p>
</details>

---

### **Question 7: Pods and Persistent Volumes**
- [x] Create a pod called secret-1401 in the admin1401 namespace using the busybox image.
- [x] The container within the pod should be called secret-admin and should sleep for 4800 seconds.
- [x] The container should mount a read-only secret volume called secret-volume at the path /etc/secret-volume. 
- [x] The secret being mounted has already been created for you and is called dotfile-secret.

<details><summary><b>Solution</b></summary>
<p>

```yaml
Use the command kubectl run to create a pod definition file. Add secret volume and update container name in it.
Alternatively, run the command:
kubectl run secret-1401 --image=busybox --dry-run=client -oyaml --command -- sleep 4800 > admin.yaml

Add the secret volume and mount path to create a pod called secret-1401 in the admin1401 namespace as follows:
---
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: secret-1401
  name: secret-1401
  namespace: admin1401
spec:
  volumes:
  - name: secret-volume
    # secret volume
    secret:
      secretName: dotfile-secret
  containers:
  - command:
    - sleep
    args:
    - "4800"
    image: busybox
    name: secret-admin
    # volumes' mount path
    volumeMounts:
    - name: secret-volume
      readOnly: true
      mountPath: "/etc/secret-volume"
```
</p>
</details>

---
# <div id='exam1'/> 📕 **Mock Exam 1**
### **Question 1: Pod**
- [x] Deploy a pod named nginx-pod using the nginx:alpine image.

<details><summary><b>Solution</b></summary>
<p>

```yaml
kubectl run nginx-pod --image=nginx:alpine
```
</p>
</details>

---
### **Question 2: Pod with Labels**
- [x] Deploy a messaging pod using the redis:alpine image with the labels set to tier=msg.

<details><summary><b>Solution</b></summary>
<p>

```yaml
kubectl run messaging --image=redis:alpine -l tier=msg
```
</p>
</details>

---
### **Question 3: Namespaces**
- [x] Create a namespace named apx-x9984574.

<details><summary><b>Solution</b></summary>
<p>

```yaml
kubectl create namespace apx-x9984574
```
</p>
</details>

---
### **Question 4: JSON Output**
- [x] Get the list of nodes in JSON format and store it in a file at /opt/outputs/nodes-z3444kd9.json.

<details><summary><b>Solution</b></summary>
<p>

```yaml
kubectl get nodes -o json > /opt/outputs/nodes-z3444kd9.json
```
</p>
</details>

---
### **Question 5: Services**
- [x] Create a service messaging-service to expose the messaging application within the cluster on port 6379.
- [x] Use imperative commands.

<details><summary><b>Solution</b></summary>
<p>

```yaml
kubectl expose pod messaging --port=6379 --name messaging-service
```
</p>
</details>

---
### **Question 6: Deployments**
- [x] Create a deployment named hr-web-app using the image kodekloud/webapp-color with 2 replicas.

<details><summary><b>Solution</b></summary>
<p>

```yaml
kubectl create deployment hr-web-app --image=kodekloud/webapp-color --replicas=2
```
</p>
</details>

---
### **Question 7: Static Pods**
- [x] Create a static pod named static-busybox on the controlplane node that uses the busybox image and the command sleep 1000.

<details><summary><b>Solution</b></summary>
<p>

```yaml
Create a pod definition file in the manifests directory. For that use command kubectl run --restart=Never --image=busybox static-busybox --dry-run=client -oyaml --command -- sleep 1000 > /etc/kubernetes/manifests/static-busybox.yaml
```
</p>
</details>

---
### **Question 8: Pods**
- [x] Create a pod in the finance namespace named temp-bus with the image redis:alpine.

<details><summary><b>Solution</b></summary>
<p>

```yaml
kubectl run temp-bus --image=redis:alpine --namespace=finance --restart=Never
```
</p>
</details>

---
### **Question 9: NodePort**
- [x] Expose the hr-web-app as service hr-web-app-service application on port 30082 on the nodes on the cluster.

<details><summary><b>Solution</b></summary>
<p>

```yaml
Run the command: 
kubectl expose deployment hr-web-app --type=NodePort --port=8080 --name=hr-web-app-service --dry-run=client -o yaml > hr-web-app-service.yaml to generate a service definition file.
# Now, in generated service definition file add the nodePort field with the given port number under the ports section and create a service.
```
</p>
</details>

---
### **Question 10: JSONPath**
- [x] Use JSON PATH query to retrieve the osImages of all the nodes and store it in a file /opt/outputs/nodes_os_x43kj56.txt.
- [x] The osImages are under the nodeInfo section under status of each node.

<details><summary><b>Solution</b></summary>
<p>

```yaml
kubectl get nodes -o jsonpath='{.items[*].status.nodeInfo.osImage}' > /opt/outputs/nodes_os_x43kj56.txt
```
</p>
</details>

---
### **Question 11: Persistent Volumes**
- [x] Create a Persistent Volume with the given specification.

<details><summary><b>Solution</b></summary>
<p>

```yaml
---
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
</p>
</details>

---

# <div id='exam2'/> 📘 **Mock Exam 2**
### **Question 1: ETCD Backup**
- [x] Take a backup of the etcd cluster and save it to /opt/etcd-backup.db

<details><summary><b>Solution</b></summary>
<p>

```yaml
Run the following command to take a backup:
export ETCDCTL_API=3
etcdctl snapshot save --endpoints https://[127.0.0.1]:2379 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key  /opt/etcd-backup.db
```
</p>
</details>

---
### **Question 2: Pods with Ephemeral Volume**
- [x] Create a Pod called redis-storage with image: redis:alpine with a Volume of type emptyDir that lasts for the life of the Pod.

<details><summary><b>Solution</b></summary>
<p>

```yaml
# Use the command kubectl run and create a pod definition file for redis-storage pod and add volume.
Alternatively run the command:
kubectl run redis-storage --image=redis:alpine --dry-run=client -oyaml > redis-storage.yaml
and add volume emptyDir in it.

Solution manifest file to create a pod redis-storage as follows:
---
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: redis-storage
  name: redis-storage
spec:
  containers:
  - image: redis:alpine
    name: redis-storage
    volumeMounts:
    - mountPath: /data/redis
      name: temp-volume
  volumes:
  - name: temp-volume
    emptyDir: {}
```
</p>
</details> 

✅ Pod named 'redis-storage' created </br>
✅ Pod 'redis-storage' uses Volume type of emptyDir </br>
✅ Pod 'redis-storage' uses volumeMount with mountPath = /data/redis </br>

---
### **Question 3: Pods with System Capabilities**
- [x] Create a new pod called super-user-pod with image busybox:1.28. Allow the pod to be able to set system_time.

<details><summary><b>Solution</b></summary>
<p>

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: super-user-pod
  name: super-user-pod
spec:
  containers:
  - command:
    - sleep
    - "4800"
    image: busybox:1.28
    name: super-user-pod
    securityContext:
      capabilities:
        add: ["SYS_TIME"]
  dnsPolicy: ClusterFirst
  restartPolicy: Always
```
</p>
</details>

✅ `Pod:` super-user-pod </br>
✅ `Container Image:` busybox:1.28 </br>
✅ SYS_TIME capabilities for the conatiner? </br>

---
### **Question 4: Pods with PV and PVCs**
- [x] A pod definition file is created at /root/CKA/use-pv.yaml. 
- [x] Make use of this manifest file and mount the persistent volume called pv-1. 
- [x] Ensure the pod is running and the PV is bound.
  - [x] mountPath: /data
  - [x] persistentVolumeClaim Name: my-pvc

<details><summary><b>Solution</b></summary>
<p>

```yaml
# #Add a persistentVolumeClaim definition to pod definition file:
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
       storage: 10Mi

# And then update the pod definition file as follows:
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: use-pv
  name: use-pv
spec:
  containers:
  - image: nginx
    name: use-pv
    volumeMounts:
    - mountPath: "/data"
      name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: my-pvc

# Finally, create the pod by running: kubectl create -f /root/CKA/use-pv.yaml
```
</p>
</details>

---
### **Question 5: Deployments and Rollouts**
- [x] Create a new deployment called nginx-deploy, with image nginx:1.16 and 1 replica. Record the version. 
- [x] Next, upgrade the deployment to version 1.17 using rolling update. 
- [x] Make sure that the version upgrade is recorded in the resource annotation.

<details><summary><b>Solution</b></summary>
<p>


```yaml
# Explore the --record option while creating the deployment while working with the deployment definition file. Then make use of the kubectl apply command to create or update the deployment.

To create a deployment definition file nginx-deploy:
$ kubectl create deployment nginx-deploy --image=nginx:1.16 --dry-run=client -o yaml > deploy.yaml

To create a resource from definition file and to record:
$ kubectl apply -f deploy.yaml --record

To view the history of deployment nginx-deploy:
$ kubectl rollout history deployment nginx-deploy

To upgrade the image to next given version:
$ kubectl set image deployment/nginx-deploy nginx=nginx:1.17 --record

To view the history of deployment nginx-deploy:
$ kubectl rollout history deployment nginx-deploy
```
</p>
</details>

✅ `Deployment:` nginx-deploy. Image: nginx:1.16 </br>
✅ `Image:` nginx:1.16 </br>
✅ `Task:` Upgrade the version of the deployment to 1:17 </br>
✅ `Task:` Record the changes for the image upgrade

---

### **Question 6: CSR and RBAC**
- [x] Create a new user called john. Grant him access to the cluster. 
- [x] John should have permission to create, list, get, update and delete pods in the development namespace. 
- [x] The private key exists in the location: /root/CKA/john.key and csr at /root/CKA/john.csr.
- [x] Important Note: As of kubernetes 1.19, the CertificateSigningRequest object expects a signerName.

<details><summary><b>Solution</b></summary>
<p>

```yaml
# Solution manifest file to create a CSR as follows:
---
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: john-developer
spec:
  signerName: kubernetes.io/kube-apiserver-client
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZEQ0NBVHdDQVFBd0R6RU5NQXNHQTFVRUF3d0VhbTlvYmpDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRApnZ0VQQURDQ0FRb0NnZ0VCQUt2Um1tQ0h2ZjBrTHNldlF3aWVKSzcrVVdRck04ZGtkdzkyYUJTdG1uUVNhMGFPCjV3c3cwbVZyNkNjcEJFRmVreHk5NUVydkgyTHhqQTNiSHVsTVVub2ZkUU9rbjYra1NNY2o3TzdWYlBld2k2OEIKa3JoM2prRFNuZGFvV1NPWXBKOFg1WUZ5c2ZvNUpxby82YU92czFGcEc3bm5SMG1JYWpySTlNVVFEdTVncGw4bgpjakY0TG4vQ3NEb3o3QXNadEgwcVpwc0dXYVpURTBKOWNrQmswZWhiV2tMeDJUK3pEYzlmaDVIMjZsSE4zbHM4CktiSlRuSnY3WDFsNndCeTN5WUFUSXRNclpUR28wZ2c1QS9uREZ4SXdHcXNlMTdLZDRaa1k3RDJIZ3R4UytkMEMKMTNBeHNVdzQyWVZ6ZzhkYXJzVGRMZzcxQ2NaanRxdS9YSmlyQmxVQ0F3RUFBYUFBTUEwR0NTcUdTSWIzRFFFQgpDd1VBQTRJQkFRQ1VKTnNMelBKczB2czlGTTVpUzJ0akMyaVYvdXptcmwxTGNUTStsbXpSODNsS09uL0NoMTZlClNLNHplRlFtbGF0c0hCOGZBU2ZhQnRaOUJ2UnVlMUZnbHk1b2VuTk5LaW9FMnc3TUx1a0oyODBWRWFxUjN2SSsKNzRiNnduNkhYclJsYVhaM25VMTFQVTlsT3RBSGxQeDNYVWpCVk5QaGhlUlBmR3p3TTRselZuQW5mNm96bEtxSgpvT3RORStlZ2FYWDdvc3BvZmdWZWVqc25Yd0RjZ05pSFFTbDgzSkljUCtjOVBHMDJtNyt0NmpJU3VoRllTVjZtCmlqblNucHBKZWhFUGxPMkFNcmJzU0VpaFB1N294Wm9iZDFtdWF4bWtVa0NoSzZLeGV0RjVEdWhRMi80NEMvSDIKOWk1bnpMMlRST3RndGRJZjAveUF5N05COHlOY3FPR0QKLS0tLS1FTkQgQ0VSVElGSUNBVEUgUkVRVUVTVC0tLS0tCg==
  usages:
  - digital signature
  - key encipherment
  - client auth
  groups:
  - system:authenticated

# To approve this certificate, run: 
kubectl certificate approve john-developer

# Next, create a role developer and rolebinding developer-role-binding, run the command:
kubectl create role developer --resource=pods --verb=create,list,get,update,delete --namespace=development
kubectl create rolebinding developer-role-binding --role=developer --user=john --namespace=development

# To verify the permission from kubectl utility tool:
kubectl auth can-i update pods --as=john --namespace=development
```
</p>
</details>

✅ CSR: john-developer Status:Approved </br>
✅ Role Name: developer, namespace: development, Resource: Pods </br>
✅ Access: User 'john' has appropriate permissions </br>

---

### **Question 7: Services and DNS**
- [x] Create a nginx pod called nginx-resolver using image nginx, expose it internally with a service called nginx-resolver-service. 
- [x] Test that you are able to look up the service and pod names from within the cluster. 
- [x] Use the image: busybox:1.28 for dns lookup. 
- [x] Record results in /root/CKA/nginx.svc and /root/CKA/nginx.pod

<details><summary><b>Solution</b></summary>
<p>

```yaml
# Use the command kubectl run and create a nginx pod and busybox pod. Resolve it, nginx service and its pod name from busybox pod.
To create a pod nginx-resolver and expose it internally:
kubectl run nginx-resolver --image=nginx
kubectl expose pod nginx-resolver --name=nginx-resolver-service --port=80 --target-port=80 --type=ClusterIP

To create a pod test-nslookup. Test that you are able to look up the service and pod names from within the cluster:
kubectl run test-nslookup --image=busybox:1.28 --rm -it --restart=Never -- nslookup nginx-resolver-service
kubectl run test-nslookup --image=busybox:1.28 --rm -it --restart=Never -- nslookup nginx-resolver-service > /root/CKA/nginx.svc

Get the IP of the nginx-resolver pod and replace the dots with hyphens which will be used below:
kubectl get pod nginx-resolver -o wide
kubectl run test-nslookup --image=busybox:1.28 --rm -it --restart=Never -- nslookup <P-O-D-I-P.default.pod> > /root/CKA/nginx.pod
```
</p>
</details>

✅ Pod: nginx-resolver created </br>
✅ Service DNS Resolution recorded correctly </br>
✅ Pod DNS resolution recorded correctly </br>

---

### **Question 8: Static Pods in Worker Node**
- [x] Create a static pod on node01 called nginx-critical with image nginx and make sure that it is recreated/restarted automatically in case of a failure.
- [x] Use /etc/kubernetes/manifests as the Static Pod path for example.

<details><summary><b>Solution</b></summary>
<p>

```yaml
Create a static pod called nginx-critical by using below command:
kubectl run nginx-critical --image=nginx --dry-run=client -o yaml > static.yaml

Copy the contents of this file or use scp command to transfer this file from controlplane to node01 node:
root@controlplane:~# scp static.yaml node01:/root/

To know the IP Address of the node01 node:
root@controlplane:~# kubectl get nodes -o wide

Perform SSH to node01:
root@controlplane:~# ssh node01
OR
root@controlplane:~# ssh <IP of node01>

# On node01, check if static pod directory is present which is /etc/kubernetes/manifests, if it's not present then create it
root@node01:~# mkdir -p /etc/kubernetes/manifests

Add that complete path to the staticPodPath field in the kubelet config.yaml file:
root@node01:~# vi /var/lib/kubelet/config.yaml

Move/copy the static.yaml to path /etc/kubernetes/manifests/:
root@node01:~# cp /root/static.yaml /etc/kubernetes/manifests/

Go back to the controlplane node and check the status of static pod:
root@node01:~# exit
logout
root@controlplane:~# kubectl get pods 
```
</p>
</details>

---
# <div id='exam3'/> 📗 **Mock Exam 3**
### **Question 1: Service Accounts, Cluster Roles and Cluster Role Bindings**
- [x] Create a new service account with the name pvviewer. 
- [x] Grant this Service account access to list all PersistentVolumes in the cluster by creating an appropriate cluster role called pvviewer-role and ClusterRoleBinding called pvviewer-role-binding.
- [x] Next, create a pod called pvviewer with the image: redis and serviceAccount: pvviewer in the default namespace.

<details><summary><b>Solution</b></summary>
<p>

```yaml
# Pods authenticate to the API Server using ServiceAccounts. If the serviceAccount name is not specified, the default service account for the namespace is used during a pod creation.
Reference: https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/

Create a service account pvviewer:
kubectl create serviceaccount pvviewer

To create a clusterrole:
kubectl create clusterrole pvviewer-role --resource=persistentvolumes --verb=list

To create a clusterrolebinding:
kubectl create clusterrolebinding pvviewer-role-binding --clusterrole=pvviewer-role --serviceaccount=default:pvviewer

Solution manifest file to create a new pod called pvviewer as follows:
---
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: pvviewer
  name: pvviewer
spec:
  containers:
  - image: redis
    name: pvviewer
  # Add service account name
  serviceAccountName: pvviewer
```
</p>
</details>

✅ `ServiceAccount:` pvviewer </br>
✅ `ClusterRole:` pvviewer-role </br>
✅ `ClusterRoleBinding:` pvviewer-role-binding </br>
✅ `Pod:` pvviewer </br>
✅ Pod configured to use ServiceAccount pvviewer ?</br>

---
### **Question 2: JSONPath**
- [x] List the InternalIP of all nodes of the cluster. 
- [x] Save the result to a file /root/CKA/node_ips.
- [x] Answer should be in the format: InternalIP of controlplane `space` InternalIP of node01 (in a single line)

<details><summary><b>Solution</b></summary>
<p>

```yaml
# Explore the jsonpath loop.
kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address}' > /root/CKA/node_ips
```
</p>
</details>

---
### **Question 3: Multi-Container Pods**
- [x] Create a pod called multi-pod with two containers.
- [x] Container 1 - name: alpha, image: nginx
- [x] Container 2 - name: beta, image: busybox, command: sleep 4800
- [x] Create the below environment variables for each container.
    ```yaml
    # Environment Variables
    container 1:
    name: alpha
    Container 2:
    name: beta
    ```
    
<details><summary><b>Solution</b></summary>
<p>

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: multi-pod
spec:
  containers:
  - image: nginx
    name: alpha
    env:
    - name: name
      value: alpha
  - image: busybox
    name: beta
    command: ["sleep", "4800"]
    env:
    - name: name
      value: beta
```
</p>
</details>

✅ `Pod Name:` multi-pod</br>
✅ `Container 1:` alpha</br>
✅ `Container 2:` beta</br>
✅ Container beta commands set correctly?</br>
✅ Container 1 Environment Value Set</br>
✅ Container 2 Environment Value Set</br>

---
### **Question 4: Pods with Security Contexts**
- [x] Create a pod with the following specifications:
    ```yaml
    name: non-root-pod
    image: redis:alpine
    runAsUser: 1000
    fsGroup: 2000
    ```

<details><summary><b>Solution</b></summary>
<p>

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: non-root-pod
spec:
  securityContext:
    runAsUser: 1000
    fsGroup: 2000
  containers:
  - name: non-root-pod
    image: redis:alpine

# Verify the user and group IDs by using below command:
kubectl exec -it non-root-pod -- id
```
</p>
</details>

✅ Pod non-root-pod fsGroup configured</br>
✅ Pod non-root-pod runAsUser configured</br>

---
### **Question 5: Network Policies**
- [x] We have deployed a new pod called np-test-1 and a service called np-test-service. 
- [x] Incoming connections to this service are not working. Troubleshoot and fix it.
- [x] Create NetworkPolicy, by the name ingress-to-nptest that allows incoming connections to the service over port 80.
- [x] Important: Don't delete any current objects deployed.

<details><summary><b>Solution</b></summary>
<p>

```yaml
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: ingress-to-nptest
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: np-test-1
  policyTypes:
  - Ingress
  ingress:
  - ports:
    - protocol: TCP
      port: 80
```
</p>
</details>

✅ `NetworkPolicy:` Applied to All sources (Incoming traffic from all pods)?</br>
✅ `NetWorkPolicy:` Correct Port?</br>
✅ `NetWorkPolicy:` Applied to correct Pod?</br>

---
### **Question 6: Taints and Tolerations**
- [x] Taint the worker node node01 to be Unschedulable. 
- [x] Once done, create a pod called dev-redis, image redis:alpine, to ensure workloads are not scheduled to this worker node. 
- [x] Finally, create a new pod called prod-redis and image: redis:alpine with toleration to be scheduled on node01.
- [x] Use the below specifications for Node Taints and Pod Tolerations:
    ```yaml
    key: env_type
    value: production
    operator: Equal
    effect: NoSchedule
    ```

<details><summary><b>Solution</b></summary>
<p>

```yaml
To add taints on the node01 worker node:
kubectl taint node node01 env_type=production:NoSchedule

Deploy dev-redis pod and to ensure that workloads are not scheduled to this node01 worker node:
kubectl run dev-redis --image=redis:alpine

To view the node name of recently deployed pod:
kubectl get pods -o wide

Solution manifest file to deploy new pod called prod-redis with toleration to be scheduled on node01 worker node:
---
apiVersion: v1
kind: Pod
metadata:
  name: prod-redis
spec:
  containers:
  - name: prod-redis
    image: redis:alpine
  tolerations:
  - effect: NoSchedule
    key: env_type
    operator: Equal
    value: production     
  # Alternatively, you can apply toleration of the default Node Taint
  - effect: NoSchedule
    key: node.kubernetes.io/unschedulable:NoSchedule
    operator: Exists

To view only prod-redis pod with less details:
kubectl get pods -o wide | grep prod-redis
```
</p>
</details>

✅ `Key:` env_type</br>
✅ `Value:` production</br>
✅ `Effect:` NoSchedule</br>
✅ pod 'dev-redis' (no tolerations) is not scheduled on node01?</br>
✅ Create a pod 'prod-redis' to run on node01</br>

---
### **Question 7: Pods with Labels**
- [x] Create a pod called hr-pod in hr namespace belonging to the production environment and frontend tier.
- [x] Use appropriate labels and create all the required objects if it does not exist in the system already.
    ```yaml
    name: hr-pod
    namespace: hr
    image: redis:alpine
    labels: environment:production, tier:frontend
    ```

<details><summary><b>Solution</b></summary>
<p>

```yaml
Create a namespace if it doesn't exist:
kubectl create namespace hr

Create a hr-pod with given details:
kubectl run hr-pod --image=redis:alpine --namespace=hr --labels=environment=production,tier=frontend
```
</p>
</details>

✅ hr-pod labeled with environment production? </br>
✅ hr-pod labeled with tier frontend? </br>

---
### **Question 8: Troubleshooting KubeConfig**
- [x] A kubeconfig file called super.kubeconfig has been created under /root/CKA. There is something wrong with the configuration. Troubleshoot and fix it.

<details><summary><b>Solution</b></summary>
<p>

```yaml
# Verify host and port for kube-apiserver are correct.
# Open the super.kubeconfig in vi editor
Change the 9999 port to 6443 and run the below command to verify:
kubectl cluster-info --kubeconfig=/root/CKA/super.kubeconfig
```
</p>
</details>

✅ Fix /root/CKA/super.kubeconfig</br>

---
### **Question 9: Troubleshooting Deployments**
- [x] We have created a new deployment called nginx-deploy. scale the deployment to 3 replicas. 
- [x] Has the replica's increased? Troubleshoot the issue and fix it.

<details><summary><b>Solution</b></summary>
<p>

```yaml
Use the command kubectl scale to increase the replica count to 3:
kubectl scale deploy nginx-deploy --replicas=3

# The controller-manager is responsible for scaling up pods of a replicaset. If you inspect the control plane components in the kube-system namespace, you will see that the controller-manager is not running.
kubectl get pods -n kube-system

# The command running inside the controller-manager pod is incorrect.
# After fix all the values in the file and wait for controller-manager pod to restart.
Alternatively you can run sed command to change all values at once:
sed -i 's/kube-contro1ler-manager/kube-controller-manager/g' /etc/kubernetes/manifests/kube-controller-manager.yaml
# This will fix the issues in controller-manager yaml file.

At last, inspect the deployment by using below command:
kubectl get deploy
```
</p>
</details>

✅ Deployment has 3 replicas</br>

---
### [📋 **Back to Table of Contents**](#toc)
---