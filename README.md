# Kubernetes Guidelines

## Creating Kubernetes Cluster
Below link can be used as a reference to setup a kubernetes cluster. 

https://phoenixnap.com/kb/install-kubernetes-on-ubuntu https://computingforgeeks.com/deploy-kubernetes-cluster-on-ubuntu-with-kubeadm/

## View all kubernetes resources
```
kubectl api-resources
```
## Get HELP
```
kubectl run --help
```
## Delete something
```
kubectl delete svc servicename
```
## Pods
### Create and edit pods
Using YAML
```
kubectl create -f newpod.yaml
```
Using command
```
kubectl run <name-of-pod> --image=<image>
```
View pod yaml without creating pod
```
kubectl run <name-of-pod> --image=<image> --dry-run=client -o yaml > pod.yaml
```
Change pod image
```
kubectl set image pod <pod-name> <current-image>=<new-image>
```
Edit pod
```
kubectl edit pod <pod-name>
```
### View pod information
View Labels
```
kubectl get pods --show-labels
```
Filter using labels
```
kubectl get pods --selector=foo=bar
```
```
kubectl get pods -l foo=bar
```
View the yaml file
```
kubectl get pod <pod-name> -o yaml
```
View more info
```
kubectl get pod <pod-name> -o wide
```
Describe a pod
```
kubectl describe pod <pod-name>
```
### Add/ Edit/ Delete Labels
Add labels
```
kubectl label pod <pod-name> k1=v1 k2=v2 k3=v3
```
Add labels by filtering pods
```
kubectl label pod <pod-name> k1=v1 k2=v2 k3=v3 -l key=selected
```
Delete Labels
```
kubectl label pod <pod-name> k1-
```
Edit Labels
```
kubectl label pod <pod-name> k2=v2.1 --overwrite

```
### Add/ Edit/ Delete Annotations
Add Annotation
```
kubectl annotate pod <pod-name> k1=v1 k2=v2 k3=v3
```
Delete Annotation
```
kubectl annotate pod <pod-name> k1-
```
Edit Annotation
```
kubectl annotate pod <pod-name> k2=v2.1 --overwrite
```
### Create a pod with arguments 
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - image: ubuntu
    name: mypod
    args:
    - /bin/sh
    - -c
    - touch /niran/info.txt; sleep 600
```
### Setting Pod Resource Limits and Requirements
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - image: ubuntu
    name: mypod
    resources:
        limits:
            cpu: "2"
            memory: "33Gi"
        requests:
            cpu: "0.4"
            memory: "36Mi"
```
## Namespaces
List Namespacses
```
kubectl get namespaces
```
Create Namespace
```
kubectl create namespace mynamespace
```
List pods from all namespaces
```
kubectl get pods -A 
```
```
kubectl get pods --all-namespaces
```
List pods under a namespacse
```
kubectl get pods -n mynamespace
```
Create a pod in a differenet namespace
```
kubectl run ntest --image=nginx -n mynamespace
```
## Services
### Simple Service
1. Create pod yaml 
```
kubectl run niranpod --image=nginx --dry-run=client -o yaml
```
```
kubectl run niranpod --image=nginx --dry-run=client -o yaml >> info.yaml
```
2. Create pod.
```
kubectl create -f info.yaml
```
3. View pod
```
kubectl get pod -o wide
```
4. Delete pod.
```
kubectl delete pod niranpod
```
5. Expose port 80 while creating pod.
```
kubectl run niranpod --image=nginx --port=80 --dry-run=client -o yaml >> info.yaml
```
```
kubectl create -f info.yaml
```
### Expose using a service

1. Create a service. 
https://kubernetes.io/docs/concepts/services-networking/service/

```
apiVersion: v1
kind: Service
metadata:
  name: niranservice
spec:
  selector:
    type: web-server
  ports:
    - protocol: TCP
      port: 80
```
```
kubectl create -f service.yaml 
```
2. Create the pod. (Add the service selector as a label in the pod)
```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: niranpod
    type: web-server
  name: niranpod
spec:
  containers:
  - image: nginx
    name: niranpod
    ports:
    - containerPort: 80
```
```
kubectl create -f info.yaml 
```
3. View service
```
kubectl get svc
```
### Expose the service to outside
1. Delete the previous service.
```
kubectl delete svc basicservice
```
2. Change the type.
```
apiVersion: v1
kind: Service
metadata:
  name: niranservice
spec:
  selector:
    type: web-server
  type: NodePort
  ports:
    - protocol: TCP
      port: 80
```
```
kubectl create -f service.yaml 
```
4. Access via the public IP.
```
curl <public-ip>:<node-port>
```
## Expose a pod using ClusterIP/NodePort Service
```
kubectl run svcpod --image=nginx
```
```
kubectl label pod svcpod new=test
```
#### ClusterIP
```
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  name: newsvc
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:
    new: test
```
#### NodePort
```
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  name: npsvc
spec:
  ports:
  - port: 80
    protocol: TCP
    nodePort: 32700
  type: NodePort
  selector:
    new: test
```

## Ingress
Create a pod
```
kubectl run ingresspod --image=nginx
```
Create service 
```
apiVersion: v1
kind: Service
metadata:
  name: ingresssvc      
spec:
  selector:
    ex: ingressexample
  ports:
    - protocol: TCP
      port: 80
```
Map the service
```
kubectl label pod inpod ex=ingressexample
```
Creating a ingress
```
kubectl create ingress demo --class=nginx --rule="test.com/*=ingresssvc:80"
```
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: demo
  namespace: default
spec:
  ingressClassName: nginx
  rules:
  - host: test.com
    http:
      paths:
      - backend:
          service:
            name: ingresssvc
            port:
              number: 80
        path: /
        pathType: Prefix
```
Now, forward a local port to the ingress controller:
```
kubectl port-forward --namespace=ingress-nginx service/ingress-nginx-controller 8080:80
```
Access the pod
```
curl http://test.com:8080/
```

## Multi-Container Pods

1. Create a basic multi-container pod
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: niranpod
    type: webserver
  name: niranpod
spec:
  containers:
  - image: nginx
    name: niranpod
    ports:
    - containerPort: 80
  - name: fdlogger
    image: fluent/fluentd
```
```
kubectl create -f info.yaml
```
Get the teminal of a one specific container
```
kubectl exec -it niranpod -c fdlogger -- /bin/bash
```
## Deployments
Create a simple deployment
```
kubectl create deployment niranpod --image=nginx
```
```
kubectl create deployment niranpod --image=nginx -n mynamespace
```
Connect to a container
```
kubectl exec -i​t <Pod-Name> -- /bin/bash
```
Scaling Deployments
```
kubectl scale deploy/dev-web --replicas=4
```
Edit Deployment
```
kubectl edit deployment nginx
```
## Configure Probes
### Liveness
1. Create a pod which creates a directory called "/tmp/niran" while starting. Then removes it after 30 seconds.
```                                                                     
apiVersion: v1
kind: Pod
metadata:
  name: liveness
spec:
  containers:
  - image: ubuntu
    name: liveness
    args:
    - /bin/sh
    - -c
    - touch /tmp/niran; sleep 30; rm -f /tmp/niran; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/niran
      initialDelaySeconds: 5
      periodSeconds: 5
```
- periodSeconds - specifies that the kubelet should perform a liveness probe every 5 seconds. 
- initialDelaySeconds - tells the kubelet that it should wait 5 seconds before performing the first probe. 
- To perform a probe, the kubelet executes the command cat /tmp/healthy in the target container. 
- If the command succeeds, it returns 0, and the kubelet considers the container to be alive and healthy. 
- If the command returns a non-zero value, the kubelet kills the container and restarts it.

2. Wait 30 seconds and observe.

### Readiness
1. Create a pod which creates a folder called "/tmp/niran" while starting. 
```
apiVersion: v1
kind: Pod
metadata:
  name: readiness
spec:
  containers:
  - image: ubuntu
    name: readiness
    args:
    - /bin/sh
    - -c
    - touch /tmp/niran; sleep 600
    readinessProbe:
      exec:
        command:
        - cat
        - /tmp/niran
      initialDelaySeconds: 5
      periodSeconds: 10
```
2. Wait 5 seconds and check the ready status.

## Jobs
```
kubectl create job niran-job --image=busybox 
```
```
kubectl create cronjob niran-job --image=busybox --schedule="*/1 * * * *" -- date
```
### Create a Job

Job will run a container that sleeps for 40s and then stops
```
apiVersion: batch/v1
kind: Job
metadata:
  name: sleep
spec:
  template:
    spec:
      containers:
      - name: sleep
        image: busybox
        command: ["/bin/sleep"]
        args: ["40"]
      restartPolicy: Never
```
```
kubectl apply -f job.yaml
```
**parameters**
- completions - How manay times need to run the job
- parallelism - How many jobs to be started parally
- activeDeadlineSeconds - Deadline for all the jobs

### Create a CronJob
CronJob will create a watch loop. Then create a batch job when the time constraint become true. Below will run after 1m.
```
apiVersion: batch/v1
kind: Job
metadata:
  name: cronjob
spec:
  schedule: "*/2****"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: cronjobimg
            image: busybox
            command: ["/bin/sleep"]
            args: ["30"]
          restartPolicy: Never
```
```
kubectl apply -f job.yaml
```

## ConfigMaps

### Create a configmap
```
kubectl create configmap colors --from-literal=test=black --from-file=./green.txt --from-file=./colors/
```
### View configmap
```
kubectl get configmap colors -o yaml
```
### Example: This Pod mounts a ConfigMap in a volume

```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    configMap:
      name: myconfigmap
```
This will cooy the content of the configmaps to /etc/foo location 
### Example: This Pod uses values from configmap to create an enviroment variable 
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod2
spec:
  containers:
  - name: mypod2
    image: redis
    env: 
      - name: TEST_ENV
        valueFrom:
          configMapKeyRef:
            name: colors
            key: test
```
This pod will create an enviroment variable name TEST_ENV which contains the value from colors configmap

Check enviriment variables
```
kubectl exec -it podname -- /bin/bash -c 'env'
```
## Volumes
### Script to setup NFS on linux machine
Configure the NFS server
```
#!/bin/bash
sudo apt-get update && sudo apt-get install -y nfs-kernel-server
sudo mkdir /opt/sfw
sudo chmod 1777 /opt/sfw/
sudo bash -c "echo software > /opt/sfw/hello.txt"
sudo bash -c "echo '/opt/sfw/ *(rw,sync,no_root_squash,subtree_check)' >> /etc/exports"
sudo exportfs -ra
echo
echo "Should be ready. Test here and second node"
echo
sudo showmount -e localhost
```
Install NFS on second node.
```
sudo apt-get -y install nfs-common nfs-kernel-server
```
Test and see the exported directory using showmount from you second node
```
showmount -e hostname-of-the-node-one
```
Mount the directory
```
sudo mount cp:/opt/sfw /mnt
```
### Create a PersistentVolume
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pvvol-niran
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /opt/sfw
    server: mycp
    readOnly: false
```
Now persistent volume claim (pvc) can will be needed to use this in a Pod.  
### Create a PersistentVolumeClaim
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-one
spec:
  accessModes:
  - ReadWriteMany
  resources:
     requests:
       storage: 200Mi
```
### Add a volume to a pod
```
apiVersion: v1
kind: Pod
metadata:
  name: vol-pod
spec:
  containers:
  - name: vol-pod
    image: redis
    volumeMounts:
    - name: nfs
      mountPath: "/etc/nfs"
  volumes:
  - name: nfs
    persistentVolumeClaim:
      claimName: pvc-one
```

## Rolling Updates and Rollbacks
1. Create an image with redis image. 
```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: testrollout
  name: testrollout
spec:
  replicas: 1
  selector:
    matchLabels:
      app: testrollout
      labels:
        app: testrollout
    spec:
      containers:
      - image: redis
        name: redis
```
2. Change the image to nginx 
```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: testrollout
  name: testrollout
spec:
  replicas: 1
  selector:
    matchLabels:
      app: testrollout
      labels:
        app: testrollout
    spec:
      containers:
      - image: nginx
        name: redis
```
```
kubectl apply -f pod.yaml
```
3. List down history
```
kubectl rollout history deployment testrollout
```
4. Compare the output of therollout history for the two revisions
```
kubectl rollout history deployment testrollout --revision=1
```
```
kubectl rollout history deployment testrollout --revision=2
```
5. View what would be undone using the–dry-run option while undoing the rollout.
```
kubectl rollout undo --dry-run=client deployment/testrollout
```
6. Undo the changes
```
kubectl rollout undo deployment testrollout --to-revision=1
```

## Set SecurityContext for a Pod and Container
Example: Allows access to a shell, but not much more. We will add a runAsUser to both the pod and the container.
```
apiVersion: v1
kind: Pod
metadata:
  name: niran-app
spec:
  securityContext:
    runAsUser: 1000
  containers:
  - name: busy 
    image: busybox
    command:
      - sleep
      - "3600"
    securityContext:
      runAsUser: 2000
      allowPrivilegeEscalation: false
```
Then login to the pod and execute 'ps aux'
```
kubectl exec -it niran-app -- sh
```
Result
```
~ $ ps aux
PID   USER     TIME  COMMAND
    1 2000      0:00 sleep 3600
    8 2000      0:00 sh
   16 2000      0:00 ps aux
```
### Check the capabilities of the kernel
```
grep Cap /proc/1/status
```
Output
```
CapInh: 0000000000000000
CapPrm: 0000000000000000
CapEff: 0000000000000000
CapBnd: 00000000a80425fb
CapAmb: 0000000000000000
```
Decode the output
```
capsh --decode=00000000a80425fb
```
Result
```
0x00000000a80425fb=cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap
```
### Include new capabilities for the container
```
apiVersion: v1
kind: Pod
metadata:
  name: niran-app
spec:
  securityContext:
    runAsUser: 1000
  containers:
  - name: busy 
    image: busybox
    command:
      - sleep
      - "3600"
    securityContext:
      runAsUser: 2000
      allowPrivilegeEscalation: false
      capabilities:
        add: ["NET_ADMIN","SYS_TIME"]
```

## Create and Consume Secrets

### Base64 in terminal
```
echo niran123 | base64
```
### Create a secret
```
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
data:
  password: bmlyYW4xMjM=
```
```
kubectl create secret generic test-secret --from-literal=username=testuser --from-literal=password=thisispwd
```
### Use the Secret
Method one
```
apiVersion: v1
kind: Pod
metadata:
  name: sec-pod
spec:
  containers:
  - name: se-pod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mysecret
```
Method two
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-sec
spec:
  containers:
  - name: pod-sec
    image: redis
    env: 
      - name: TEST_ENV
        valueFrom:
          secretKeyRef:
            name: test-secret
            key: username
```

## ServiceAccounts

1. Creating a ServiceAccounts
```
apiVersion: v1
kind: ServiceAccount
metadata:
 name: secret-sa
```
2. Creating a ClusterRole
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: secret-cr
rules:
- apiGroups:
  - ''
  resources:
  - 'secrets'
  verbs:
  - get
  - list
```
This role will allow pod to list/get secrets 
3. Creating a RoleBinding
```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
 name: secret-rb
subjects:
- kind: ServiceAccount
  name: secret-sa
roleRef:
 kind: ClusterRole
 name: secret-cr
 apiGroup: rbac.authorization.k8s.io
```
4. Testing the ServiceAccount
Creating a pod with a ServiceAccount 
```
apiVersion: v1
kind: Pod
metadata:
  name: satest
spec:
  serviceAccountName: secret-sa
  containers:
  - name: satest
    image: nginx
```
Exec to the pod
```
kubectl exec -it sapod -- sh
```
Check the kubernetes token inside the container
```
cd /var/run/secrets/kubernetes.io/serviceaccount
```
```
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
```
Invoking the kubeapi server to get pods
```
curl -ssSk -H "Authorization: Bearer $TOKEN" https://kubernetes.default:443/api/v1/namespaces/default/pods
```
Result
```
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "pods \"satest\" is forbidden: User \"system:serviceaccount:default:secret-sa\" cannot get resource \"pods\" in API group \"\" in the namespace \"default\"",
  "reason": "Forbidden",
  "details": {
    "name": "satest",
    "kind": "pods"
  },
  "code": 403
}
```
It doesn't give any result. Then invoke the kubeapi server to get secrets
```
curl -ssSk -H "Authorization: Bearer $TOKEN" https://kubernetes.default:443/api/v1/namespaces/default/secrets
```
Result
```
{
  "kind": "SecretList",
  "apiVersion": "v1",
  "metadata": {
    "resourceVersion": "2239150"
  },
  "items": [
    {
      "metadata": {
        "name": "secret-test",
        "namespace": "default",
        "uid": "0c5e1800-f65a-4b8a-adbf-cfafcb28222f",
        "resourceVersion": "2200419",
        "creationTimestamp": "2023-03-01T08:17:41Z",
        "managedFields": [
          {
            "manager": "kubectl-create",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2023-03-01T08:17:41Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {
              "f:data": {
                ".": {},
                "f:password": {}
              },
              "f:type": {}
            }
          }
        ]
      },
      "data": {
        "password": "TEZUckAxbgo="
      },
      "type": "Opaque"
    },
    {
      "metadata": {
        "name": "test-db-secret",
        "namespace": "default",
        "uid": "54a5b34e-c929-4c32-91b6-7dc3f0f1ac6c",
        "resourceVersion": "2201788",
        "creationTimestamp": "2023-03-01T08:31:53Z",
        "managedFields": [
          {
            "manager": "kubectl-create",
            "operation": "Update",
            "apiVersion": "v1",
            "time": "2023-03-01T08:31:53Z",
            "fieldsType": "FieldsV1",
            "fieldsV1": {
              "f:data": {
                ".": {},
                "f:password": {},
                "f:username": {}
              },
              "f:type": {}
            }
          }
        ]
      },
      "data": {
        "password": "aWx1dnRlc3Rz",
        "username": "dGVzdHVzZXI="
      },
      "type": "Opaque"
    }
  ]
}
```

## NetworkPolicy
Create a pod
```
kubectl run podname --image=nginx
```
Expose the webserver using NodePort service and test the connection
```
kubectl expose pod testnet --type=NodePort --port=80
```
Then create below NetworkPolicy
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-default
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```
after that nothing is working. 

## Troubleshooting
```
kubectl get pods secondapp
```
```
kubectl describe pod secondapp
```
```
kubectl logs secondapp webserver
```
Get event flow
```
kubectl get event
```

## Taints and Tolerations
Places a taint on node node1. The taint has key=key1, value=value1, and taint effect=NoSchedule. No pod will be able to schedule onto node1 unless it has a matching toleration.
```
kubectl taint nodes node1 key1=value1:NoSchedule
```
To remove the taint
```
kubectl taint nodes node1 key1=value1:NoSchedule-
```
To schedule a pod into node1 we have to match the toleration to the taint like below
```
apiVersion: v1
kind: Pod
metadata:
  name: y1
  labels:
    env: test
spec:
  containers:
  - name: y1
    image: nginx
    imagePullPolicy: IfNotPresent
  tolerations:
  - key: "key1"
    operator: "Equal"
    value: "value1"
    effect: "NoSchedule"
```
```
apiVersion: v1
kind: Pod
metadata:
  name: y1
  labels:
    env: test
spec:
  containers:
  - name: y1
    image: nginx
    imagePullPolicy: IfNotPresent
  tolerations:
  - key: "key1"
    operator: "Exists"
    value: "value1"
    effect: "NoSchedule"
```

## HELM - Package manager for kubernetes
Add repository
```
helm repo add my-repo https://charts.bitnami.com/bitnami
```
Install 
```
helm install my-releas my-repo/mysql
```
Uninstall
```
helm uninstall my-releas
```
List helm repositories we have registered
```
helm repo list
```
List all charts
```
helm list
```
```
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                           APP VERSION
monitoring      default         1               2023-03-03 04:10:47.812821918 +0000 UTC deployed        kube-prometheus-stack-45.5.0    v0.63.0 
```
### How to Install Prometheus and Grafana
Get Helm Repository Info
`https://artifacthub.io/packages/helm/prometheus-community/kube-prometheus-stack`
Adding the repository
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```
Updating the repository
```
helm repo update
```
Install Helm Chart
```
helm install monitoring prometheus-community/kube-prometheus-stack
```
Expose monitoring-grafana UI
```
kubectl edit svc monitoring-grafana
```
- Change type: NodePort
- Add nodePort
```
ports:
  - name: http-web
    port: 80
    protocol: TCP
    targetPort: 3000
    nodePort: 30001
```
Access using `http://ip-address:30001/login`

### Working with chart values
List values
```
helm show values prometheus-community/kube-prometheus-stack
```
Override helm value after installing the charts. (We can pass the same parameters even with install command)
```
helm upgrade monitoring prometheus-community/kube-prometheus-stack --set grafana.adminPassword=admin 
```
### How to write a values.yaml file
> Edit monitoring-grafana svc through values.yaml file
Make the chnages in values.yaml file and execute below command. 
```
helm upgrade monitoring prometheus-community/kube-prometheus-stack --values=values.yaml
```
###  How do I download Helm Charts
```
helm pull prometheus-community/kube-prometheus-stack
```
```
helm pull prometheus-community/kube-prometheus-stack --untar=true
```
### Instal helm using local files
```
helm install monitoring ./kube-prometheus-stack
```
```
helm upgrade monitoring --values=values.yaml .
```
### How can I create yaml from a Helm Chart
```
helm template monitoring ./kube-prometheus-stack/ --values=./kube-prometheus-stack/myvalue.yaml
```
```
helm template monitoring ./kube-prometheus-stack/ --values=./kube-prometheus-stack/myvalue.yaml > monitor.yaml
```
### Make Your Own Charts
1. Create the blueprint
```
helm create fleetman-helm-chart
```
2. Then rmove the template folder content
3. Create a file inside template called one.yaml
```
hello:
  world: true
```
4. Then run below command 
```
helm template .
```
Result:
```
---
# Source: fleetman-helm-chart/templates/one.yaml
hello:
  world: true
```
5. Remove all the files inside template folder and create file called fleetman.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-webapp
spec:
  selector:
    matchLabels:
      app: webapp
  replicas: {{.Values.numberOfReplicas}}
  template: # template for the pods
    metadata:
      labels:
        app: webapp
    spec:
      containers:
        # image details have been substituted
        # {{- template "webappImage" . }}
        # {{- include "webappImage" . | indent 6 }}
---
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-fleetman-webapp

spec:
  selector:
    app: webapp
  ports:
    - name: http
      port: 80
      nodePort: {{.Values.nodePort}}
  type: NodePort
```
> lower - lowercase characters

> default - define default value if dockerRepoName not defined

> If enviroment equeals to "dev" then only "-dev" will be appended 

> If development equeals to true then only "-dev" will be appended 

> `-` to remove whitelines

> indent - spaces from the begining

6. Then edit the values.yaml file
```
numberOfReplicas : 4
dockerRepoName : richardchesterwood
## enviroment : prod
development: true
nodePort : 30010
```
7. Create a new yaml file called _commonfile.tpl inside template folder
```
{{- define "webappImage" }}
- name: webapp
  ## image: {{ lower .Values.dockerRepoName }}/k8s-fleetman-helm-demo:v1.0.0 
  ## image: {{ .Values.dockerRepoName | default "richardchesterwood" | lower }}/k8s-fleetman-helm-demo:v1.0.0{{ if eq .Values.enviroment "dev" }}-dev{{ end }}
  ## image: {{ .Values.dockerRepoName | default "richardchesterwood" | lower }}/k8s-fleetman-helm-demo:v1.0.0{{ if eq .Values.enviroment "dev" }}-dev{{else}}-prod{{ end }}
  image: {{ .Values.dockerRepoName | default "richardchesterwood" | lower }}/k8s-fleetman-helm-demo:v1.0.0{{ if .Values.development }}-dev{{ end }}
{{- end}}
```
> _ to ignore by helm

8. Check the output
```
helm template .
```
9. Install the chart for dev
```
helm install dev-fleet .
```
10. Install the chart for prod
```
helm install prod-fleet . --set development=false --set nodePort=30012
```

---

## Helpful Commands

1. Creation of pods
```
kubectl run nginx-pod --image=nginx
```
2. Creation of config maps
```
kubectl create cm text-cm --from-literal=name=niran
kubectl create cm file-cm --from-file=info.txt
```
3. Creation of secrets
```
kubectl create secret text-sc --from-literal=name=niran
kubectl create secret file-sc --from-file=sc.txt'
```
4. Creation of deployments
```
kubectl create deployment nginx-dep --image=nginx 
kubectl create deployment ubuntu-dep --image=ubuntu --replicas=3
```
5. Modify properties of an existing deployment
```
kubectl scale deployment nginx-dep --replicas=3
kubectl set image deployment ubuntu-dep ubuntu=ubuntu:14
```
6. Performing rollout actions on existing deployment
```
kubectl rollout status deployment nginx-dep
kubectl rollout history deployment nginx-dep
kubectl rollout undo deployment nginx-dep --to-revision=2
```
7. Creating services for exposing deployments
```
kubectl expose deployment nginx-dep --type=ClusterIP --port=8080 --target-port=80 --name=nginx-clusterip-svc
kubectl expose deployment ubuntu-dep --type=NodePort --port=8080 --target-port=80 --name=nginx-nodeport-svc
```
8. Creating pods with additional properties
```
kubectl run alpine-pod --image=alpine --requests=cpu=250m,memory=512Mi --limits=cpu=500m,memory=1024Mi
```
9. Creating deployment with additional properties
```
kubectl create deployment ubuntu-dep --image=ubuntu --replicas=5 -- sleep 600
kubectl create deployment nginx-dep --image=nginx --replicas=6 --port=80
```
10. Creating jobs and crons
```
kubectl create job test-job --image=ubuntu -- sleep 300
kubectl create cronjob test-cj --image=ubuntu --schedule="* * * * *" -- sleep 400
```
11. Create a service account
```
kubectl create sa sa-name
```
12. Create role/clusterrole
```
kubectl create role role-name --verb=get,list --resources=pods,services
kubectl create clusterrole clusterrole-name --verb=get,list --resources=pods,services
```
13. Create role bindings
```
kubectl create rolebinding rb --role=role-name --serviceaccount=default:sa-name
kubectl create clusterrolebinding crb --role=clusterrole-name --serviceaccount=default:sa-name
```
14. Dry-run potion
```
kubectl create deployment nginx-dep --dry-run=client -o yaml
kubectl expose deployment nginx-dep --type=ClusterIP --port-80 --name=nginx-svc --dry-run=client -o yaml
kubectl create sa sanew --dry-run=client -o yaml
```
15. Create a temparory pod to wget to another pod
```
kubectl run wget-test --image=busybox --rm -it --restart=Never -- wget -O- ipaddress:80
```
16. Create an image with enviroment variable
```
kubectl run env-pod --image=nginx --env="name=Niran Peiris"
```