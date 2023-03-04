# Kubernetes-Challenge
## Set up EKS Cluster in AWS
I set up an AWS EKS Cluster

Create  a new VM with Database (postgresSQL)- This assignment requires apis to return users and shifts details from the database, I created couple of tables. I installed a postgressql in a linux server by using the method stated in the link below:
[https://devopscube.com/install-configure-postgresql-amazon-linux/]

## Create two deployments and deploy them on Kubernetes Cluster
To deploy the application in Kubernetes I created yaml manifests for users and shifts.

### To deploy services in Kubernetes run below commands

### Create namespace
```
kubectl create namespace usermanagement
```

### Deploy Configmap
```
kubectl create -f configmap.yaml -n usermanagement
```

### Deploy User Management application
```
'kubectl create -f deployment.yaml -n usermanagement'
```
