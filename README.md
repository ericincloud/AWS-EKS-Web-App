# AWS EKS - Kubernetes Web App/Server
#### Desc.....
#### Desc......

## Architectural Diagram


### NOTE: 

## Step 1: Install kubectl and eksctl (Optional)
#### Install `kubectl` and `eksctl` to enable communication and ability to manage EKS clusters. In addition, ensure AWS CLI is already installed or configured.

#### Install [kubectl]

```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

#### Install the [eksctl] command-line tool.


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
#### 

## Step 3: Create and Deploy ServiceAccount.yml file (Optional)
#### 

## Step 4: Create and Deploy Deployment.yml file
#### 

## Step 5: Create and Deploy Service.yml file
#### 

## Step 6: Verify Functionality
#### 

## Step 7: Deleting Cluster
#### 

## Step 8: 
#### 

## Notes
* 
* 
* 

## Reference 
* 

* 

* 

