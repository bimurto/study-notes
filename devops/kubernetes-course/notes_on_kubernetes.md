# Notes on Kubernetes

#### See information of cluster

See all pods in the cluster
`kubectl get pods`
with more information
`kubectl get pods -o wide`

See replicasets in cluster
`kubectl get replicasets` or `kubectl get replicasets -o wide`

See services in cluster
`kubectl get services` or `kubectl get services -o wide`

#### Describe information of Kubernetes objects
Get pod information
`kubectl describe pod pod_name`

Get replica set information
`kubectl describe replicaset replicaset_name`

#### Create and run a POD

Run a pod with name nginx with image nginx : 
`kubectl run nginx --image=nginx`

via yaml files `nginx.yml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    tier: front-end
spec:
  containers:
    - name: nginx
      image: nginx
```
Create a pod with the yaml file
`kubectl create -f nginx.yml` 
or 
`kubectl apply -f nginx.yml`



#### Create a ReplicaSet 
``kubectl create replicaset replicaset_name --image=image_name --replicas=2``
Filename : `frontend-replicaset.yaml`
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: mywebsite
    tier: frontend
spec:
  replicas: 4
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
    spec:
      containers:
        - name: nginx
          image: nginx
  selector:
    matchLabels:
      app: myapp
```
`kubectl apply -f frontend-replicaset.yaml`

#### Create a Deployment
`kubectl create deployment deployment_name --image=image_name --replicas=2`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  labels:
    app: mywebsite
    tier: frontend
spec:
  replicas: 4
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
    spec:
      containers:
        - name: nginx
          image: nginx
  selector:
    matchLabels:
      app: myapp
      app: myapp
```
`kubectl apply -f filename.yaml`

#### Create a Service
**NodePort**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: image-processing
  labels:
    app: myapp
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30004
  selector:
    tier: backend
```
`kubectl apply -f filename.yaml`
Selector labels must match with POD.

**ClusterIp**
`kubectl expose deployment deployment_name --name=service_name --target-port=8080 --port=80 --type=ClusterIp`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: image-processing
  labels:
    app: myapp
spec:
  type: ClusterIp
  ports:
    - port: 80
      targetPort: 8080
  selector:
    tier: backend
```
`kubectl apply -f filename.yaml`
Selector labels must match with POD.

#### Create a DaemonSet 
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: elasticsearch
  labels:
    app: elasticsearch
spec:
  template:
    metadata:
      name: elasticsearch-pod
      labels:
        app: elasticsearch
    spec:
      containers:
        - name: elasticsearch
          image: k8s.gcr.io/fluentd-elasticsearch:1.20
  selector:
    matchLabels:
      app: elasticsearch
```

#### Create a yaml definition file from command line
`kubectl run nginx --image=nginx --dry-run=client -o yaml > nginx.yml`

#### Edit a POD
`kubectl edit pod pod_name` 
then update the live pod information
or 
update the yaml file to create the pod and use
`kubectl apply -f file_name.yml`

#### Edit a REPLICASET
`kubectl edit replicaset replicaset_name` 
then update the live replicaset information
or 
update the yaml file to create the replicaset and use
`kubectl apply -f file_name.yml`

#### Delete a POD
`kubectl delete pod pod_name`

#### Delete a Deployment
`kubectl delete deployment deployment
_name`

#### Scale a Replica Set
`kubectl scale replicaset --replicas=num_of_replicas replica_set_name`
or update the yaml file and apply the file.

#### Scale a Deployment Set
`kubectl scale deployment --replicas=num_of_replicas deployment_name`
or update the yaml file and apply the file.


#### Resource Management in POD
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    tier: front-end
spec:
  containers:
    - name: nginx
      image: nginx
      resources:
        requests: 
          memory: "1Gi"
          cpu: 1
        limits:
          memory: "2Gi"
          cpu: 2
```
You can only change 
* spec.containers[*].image
* spec.initContainers[*].image
* spec.activeDeadlineSeconds
* spec.tolerations
for other changes delete the pod and re create it.

#### LimitRange
Default memory limit
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
spec:
  limits:
  - default:
      memory: 512Mi
    defaultRequest:
      memory: 256Mi
    type: Container
```
Default CPU limit
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-limit-range
spec:
  limits:
  - default:
      cpu: 1
    defaultRequest:
      cpu: 0.5
    type: Container
```

### Static PODs
Kubelet can be configured to read yaml files from a local directory. PODs created this way is called Static Pods. Only Pods can be created this way.

While starting `kubelet` need to pass the path as `--pod-manifest-path=/dir`. Or you can pass a config yaml file with `--config`. Config property is `staticPodPath`.

#### Add a custom scheduler in Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    tier: front-end
spec:
  containers:
    - name: nginx
      image: nginx
  schedulerName: my-custom-scheduler
```

## Logging and Monitoring

#### Metrics Server 
**Installation**
`git clone https://github.com/kodekloudhub/kubernetes-metrics-server.git`
`kubectl apply -f ./kubernetes-metrics-server/`

**Monitoring**
`kubectl top node`
`kubectl top pod`

#### Monitor Logs in Kubernetes
`kubectl log -f pod_name`
For multiple container pods
`kubectl log -f pod_name container_name`


## Application Lifecycle Management

#### Update and Rollback of Deployment
Rolling Update - default deployment strategy for new deployment.
Create a deployment
`kubectl apply -f deployment.yml --record`
--record fills out change cause in history
Check Rollout status
`kubectl rollout status deployment/deployment_name`
See Rollout history
`kubectl rollout history deployment/deployment_name`
Undo last rollout
`kubectl rollout undo deployment/deployment_name`

#### Command and Argument
ENTRYPOINT in docker -> command in POD
CMD in docker -> args in POD

#### Environment Variables in POD
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    tier: front-end
spec:
  containers:
    - name: nginx
      image: nginx
      env:
      - key: ENV_VAR
        value: value
      - key: ENV_VAR_CONFIG_MAP
        valueFrom: 
          configMapKeyRef: 
      - key: ENV_VAR_SECRET_KEY
        valueFrom: 
          secretKeyRef: 
```

#### Create a ConfigMap
`kubectl create configmap config_map_name --from-literal=KEY1=value1 --from-literal=KEY2=value2`
`kubectl create configmap config_map_name --from-file=config.properties`
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: config_map_name
data:
  KEY1: value1
  KEY2: value2
```

#### Add configmap in POD
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    tier: front-end
spec:
  containers:
    - name: nginx
      image: nginx
      env:
      - key: ENV_VAR_CONFIG_MAP
        valueFrom: 
          configMapKeyRef: 
            name: config_map_name
            key: KEY1
```
or
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    tier: front-end
spec:
  containers:
    - name: nginx
      image: nginx
      envFrom: 
      - configMapRef: 
          name: config_map_name
```

#### Create a Secret
`kubectl create secret generic secret_name --from-literal=KEY1=value1 --from-literal=KEY2=value2`
`kubectl create secret generic secret_name --from-file=config.properties`
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret_name
data:
  KEY1: base64_encoded(value1)
  KEY2: base64_encoded(value2)
```
use base64 to encode values of secret in yaml file

#### Add secret in POD
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    tier: front-end
spec:
  containers:
    - name: nginx
      image: nginx
      env:
      - key: ENV_VAR_CONFIG_MAP
        valueFrom: 
          secretKeyRef: 
            name: secret_name
            key: KEY1
```
or
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    tier: front-end
spec:
  containers:
    - name: nginx
      image: nginx
      envFrom: 
      - secretRef: 
          name: secret_name
```

#### Create an InitContainer
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
```