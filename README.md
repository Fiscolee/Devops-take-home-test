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
### Accessing the applications
Once the applications were deployed I was able to access them using my browser

http://a399f5153a514471bafcd4de28b6ff50-1163751732.us-east-2.elb.amazonaws.com:8080/usermanagement/allusers![image](https://user-images.githubusercontent.com/111366682/222902243-125de8dc-65d5-47ef-a259-f7d033de362e.png)
http://a843ab4143e824286a59a6c519c8997c-911445182.us-east-2.elb.amazonaws.com:8080/shiftmanagement/allshifts/ ![image](https://user-images.githubusercontent.com/111366682/222902261-de849beb-c5cc-48ee-b1b8-81edae897534.png)

## Autoscale the Service

I deployed a metric server with the following command
```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
Verify that the metrics-server deployment is running with the following command
```
kubectl get deployment metrics-server -n kube-system
```
The output is below:
```
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
metrics-server   1/1     1            0           50s
```
Set the Autoscaling for the applications by running the following command
```
kubectl autoscale deployment shiftswebapp --cpu-percent=70 --min=1 --max=3 -n shiftmanagement
```
Check the HPA status
```
kubectl get hpa shiftswebapp -n shiftmanagement
```
Set the HPA for users webapp
'''
kubectl autoscale deployment userswebapp --cpu-percent=70 --min=1 --max=3 -n usermanagement
```
Check the HPA status
```
kubectl get hpa -n usermanagement
```
The output is:
```
NAME          REFERENCE                TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
userswebapp   Deployment/userswebapp   1%/70%   1         3        0          14s
```
The auto scaler configuration is based on CPU usage. If the CPU usage is more than 70%, it will start a new instance.
To test the autoscaling where you can generate load on the service, run
```
kubectl run -i  --tty load-generator --rm --image=busybox  --restart=Never  -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://a843ab4143e824286a59a6c519c8997c-911445182.us-east-2.elb.amazonaws.com:8080/shiftmanagement/allshifts/; done"
```
After running the load, the CPU usage will increase
```
[cloudshell-user@ip-10-4-70-197 ~]$ kubectl get hpa shiftswebapp   -n shiftmanagement
NAME           REFERENCE                 TARGETS    MINPODS   MAXPODS   REPLICAS   AGE
shiftswebapp   Deployment/shiftswebapp   119%/17%   1         3        1          58m
```
And application will automatically start spinning new pods
```
[cloudshell-user@ip-10-4-70-197 ~]$ kubectl get pods -A
NAMESPACE         NAME                              READY   STATUS    RESTARTS   AGE
kube-system       aws-node-85c7r                    1/1     Running   0          5h32m
kube-system       aws-node-dl79b                    1/1     Running   0          5h32m
kube-system       coredns-744dc4f967-4vxwn          1/1     Running   0          5h45m
kube-system       coredns-744dc4f967-qsqj6          1/1     Running   0          5h45m
kube-system       kube-proxy-4qtmg                  1/1     Running   0          5h32m
kube-system       kube-proxy-plcdp                  1/1     Running   0          5h32m
kube-system       metrics-server-679799879f-74pdv   1/1     Running   0          12m
shiftmanagement   shiftswebapp-79d9f468fb-5n82b     1/1     Running   0          25s
shiftmanagement   shiftswebapp-79d9f468fb-6cbwl     1/1     Running   0          25s
shiftmanagement   shiftswebapp-79d9f468fb-6vx72     1/1     Running   0          10s
usermanagement    userswebapp-68547fc79b-9mmxf      1/1     Running   0          4h52m
```
## Rolling Deployment and RollBack
Kubernetes by default supports Rolling deployment. It means when you deploy a new version of your service by updating the deployment file, the old version will continue running and serving traffic until the old version is full up and running. 
You can use roll back commands to roll back your deployment to the previous versions.
You can find more details on rolling deployments and rollback in the link below
(https://medium.com/the-programmer/working-with-deployments-roll-back-and-rolling-updates-bb4299488e34)

To see a deployment, run the command below:
```
kubectl describe deploy shiftswebapp -n shiftmanagement
```
If you change something in a deployment, you can see the rollout status with below command
```
[cloudshell-user@ip-10-4-70-197 shiftmanagement]$ kubectl rollout status deployment/shiftswebapp -n shiftmanagement
deployment "shiftswebapp" successfully rolled out.
```
To see the previous versions of your deployment. Run the command
```
[cloudshell-user@ip-10-4-70-197 shiftmanagement]$ kubectl rollout history deployment/shiftswebapp -n shiftmanagement
deployment.apps/shiftswebapp 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
3         <none>
4         <none>
5         <none>
6         <none>
```
To go back to a previous version
```
[cloudshell-user@ip-10-4-70-197 shiftmanagement]$ kubectl rollout undo deployment/shiftswebapp --to-revision=5 -n shiftmanagement
deployment.apps/shiftswebapp rolled back
```

## Developer Access in the Kubernetes Cluster
To give developer less cluster access you need to create a new AWS IAM policy, which will give developer limited access. The next step is to map this policy with Kubernetes roll bindings and the last step is to update the AWS config maps with new role details.

The first 12 minutes of this video explains how you can setup a new Developer account.
(https://www.youtube.com/watch?v=aIpHYYcR7oU)

### Step 1:Create a Developer Cluster Role and Cluster Bindings in Kubernetes

The developer role has only access to create/edit deployments in all namespaces. It cannot change cluster level properties. The yaml file is stored in DeveloperClusterRole.yaml file
  
Create this role in Kubernetes by running the command
```
kubectl create -f DeveloperClusterRole.yaml
```
The output is
```
clusterrole.rbac.authorization.k8s.io/developers-k8s-cluster-role created
```
Create a new Role binding in Kubernetes. The yaml file is stored in DeveloperClusterRole.yaml file.
  
```
  kubectl create -f DeveloperRoleBinding.yaml
```
  
The output is
  
```
clusterrolebinding.rbac.authorization.k8s.io/developers-k8s-role-binding created
```

  ### Step 2 Create an IAM Policy and Group in AWS
  Create policy
  
<img width="452" alt="image" src="https://user-images.githubusercontent.com/111366682/222906580-38bc0b4f-33a2-4e2b-b881-b809d0d44e88.png">







