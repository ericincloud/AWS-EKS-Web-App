# AWS EKS - Kubernetes Web App/Server
#### AWS EKS - Kubernetes Web App/Server allows for the deployment choice either of a calculator web application or NGINX web server. The calculator web app built using flask is deployed using a private Docker image. On the other hand, the NGINX web server uses a public NGINX iamge. Both deployments are hosted on a AWS Kubernetes cluster that users access through a load balancer. AWS EKS - Kubernetes Web App/Server simplifies container management whiling allowing users to access containerized apps, servers, and services while maintaining high availability. 

## Step 1: Install kubectl and eksctl 
#### Install `kubectl` and `eksctl` to enable communication and ability to manage EKS clusters. In addition, ensure AWS CLI is already installed or configured.

### Install [kubectl]

```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

### Install the [eksctl] command-line tool.


```bash
# for ARM systems, set ARCH to: `arm64`, `armv6` or `armv7`
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH

curl -sLO "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"

# (Optional) Verify checksum
curl -sL "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check

tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz

sudo mv /tmp/eksctl /usr/local/bin
```

#### Save the above script as `eksctl.sh` then run `chmod +x eksctl.sh` and `sudo sh eksctl.sh`.

#### E.g. `nano eksctl.sh`


## Step 2: Create Cluster
#### Create an EKS Cluster, use command:

```
eksctl create cluster --name my-eks-cluster --region us-east-1 --nodegroup-name my-nodegroup --node-type t2.small --nodes 3 --nodes-min 1 --nodes-max 5 --managed
```
#### Replace `my-eks-cluster` and `us-east-1` with your cluster name and AWS region. Resoures will be deployed via CloudFormation. 

![image](https://github.com/ericincloud/AWS-EKS-Web-App/assets/144301872/3bf4469d-a208-4b8e-aaec-f0562061a7d6)
![image](https://github.com/ericincloud/AWS-EKS-Web-App/assets/144301872/e3c8401f-f035-412e-ac30-c862d6909f64)


## Step 3: Create Kubernestes secret (Optional: For private DockerHub image only)
#### Create Kubernetes secret to allow access to Private DocekrHub repository. Enter username and password.

#### 

```
kubectl create secret docker-registry regcred --docker-username=<username> --docker-password=<password>
```

## Step 4: Create and Deploy Deployment.yml file
#### Create and Deploy deployment configurtion. Use `deployment.yml` to deploy a Web App from a private Docker image repository OR use `NginxDeployment.yml` to deploy a NGINX Web Server.
#### Create file e.g. `nano deployment.yml` then deploy it using e.g. `kubectl apply -f deployment.yml`

### deployment.yml

```
#Private DockerHub Web App Image

apiVersion: apps/v1
kind: Deployment
metadata:
  name: cicdpipeline-server-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: cicdpipeline-server
  template:
    metadata:
      labels:
        app: cicdpipeline-server
    spec:
      containers:
      - image: ericincloud/cicdpipeline-server:latest
        name: cicdpipeline-server
        ports:
        - containerPort: 8080
        command: ["gunicorn"]
        args: ["-w", "4", "-b", ":8080", "app:app"]
      imagePullSecrets:
      - name: regcred
```

### NginxDeployment.yml

```
#NGINX Web Server Image

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

#### 

## Step 5: Expose App/Server using Kubernetes service
#### Create and Deploy Service.yml file if using Web App. If deploying NGINX Web Server, use command below. This creates a load balacner that grants access to the app or server. 

#### Create file e.g. `nano service.yml` then deploy it using e.g. `kubectl apply -f service.yml`

### Web App Service

```
#For Web App 

apiVersion: v1
kind: Service
metadata:
  name: cal-web-app
spec:
  ports:
  - name: http
    port: 80
    targetPort: 8080
  selector:
    app: cicdpipeline-server
  type: LoadBalancer
```

### NGINX Web Server Service

`kubectl expose deployment nginx-deployment --type=LoadBalancer --name=my-service`

## Step 6: Verify Functionality
#### Access the Web App or Web Server via the load balancer IP address. Get external IP address of Load Balancer using: <br>

```
kubectl get services my-service
```

![image](https://github.com/ericincloud/AWS-EKS-Web-App/assets/144301872/0e9e59b9-609d-492c-824b-8b9fd2b9a24d)
![image](https://github.com/ericincloud/AWS-EKS-Web-App/assets/144301872/4f9fbaca-c3d1-45b5-b979-474183f68219)



#### You should now be able to access the Calculator Flask Web App or the NGINX Web Server!

## Step 7: Deleting Cluster
#### Delete service and cluster.

#### Delete Service: <br> 
`kubectl delete svc my-service`

#### Delete Cluster: <br>
`eksctl delete cluster --name my-eks-cluster`

## Notes
* Ensure architecture (arm64,amd64) match between the app/server/service and instance to prevent any exec errors.
* Use appropriate compute type for application/server.
* Use IAM role for private image repositories. 
* Ensure required app dependencies.
* Enter `sudo sh eksctl.sh` for related error.

## Reference 

* Deploy file <br>
```
kubectl apply -f <file>
```

* Create Cluster <br>
```
eksctl create cluster --name my-eks-cluster --region us-east-1 --nodegroup-name my-nodegroup --node-type t2.small --nodes 3 --nodes-min 1 --nodes-max 5 --managed
```

* Delete service and cluster <br>
```
kubectl delete svc my-service

eksctl delete cluster --name my-eks-cluster
```

*Troubleshooting

* Get events
`kubectl get events` 

* Lists all pods in the current namespace. <br>
`kubectl get pods`

* hows detailed information about a specific pod. <br>
`kubectl describe pod <pod-name>`

* Prints the logs for a specific pod. <br>
`kubectl logs <pod-name>`

* Lists all services in the current namespace. <br>
`kubectl get svc`

* Lists all nodes in the cluster. <br>
`kubectl get nodes`

* Shows detailed information about a specific node. <br>
`kubectl describe node <node-name> `

* Lists all EKS clusters. <br>
`eksctl get cluster`


