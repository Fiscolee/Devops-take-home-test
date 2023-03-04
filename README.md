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
kubectl create -f deployment.yaml -n usermanagement
```
I ran the similar commands for the shift application

'''
kubectl create namespace shiftmanagement
kubectl create -f shift-configmap.yaml -n shiftmanagement
kubectl create -f shift-deployment.yaml -n shiftmanagement
kubectl create -f shift-service.yaml -n shiftamangement
'''

I checked application pods by running the following command
'''
kubectl get pods -A
'''
Below image shows the feedback from my EKS cluster
'''
NAMESPACE         NAME                           READY   STATUS    RESTARTS   AGE
kube-system       aws-node-85c7r                 1/1     Running   0          44m
kube-system       aws-node-dl79b                 1/1     Running   0          44m
kube-system       coredns-744dc4f967-4vxwn       1/1     Running   0          56m
kube-system       coredns-744dc4f967-qsqj6       1/1     Running   0          56m
kube-system       kube-proxy-4qtmg               1/1     Running   0          44m
kube-system       kube-proxy-plcdp               1/1     Running   0          44m
shiftmanagement   shiftswebapp-b8d9c8bcc-p674t   1/1     Running   0          46s
usermanagement    userswebapp-68547fc79b-9mmxf   1/1     Running   0          4m7s
![image](https://user-images.githubusercontent.com/111366682/222901537-62b95df0-a25b-444a-9f66-909207561333.png)

