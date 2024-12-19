# Kubernetes Project - Web Application with MongoDB

## NOTES : 

- Le projet a été réalisé presque sans IA, uniquement pour la compréhension de concepts, et l'édition du README.md.
- La segmentation actuelle fait que l'utilisation des Namespace n'est pas obligatoire selon moi.

## 📘 **Project Overview**
This project consists of a Kubernetes deployment that runs a **web application** and a **MongoDB database**. The architecture ensures persistence of data, proper resource allocation, secure environment variables, and log sharing between containers.

The application has two primary components:
1. **Web Application**: A Node.js-based web app with logs shared with an Alpine-based log collector.
2. **MongoDB Database**: A MongoDB instance with persistent storage via a Persistent Volume (PV) and a Persistent Volume Claim (PVC).

The services are exposed using a **LoadBalancer** for the web application and an **internal ClusterIP service** for MongoDB.

---

## 📂 **Project Structure**
```
├── mongo-config.yaml     # ConfigMap of the mongo service
├── mongo-pvc.yaml        # Pv and PVC of mongo
├── mongo-secret.yaml     # Secret of the mongo DB
├── mongo.yaml            # Config of the mongo DB
├── webapp.yaml           # Config of the webapp, and the log collector
└── README.md             # This documentation file
```

---

## 🚀 **Deployment Instructions**

1. **Apply the MongoDB Deployment**
   ```bash
   kubectl apply -f mongo-secret.yaml
   kubectl apply -f mongo-config.yaml
   kubectl apply -f mongo-pvc.yaml
   kubectl apply -f mongo.yaml
   ```
   This will create a **MongoDB Deployment** with a Persistent Volume Claim (PVC) to ensure that data is not lost if the pod is deleted or restarted.

2. **Apply the Web Application Deployment**
   ```bash
   kubectl apply -f webapp.yaml
   ```
   This will create a **Web Application Deployment** that is accessible via a LoadBalancer service.

3. **Check the status of the pods, services, PVC, and PV**
   ```bash
   kubectl get pods
   kubectl get services
   kubectl get pvc
   kubectl get pv
   ```
   Ensure that all components are running correctly.

---

## ⚙️ **Configuration Details**

### **1️⃣ Web Application**

**Deployment Details:**
- **Replicas**: 1
- **Image**: `nanajanashia/k8s-demo-app:v1.0`
- **Ports**: 3000 (accessible via LoadBalancer)
- **Volumes**: Uses a shared log volume (`emptyDir`) to persist logs between two containers in the same pod.

**Containers in Pod:**
- **Web Application**: Runs `node server.js` and writes logs to `/app/logs/test.log`.
- **Alpine Log Collector**: Runs `tail -f /app/logs/*.log` to continuously display log file changes.

**Example Command to View Logs:**
```bash
kubectl logs <webapp-pod-name> -c alpine-log -f
```

![Capture d'écran 2024-12-17 095138](https://github.com/user-attachments/assets/922bab46-b891-44ce-b402-5771d239c602)

**Resource Requests & Limits:**
- **CPU**: Requests 250m, Limits 500m
- **Memory**: Requests 64Mi, Limits 128Mi

**Service Details:**
- **Type**: LoadBalancer (Exposes port 3000 to the outside world)
- **Port**: 3000

---

### **2️⃣ MongoDB**

**Deployment Details:**
- **Replicas**: 1
- **Image**: `mongo:5.0`
- **Ports**: 27017 (MongoDB default port)
- **Persistent Storage**: Uses a **Persistent Volume Claim (PVC)** to ensure data persistence.
- **Environment Variables**: Securely configured using **Secrets**.

**Persistent Volume (PV) and Persistent Volume Claim (PVC):**
- **Persistent Volume**: Binds 1Gi of storage to the MongoDB container.
- **PVC**: Ensures persistent storage for `/data/db` within the MongoDB container.

**Service Details:**
- **Type**: ClusterIP (only accessible within the cluster)
- **Port**: 27017 (the port used to connect to the database)

**Check MongoDB Logs:**
```bash
kubectl logs <mongo-pod-name>
```
---

## 🔐 **Secrets and ConfigMaps**

### **Secrets**
| **Name**       | **Key**         | **Value**      |
|----------------|-----------------|----------------|
| `mongo-secret` | `mongo-user`    | **(hidden)**   |
| `mongo-secret` | `mongo-password`| **(hidden)**   |

**How to create a Secret:**
```bash
kubectl create secret generic mongo-secret \
  --from-literal=mongo-user=admin \
  --from-literal=mongo-password=secretpassword
```

### **ConfigMaps**
| **Name**       | **Key**         | **Value**        |
|----------------|-----------------|------------------|
| `mongo-config` | `mongo-url`     | `mongodb://<username>:<password>@mongo-service:27017/` |

**How to create a ConfigMap:**
```bash
kubectl create configmap mongo-config --from-literal=mongo-url=mongodb://admin:secretpassword@mongo-service:27017/
```

---

## 📊 **Resource Requests & Limits**
| **Container**       | **CPU Requests** | **CPU Limits** | **Memory Requests** | **Memory Limits** |
|---------------------|------------------|-----------------|---------------------|-------------------|
| **webapp**          | 250m             | 500m            | 64Mi                | 128Mi             |
| **alpine-log**      | 250m             | 500m            | 64Mi                | 128Mi             |
| **mongodb**         | Default          | Default         | Default             | Default           |

---

## 📈 **Monitoring & Logs**
- **Logs** from the web application are written to `/app/logs` and shared with the **Alpine log collector** container.
- **Pod Logs** can be viewed with the following command:
  ```bash
  kubectl logs <pod-name> -c webapp
  kubectl logs <pod-name> -c alpine-log
  ```

---

## 🛠️ **Common Issues & Troubleshooting**

### **Pod Stuck in 'Pending' State**
- Check the events for the pod:
  ```bash
  kubectl describe pod <pod-name>
  ```
- Look for volume attachment issues or missing PVCs.

### **MongoDB Data is Missing After Pod Restart**
- Check that the PVC is bound and attached to the pod:
  ```bash
  kubectl get pvc
  ```
- Verify that the `/data/db` path is properly mounted and not an ephemeral file system.

---

## 🧹 **Clean Up**
To clean up the cluster resources, run:
```bash
   kubectl delete -f mongo-secret.yaml
   kubectl delete -f mongo-config.yaml
   kubectl delete -f mongo-pvc.yaml
   kubectl delete -f mongo.yaml
   kubectl delete -f webapp.yaml
```
This will remove all pods, services, and persistent volumes created by this project.

---

The Mongo pod will have the PV and the PVC attached to it.

```bash
kubectl describe pod mongo-deployment-XXXXXXXX
```
![Capture d'écran 2024-12-17 095024](https://github.com/user-attachments/assets/13bdd0a7-7460-483e-ae82-bcbf48196941)


## 📜 **Authors & Contributors**
This project was created by **Sébastien Grard** as part of a Kubernetes learning experience.
