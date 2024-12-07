+++
author = "M0D4S"
title = "Chaos Engineering Exploration: Containerization and Test K8S Cluster Deployment."
date = "2024-05-31"
description = "Containerizing the developed features and deploying it to a test K8S cluster"
tags = [
]
+++

## I. Containerization: A Step Toward Portability

Containerization was a crucial step in this project, ensuring the application could run consistently across different environments. By packaging the application and its dependencies into lightweight containers, we eliminated potential issues caused by environment differences during deployment.

### **1. Why Containerization?**
- **Portability**: The application can run identically in development, testing, and production environments.
- **Isolation**: Each container is isolated, ensuring secure and independent execution.
- **Scalability**: Containers can be easily scaled horizontally to handle increased workloads.
- **Efficiency**: Containers are lightweight and utilize resources more efficiently than traditional virtual machines.

---

### **2. How It Was Done**
A. **Dockerization of Backend**:
   - Created a Dockerfile for the Django application.
   - Defined environment variables for database connections and Azure Key Vault secrets.
   - Published the image to Docker Hub for easy access: [Django API Image](https://hub.docker.com/r/elyass359/fssp-django)

B. **Dockerization of Frontend**:
   - Built a Vue.js production-ready image using Node.js.
   - Customized the image for Nginx to serve the static files.
   - Published the image to Docker Hub for easy access: [VueJS Image](https://hub.docker.com/r/elyass359/fssp-vuejs)

C. **Database Setup**:
   - Included PostgreSQL in the containerized stack.
   - Configured persistent storage and secure credentials.

D. **Docker Compose**:
   - Integrated all services (frontend, backend, PostgreSQL) using a `docker-compose.yml` file.
   - Simplified running the entire stack locally for testing.

---

### **3. Challenges and Solutions**
- **Challenge**: Managing configurations for multiple environments (local, test, production).
  - **Solution**: Used `.env` files and Docker Compose to handle environment-specific variables.
- **Challenge**: Reducing the size of Docker images for faster builds and deployments.
  - **Solution**: Leveraged multi-stage builds in the Dockerfiles as well as the use of minimal base images.
- **Challenge**: Encountering difficulties in finding Virtual Machines with x64 processors for AKS nodes.
  - **Solution**: Changed the base images in Dockerfiles to arm64 instead of x64.

Please consult [the github repository of the project](https://github.com/Yassine-Rejeb/fssp) to have a look at the actual definitions of the Dockerfiles.

---

## II. Deployment on a Dev/Test Kubernetes Cluster

### **1. Why Deploying on a Dev/Test Cluster?**

- **Development**: The application's final feature, requiring a running Kubernetes cluster, can be developed and tested locally.
- **Testing**: The application can be tested in a controlled environment before deploying it to production.
- **Validation**: The application can be validated for functionality and security before going live.
- **Scalability**: The application can be tested for scalability and performance under different loads.

---

### **2. How It Was Done**

1. **Kubernetes Setup**:
   - The Dev/Test cluster was created on local ubuntu machines.
   - The cluster was configured with a single control plane node and two worker nodes.
   - The Kubernetes cluster was deployed the kubeadm way.
   - More details on the setup can be found [here](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/).

2. **Manifests**:
   - The Kubernetes manifests are those in the folder `k8s_manifests` of the repository.
   - The manifests include YAML files:
     - `django-service-deployment.yaml`: manifest containing the definitions for the Deployment, Service (primarily NodePort), Secrets and HPA.
     - `vuejs-service-deployment.yaml`: manifest containing the definitions for the Deployment, Service (primarily NodePort) and HPA.
     - `postgresql-service-deployment.yaml`: manifest containing the definitions for the PV, PVC, Secrets, Deployment, Service (primarily NodePort) and HPA.
     - `pgadmin-service.yaml.old`: dev/test ONLY manifest containing the definitions for the Deployment, Service and HPA.
     - `pgadmin-service.yaml`: manifest containing the definitions for the Deployment, Service and HPA.

3. **Up and Running**:
   - The manifests were deployed on the test cluster using `kubectl apply -f <directory>`.
   - The deployments were validated by accessing the services using the NodePorts.

---

## III. Development of the Feature File Management
<!-- ### 1. File Management Frontend
The File Management feature encompasses the creation, deletion, retrieval, share and revocation of the file.

#### File Management Frontend

<!-- ![File Managements UIs](/images/portfolio/fssp/fssp_ui.png)

#### File Management Backend -->

<!-- The following table documents the endpoints of the back-end API that are relevant to the File Management feature. --> -->

## o. What’s Next?
<!-- With the development phase of these two features compete, the next step involved **containerization** and deploying it to a **test Kubernetes cluster**. Stay tuned for the next post, where I’ll discuss this phase in detail.

Next Post on the series: [Chaos Engineering Exploration: Final Feature Development and Setting-up Monitoring and Chaos Mesh.](post/graduation-project-pt3/) -->

