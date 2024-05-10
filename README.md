# AWS EKS - Kubernetes Web App/Server
#### Desc.....
#### Desc......

## Architectural Diagram


### NOTE: 

## Step 1: Install kubectl and eksctl (Optional)
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
#### Replace `my-eks-cluster` and `us-east-1` with your cluster name and AWS region.

## Step 3: Create and Deploy ServiceAccount.yml file (Optional)
#### Create and deploy Service Account only if using PRIVATE ECR deployment. Create the IAM role first then reference it.

#### `nano ServiceAccount` > Copy and Paste configuration > Save. 

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-serviceaccount
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::031141527841:role/EKSPullFromEC
```

#### Replace role ARN with your IAM role ARN.

#### Deploy file using command: `kubectl apply -f ServiceAccount.yml`

## Step 4: Create and Deploy Deployment.yml file
#### Create and Deploy deployment configurtion. Use `deployment.yml` to deploy a Web App from a public Docker image repository OR use `NginxDeployment.yml` to deploy a NGINX Web Server OR use `PrivateDeployment.yml` to deploy a Web App from a Private ECR repository. 

#### Create file e.g. `nano deployment.yml` then deploy it using e.g. `kubectl apply -f ServiceAccount.yml`

### Deployment.yml

```
#Public Web App Docker Image

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

### PrivateDeployment.yml

```
#Private AWS ECR Web App Image

apiVersion: apps/v1
kind: Deployment
metadata:
  name: cicdpipeline-server-deployment
spec:
  selector:
    matchLabels:
      app: cicdpipeline-server
  replicas: 3
  template:
    metadata:
      labels:
        app: cicdpipeline-server
    spec:
      serviceAccountName: my-serviceaccount
      containers:
      - name: cicdpipeline-server
        image: 031141527841.dkr.ecr.us-east-1.amazonaws.com/container-web-app:latest
        ports:
          - containerPort: 8080
        command: ["gunicorn"]
        args: ["-w", "4", "-b", ":8080", "app:app"]
```

#### 

## Step 5: Create and Deploy `Service.yml` file
#### Create and Deploy Service.yml file if using Web App. If deploying NGINX Web Server, use command below.

#### Create file e.g. `nano Service.yml` then deploy it using e.g. `kubectl apply -f Service.yml`

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
#### Access the Web App or Web Server via the load balancer IP address. You should now be able to access the Calculator Flask Web App or the NGINX Web Server!

#### Get external IP address of Load Balancer: e.g. `kubectl get services my-service`

## Step 7: Deleting Cluster
#### Delete service and cluster.

#### Delete Service: <br> 
`kubectl delete svc my-service`

#### Delete Cluster: <br>
`eksctl delete cluster --name my-eks-cluster`

## Notes
* 
* 
* 

## Reference 
* 

* 

* 

