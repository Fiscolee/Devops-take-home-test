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

http://a399f5153a514471bafcd4de28b6ff50-1163751732.us-east-2.elb.amazonaws.com:8080/usermanagement/allusers
http://a843ab4143e824286a59a6c519c8997c-911445182.us-east-2.elb.amazonaws.com:8080/shiftmanagement/allshifts/ 

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
```
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

  ### Step 2: Create an IAM Policy and Group in AWS
  Create policy
  
<img width="452" alt="image" src="https://user-images.githubusercontent.com/111366682/222906580-38bc0b4f-33a2-4e2b-b881-b809d0d44e88.png">

I named the policy "AmazonEKSDeveloperPolicy"

<img width="452" alt="image" src="https://user-images.githubusercontent.com/111366682/222907049-d0cb7c2e-9b3c-4d4d-ab03-1bfec5b085ab.png">

After policy creation, its best practice to associated it with a group. So, lets create a new group and link this policy with the group. Please note, the group name should be developer-aws-group because this what we mentioned earlier in Kubernetes Role binding.

<img width="452" alt="image" src="https://user-images.githubusercontent.com/111366682/222907073-14d28765-7739-4994-bb7f-29184198cdb7.png">


Attach policy to the group

<img width="452" alt="image" src="https://user-images.githubusercontent.com/111366682/222907085-4cf66c48-c9ed-4c92-9901-5ed4c0d8c246.png">

<img width="452" alt="image" src="https://user-images.githubusercontent.com/111366682/222907091-24100dc1-ff0a-4c9e-9c78-d435faf16d82.png">

### STEP 3: Create a Developer User ID

<img width="452" alt="image" src="https://user-images.githubusercontent.com/111366682/222907123-dd85bf10-dfe4-4977-9185-e8461abe660f.png">

<img width="452" alt="image" src="https://user-images.githubusercontent.com/111366682/222907127-b63da51f-f9c3-4fe7-87e5-52a4a36c03d7.png">

<img width="452" alt="image" src="https://user-images.githubusercontent.com/111366682/222907150-6a09db1c-7b5c-460d-bab4-67b47caf5e84.png">

Create Access Key for the newly created developer. We need them when we setup this developer profile

<img width="452" alt="image" src="https://user-images.githubusercontent.com/111366682/222907172-56b3cb08-d8f5-465b-9584-8532ec367b07.png">

<img width="452" alt="image" src="https://user-images.githubusercontent.com/111366682/222907179-ff0f9aa7-db47-4f2a-9152-537636bcd852.png">


### Step 4: Setup this developer by creating a profile

Open command prompt or cloud shell, wherever you run asw commands

<img width="452" alt="image" src="https://user-images.githubusercontent.com/111366682/222907264-0f8d58f3-ded7-4956-a430-434e3968f0c3.png">

### Step 5: Modify K8s to bind this newly created user to the K8s group
For this you need to use the admin user who has access to modify the config map

<img width="452" alt="image" src="https://user-images.githubusercontent.com/111366682/222907375-125db208-eaba-4218-a446-e8b0d8482b2f.png">

<img width="452" alt="image" src="https://user-images.githubusercontent.com/111366682/222907422-8948346f-6717-4b31-9996-9f9f9bba8e16.png">

Save your changes

### Step 6: Verify Developer Access

Setup Developer profile
```
aws eks --region us-east-2 update-kubeconfig --name assignment-cluster --profile aws-eks-developer
```

<img width="452" alt="image" src="https://user-images.githubusercontent.com/111366682/222907518-7eaff7ba-0b90-4651-846e-6c8a496464e0.png">

Run any k8s command and you can see developer can run basic commands

<img width="452" alt="image" src="https://user-images.githubusercontent.com/111366682/222907544-301b3971-15c5-4a94-a481-a40db59984da.png">

Try to access any cluster resource. You will see developer in not allowed to do so
``
kubectl get nodes
```
The output for me 

<img width="452" alt="image" src="https://user-images.githubusercontent.com/111366682/222907590-edf34445-7886-49ea-acdf-5fa7e7fdf3b1.png">

## Deploy in multiple environments, staging and production 
There could be multiple approached used to deploy applications in multiple environments. The best approach would be to use helm charts.
(https://www.linkedin.com/pulse/manage-multiple-kubernetes-environments-helm-rohan-shetty/?trk=pulse-article_more-articles_related-content-card)

In this case you create different helm “values.yaml” files for different environments. The values.yaml file is used to store configurable parameters which differ in different environments. For example, staging and production can have their own databases with their own IP addresses and credentials. These values are stored in values-dev.yaml and values-qa.yaml files. 

<img width="194" alt="image" src="https://user-images.githubusercontent.com/111366682/222907966-70b71b4b-a53f-46e8-8908-c06efdfc9872.png">

The next step is to configure a CI/CD pipeline using Jenkins or any other CI/CD tool which use these different configuration files to deploy applications in different environments.

## Autoscale based on network metrics
By default Kubernetes provides built-in support for autoscaling deployments based on resource utilization. As we discussed earlier, you can auto scale your deployments by CPU or Memory with just a few lines of YAML. CPU and memory are two of the most common metrics to use for autoscaling. 
In order to auto scale deployments based on custom metrics, we have to provide Kubernetes with the ability to fetch those custom metrics from within the cluster.
This can be achieved using custom metrics collectors such as Prometheus. We need to install Prometheus cluster on the server to collect additional metrics. Once the Prometheus starts collecting additional metrics, we can setup a horizontal auto scaler on these custom metrics.


