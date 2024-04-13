![architecture-k8s-mongo-ui](https://github.com/mathesh-me/k8s-mondo-admin-interface/assets/144098846/1736534c-7f5d-4ad6-b96a-74dcf2b01777)# MongoDB Admin Interface Deployment using Kubernetes 🚀

This Repository describes the deployment of a MongoDB Admin Interface using Kubernetes. The MongoDB Admin Interface is a web-based application that allows users to interact with a MongoDB database via Web Interface. This Web Inteface provides a user-friendly interface for performing common database operations such as querying, inserting, updating, and deleting data.

## Architecture Diagram 📊

![architecture-k8s-mongo-ui](https://github.com/mathesh-me/k8s-mondo-admin-interface/assets/144098846/2e95079f-302d-4a6f-b5d7-1e8c5d84889a)


## Architecture 🏗️

The architecture of the MongoDB Admin Interface deployment using Kubernetes is as follows:

- We are going to Create two deployments one for the application and one for the database.

**Web Application Deployment:**

- We are using the `mongo-express:latest` image for creating the application.
- We use deployment to create the pods because if the pod goes down, the deployment will create a new pod, Deployment will help us manage the lifecycle of the pod. It also gives us the feature of scaling the application and high availability.
- For exposing the Web to the outside world, We are using the service with `NodePort` Type.
- **Nodeport:** It is a service that exposes the deployment outside the cluster.

**Database Deployment:**

- We are using the `mongo:5.0` image for creating a deployment for the database.
- We are again using deployment to create the pods because if the pod goes down, the deployment will create a new pod.
- For communication of DB with the frontend/backend, we are using the `ClusterIP` service.
- **ClusterIP:** It is a service type that exposes the deployment within the cluster.

**Other Kubernetes Resources:**
- **Secrets:** We are using the secrets feature to store sensitive information like `username` and `password`.
- **ConfigMap:** We are using ConfigMap to store the non-sensitive data in key-value pairs, In our case we are using it to store the `database URL`.
- **PersistentVolume and PersistentVolumeClaim:** We are using PV and PVC to store the data in the database, So that if our pod goes down, the data will be safe.

## Prerequisites 📋

- Kubernetes Cluster
- Kubernetes CLI `kubectl`
- Knowledge of Kubernetes Pods, Deployment, Services, Secrets, ConfigMap, PersistentVolume, and PersistentVolumeClaim.

## Steps 📝

| Step No | Document Link |
| ------ | ------ |
| 1 | [Steps to Deploy MongoDB Admin Interface using K8S][Step-1] |

   [Step-1]: <./Steps/step1.md>

## Contribution 🤝

Contributions are always welcomed. Make a pull request or open an issue if you find any.

## License 📜

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details

## Author 👨‍💻

- [Mathesh M](https://www.linkedin.com/in/mathesh-me/) on LinkedIn.
- You Can also check out my [Medium](https://medium.com/@mathesh-me) for articles on DevOps Tools and Technologies.️
