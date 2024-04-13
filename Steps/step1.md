### Steps to deploy Mern Application on Kubernetes

Before we start with the deployment, We have to Create a Directory `/storage/mongo` in the minikube node to store the data of the database. We can do this by using the below command.

```bash
docker exec -it minikube bash
sudo mkdir -p /storage/mongo
exit
```

1. **Create a PersistentVolume and PersistentVolumeClaim for MongoDB**

- Create a file named `pv.yaml` and add the below content.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: db-pv
spec:
  capacity:
    storage: 0.5Gi
  accessModes: 
    - ReadWriteMany
  local:
    path: /storage/mongo
  nodeAffinity:
    required:
      nodeSelectorTerms:
        - matchExpressions:
            - key: kubernetes.io/hostname
              operator: In
              values:
                - minikube
```

**Parameter Explanation:**

- We are creating a PersistentVolume for the mongo database.
- **capacity:** To define the capacity of the PV.
- **accessModes:** To define the accessModes.
- **local:** To define the path.
- **nodeAffinity:** To define the nodeAffinity.

- Create a file named `db-pvc.yaml` and add the below content.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: db-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 0.3Gi
  storageClassName: ""
```

**Explanation:**

- We are creating a PersistentVolumeClaim for the mongo database.
- **accessModes:** To define the accessModes, here it is ReadWriteMany
- **resources:** To define the resources.
- **storageClassName:** To define the storageClassName.

2. **Create a Secret for MongoDB**

- Create a file named `db-secret.yaml` and add the below content. In Kubernetes, we can use secrets to store sensitive information like passwords, OAuth tokens, and SSH keys.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: YWRtaW4=
  password: bXlnZmc=
```

**Param Explanation**

- We are creating a secret for the mongo database.
- **type:** Type of the secret.
- **data:** To define the data.
- **mongo-user:** The username for the mongo database.
- **mongo-password:** The password for the mongo database.

3. **Create a ConfigMap for MongoDB**

- Create a file named `db-config.yaml` and add the below content. In Kubernetes, we can use ConfigMaps to store non-sensitive data in key-value pairs.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: db-config
data:
  url: db-service 
```

**Param Explanation:**

- We are creating a configMap for the mongo database.
- **data:** To define the data.
- **mongo-url:** We give mongo-service as the value, So that the webapp can use the mongo-service to communicate with the database.

4. **Create a Deployment for MongoDB**

- Create a file named `db-depl.yaml` and add the below content.

```yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: db-deployment
  labels:
    app: db
spec:
  replicas: 1
  selector:
    matchLabels:
      app: db
  template:
    metadata:
      name: db-pod
      labels:
        app: db
    spec:
      containers:
      - name: db-container
        image: mongo:5.0
        ports:
        - containerPort: 27017
        volumeMounts:
        - mountPath: /data/db
          name: mongo-db-volume
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: username
        - name: MONGO_INITDB_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
      volumes:
      - name: mongo-db-volume
        persistentVolumeClaim:
          claimName: db-pvc
```

**Parameters Description**

- We are creating a deployment for the mongo database.
- We are using the `mongo:5.0` image.
- **labels:** To identify the deployment.
- **replicas:** Number of pods to be created.
- **selector:** To select the pods.
- **template:** Define pod structure.
- **containers:** Specify container details.
- **volumeMounts:** To define the mount path for the volume.
- **volumes:** To define the volume.
- **env:** To define the environment variables.
- **persistentVolumeClaim:** To claim the PV.
- **metadata:** To define the metadata.
- **secretKeyRef:** To refer to the secretKeyRef.
- **matchLabels:** To match the labels we defined.


5. **Create a Deployment for MongoDB Web**

- Create a file named `web-depl.yaml` and add the below content.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
  labels:
    app: web
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      name: web-pod
      labels:
        app: web
    spec:
      containers:
      - name: web-container
        image: mongo-express:latest
        ports:
        - containerPort: 8081
        env:
        - name: ME_CONFIG_MONGODB_ADMINUSERNAME
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: username
        - name: ME_CONFIG_MONGODB_ADMINPASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
        - name: ME_CONFIG_MONGODB_SERVER
          valueFrom:
            configMapKeyRef:
              name: db-config
              key: url
```

**Parameters Description**

- We are creating a deployment for the webapp.
- We are using the `mongo-express:latest` image.
- **labels:** To identify the deployment.
- **replicas:** Number of pods to be created.
- **selector:** To select the pods.
- **template:** To create the pods.
- **containers:** To define the container details.
- **volumeMounts:** To mount the volume.
- **volumes:** To define the volume.
- **env:** To define the environment variables.
- **configMapKeyRef:** To refer to the configMap.
- **valueFrom:** To refer to the secretKeyRef.


**Now that we have created the application and database deployments, But How the application will communicate with the database?**

- We need to create a service for the database to communicate with the frontend/backend.

6. **Create a Service for MongoDB Web**

- Create a file named `web-svc.yaml` and add the below content.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: NodePort
  selector:
    app: web
  ports:
    - protocol: TCP
      port: 8081
      targetPort: 8081
      nodePort: 30111 
```

**Param Description**

- We are creating a service for the mongo database.
- We used `ClusterIP` type to communicate within the cluster Because we didn't want to expose the database to the outside world.
- **selector:** To select the pods.
- **type:** Type of the service.
- **ports:** To define the ports.
- **protocol:** Protocol to be used.
- **port:** The port number we want to access our service.
- **targetPort:** The port number, the container is listening on.

7. **Create a Service for DB**

- Create a file named `db-svc.yaml` and add the below content.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: db-service
spec:
  selector:
    app: db
  type: ClusterIP
  ports:
    - protocol: TCP
      port: 27017
      targetPort: 27017 
```

**Parameters Description**

- We are creating a service for the webapp.
- We used `NodePort` type to communicate with the outside world.
- **selector:** To select the pods.
- **type:** Type of the service.
- **ports:** To define the ports.
- **protocol:** Protocol to be used.
- **port:** The port number we want to access our service.
- **targetPort:** The port number, the container is listening on.

**Commands for the required files, Apply after you created each file**

```bash
kubectl apply -f pv.yaml
kubectl apply -f pvc.yaml
kubectl apply -f db-secret.yaml
kubectl apply -f db-config.yaml
kubectl apply -f db-depl.yaml
kubectl apply -f web-depl.yaml
kubectl apply -f web-svc.yaml
kubectl apply -f db-svc.yaml
```

**Commands to check the status of the pods and services.**

```bash
kubectl get pods
kubectl get deployments
kubectl get svc
kubectl get secrets
kubectl get configmaps
kubectl get pv
kubectl get pvc
```

**Now we can access the webapp using the NodePort service.**

- Note we used `Nodeport` service to expose the webapp to the outside world of our k8s cluster.
- Now you can access the webapp using the `http://<Node-IP>:30111` URL.
- But if You are used `minikube` to create the cluster, You have to follow the below method.
    - Install Socat using `yum install socat -y` command.
    - Use `socat TCP4-LISTEN:8080,fork,reuseaddr TCP4:192.168.49.2:30111 &` command to access the webapp.

### Result:

// Will be Added

**That's it, We have successfully deployed the Mern Application on Kubernetes.**

### References:

- [Kubernetes](https://kubernetes.io/docs/home/)
- [MongoDB](https://www.mongodb.com/)
- [Mongo-Express](https://www.npmjs.com/package/mongo-express)    
- [Kubernetes Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
- [Kubernetes ConfigMap](https://kubernetes.io/docs/concepts/configuration/configmap/)
- [Kubernetes PersistentVolume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
- [Kubernetes PersistentVolumeClaim](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)
- [Kubernetes Services](https://kubernetes.io/docs/concepts/services-networking/service/)
- [Kubernetes Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Kubernetes Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod/)
- [Kubernetes Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)
- [Kubernetes NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#nodeport)
- [Kubernetes ClusterIP](https://kubernetes.io/docs/concepts/services-networking/service/#clusterip)
- [Kubernetes Minikube](https://minikube.sigs.k8s.io/docs/start/)
- [Kubernetes Kubectl](https://kubernetes.io/docs/reference/kubectl/overview/)