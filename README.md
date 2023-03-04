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

```
kubectl create namespace shiftmanagement
kubectl create -f shift-configmap.yaml -n shiftmanagement
kubectl create -f shift-deployment.yaml -n shiftmanagement
kubectl create -f shift-service.yaml -n shiftamangement
```

I checked application pods by running the following command
```
kubectl get pods -A
```
Below image shows the feedback from my EKS cluster
```
NAMESPACE         NAME                           READY   STATUS    RESTARTS   AGE
kube-system       aws-node-85c7r                 1/1     Running   0          44m
kube-system       aws-node-dl79b                 1/1     Running   0          44m
kube-system       coredns-744dc4f967-4vxwn       1/1     Running   0          56m
kube-system       coredns-744dc4f967-qsqj6       1/1     Running   0          56m
kube-system       kube-proxy-4qtmg               1/1     Running   0          44m
kube-system       kube-proxy-plcdp               1/1     Running   0          44m
shiftmanagement   shiftswebapp-b8d9c8bcc-p674t   1/1     Running   0          46s
usermanagement    userswebapp-68547fc79b-9mmxf   1/1     Running   0          4m7s
```
![image](https://user-images.githubusercontent.com/111366682/222901537-62b95df0-a25b-444a-9f66-909207561333.png)

I checked the services by running

```
kubectl get services -A
```
Below image shows feedback 
```
NAMESPACE         NAME           TYPE           CLUSTER-IP       EXTERNAL-IP                                                               PORT(S)          AGE
default           kubernetes     ClusterIP      10.100.0.1       <none>                                                                    443/TCP          3h7m
kube-system       kube-dns       ClusterIP      10.100.0.10      <none>                                                                    53/UDP,53/TCP    3h7m
shiftmanagement   shiftswebapp   LoadBalancer   10.100.25.146    a3f05fadb48704d2bb3e2b56f7558ad3-2090583156.us-east-2.elb.amazonaws.com   8080:32702/TCP   4m16s
usermanagement    userswebapp    LoadBalancer   10.100.105.129   acc258795165a498495ea92345559dcc-1706455890.us-east-2.elb.amazonaws.com   8080:30668/TCP   8s
```
![image](https://user-images.githubusercontent.com/111366682/222902072-f37ad130-bc60-45f5-b9e1-03e85d35e2cc.png)

