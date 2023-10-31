## Multi-Cloud Implementation With Cloud Anthos
![ProjectArch!]()

### 1) Create A GKE Cluster on GCP
#### A) Sign into your GCP Account
![ProjectArch!]()

#### B) Enable The GKE Service API and Cloud Anthos APIs
1. Enable GKE Service API
![GKEService API1!](https://github.com/awanmbandi/realworld-microservice-projects/blob/zdocs/images/Screen%20Shot%202023-10-30%20at%204.11.29%20PM.png)
![GKEService API2!](https://github.com/awanmbandi/realworld-microservice-projects/blob/zdocs/images/Screen%20Shot%202023-10-30%20at%204.12.59%20PM.png)

2. Enable Anthos Service APIs
![AnthosService API!](https://github.com/awanmbandi/realworld-microservice-projects/blob/zdocs/images/Screen%20Shot%202023-10-30%20at%204.07.45%20PM.png)

3. Navigate to The Anthos Dashboard
![AnthosService Dashboard!](https://github.com/awanmbandi/realworld-microservice-projects/blob/zdocs/images/Screen%20Shot%202023-10-30%20at%204.06.09%20PM.png)

#### C) Create The GKE Cluster
- Navigate to the `GKE Service`
- Click on `Create`
![GKEDashboard!](https://github.com/awanmbandi/realworld-microservice-projects/blob/zdocs/images/Screen%20Shot%202023-10-31%20at%201.09.45%20PM.png)
![GKEDashboard!](https://github.com/awanmbandi/realworld-microservice-projects/blob/zdocs/images/Screen%20Shot%202023-10-31%20at%201.10.47%20PM.png)
* Click on `SWITCH TO STANDARD CLUSTER`
  * Cluster name: `gke-anthos-managed-cluster`
  * Location type: Select `Zonal`
    * Zone: Select `us-central1-c`
    * Release channel: Select `Regular channel (default)`
    * Control plane version: Select the latest `default`
  
  * Click on `default-pool` on your left
    * Name: `default-pool`
    * Size: `2`
    * Surge upgrade: `1`
    * Surge upgrade: `0`

  * Click on `Nodes`
    * Image type: `Container-Optimize OS With Containerd (cos_containerd) (default)`
    * Machine Configurations:
      * Series: `E2`
      * Machine type: `e2-standard-4`  (We need at least between `8` Memory or more. `16` is perfect)
      * Boot disk type: `Balanced Persistent Disk`
      * Boot disk size: `100`
      * Boot disk encryption: Select `Google-managed encryption key`
      * Click `CREATE` to create the Cluster


### 2) Create An EKS Cluster on AWS
##### A) Create The Cluster
- Navigate to `AWS EKS`
- Click on `Add Cluster`
![EKSDashboard!](https://github.com/awanmbandi/realworld-microservice-projects/blob/zdocs/images/Screen%20Shot%202023-10-31%20at%201.43.36%20PM.png)
  - Name: `eks-anthos-managed-cluster`
  - Kubernetes version: `1.23`  *MAKE SURE TO SELECT BUT THIS VERSION*
  - Cluster service role: Create a New One
    - Attach Policy: `AdministratorAccess`
  - Click `Next`
  - VPC: `Default VPC`
  - Subnets: Select at least `2 Subnets`
  - Security groups: Select `Default`  *any (But you''ll have to `Edit Inbound Rules` and Open port `80`)*
  - Choose cluster IP address family: `IPv4`
  - Cluster endpoint access: Select `Public`
  - Control plane logging: `Enable ALL OF THEM` *(API server, Audit, Authenticator, Controller manager, Scheduler)*
  - Click `Next`
  - Select add-ons: Enable kube-proxy, CodeDNS, Amazon VPC CNI
    - Click on `Next`
![EKSDashboard!](https://github.com/awanmbandi/realworld-microservice-projects/blob/zdocs/images/Screen%20Shot%202023-10-31%20at%201.43.36%20PM.png)
  - CoreDNS Version: Default version
  - Amazon VPC CNI Version: Default version
  - kube-proxy version: Default version
  - Click `Next`
  - Click on `CREATE` *to create the cluster*
  - **NOTE:** *If you run into an error about resource availability within a specific zone, click on previous and delete the Zone Subnet Selection and then Create*
![EKSDashboard!](https://github.com/awanmbandi/realworld-microservice-projects/blob/zdocs/images/Screen%20Shot%202023-10-31%20at%202.03.16%20PM.png)

##### B) Create EKS Cluster Node Group
- Once the Cluster Creation is Complete and the `Status = Active`
- Click on the Cluster name `eks-anthos-managed-cluster` (Confirm it's in an Active State)
- Click on `Compute`
- Click on `Add node group`
![EKSDashboard!](https://github.com/awanmbandi/realworld-microservice-projects/blob/zdocs/images/Screen%20Shot%202023-10-31%20at%202.11.27%20PM.png)
  - Name: `eks-default-ng`
  - Node IAM role: Select the `AWSServiceRoleForAmazonEKSNodegroup` *role or any existing role, or Create a NEW ONE*
  - Click `Next`
  - AMI type: `Amazon Linux 2`
  - Capacity: `On-Demand`
  - Instance type: `t3.xlarge`
  - Disk size: `20`
  - Desired size: `2`
  - Minimum size: `2`
  - Maximum size: `2`
  - Maximum unavailable (Number): `1`
    - Click on `Next`
  - Subnets: Select atleast Two subnets (You can as well select all)
  - Configure remote access to nodes: `DISABLE`
    - Click on `Next`
  - Click on `CREATE`
![EKSDashboard!](https://github.com/awanmbandi/realworld-microservice-projects/blob/zdocs/images/Screen%20Shot%202023-10-31%20at%202.30.09%20PM.png)

### 2) Create An AKS Cluster on Azure



### 3) Create a Management Instance on AWS (For EKS Cluster Setup)
##### Create EC2 Instance 
- Name: `EKS-Setup-Env`
- AMI: `CentOs 7`
- Instance type: `t2.micro`
- Keypair: `Select or create one`
- `Launch` Instance

##### Generate API Access Keys
- Navigate to `IAM`
- Identify the `User` that you used to `create` the `EKS Cluster`
  - Could be a *`Normal User` or `Root User`*
![IAMGenerateAccessKeys!](https://github.com/awanmbandi/realworld-microservice-projects/blob/zdocs/images/Screen%20Shot%202023-10-31%20at%202.39.21%20PM.png)
- Generate a new set of `Access Keys` from the user
  - **NOTE:** *Make sure it's the user you used to create the EKS Cluster*
  - **Root User:** *For the `Root User`, Follow the bellow steps to Generate the Access Keys*
    - Click on your Account Name `Top Right`
    - Click on `Security Credentials`
    - Click on `Create Access Keys`
- `Save` them on a NodePad or Something

##### Install AWS CLI on The `EKS-Setup-Env` Instance 
- Navigate to `EC2`
- Login/SSH into the `EKS-Setup-Env` Instance 
- Install the following Utilities
  * Install `AWS CLI`
  * Install `eksctl`
  * Install `kubectl`
  * Install `Google Cloud SDK (gcloud)etc`

```bash
sudo yum update -y
sudo yum install -y python3
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo yum install unzip -y
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

##### Configure Your AWS CLI User Profile
- **NOTE:** Make sure you use the **USER Credentials** from the user you used to create the cluster **(Root or Normal user)**
- **NOTE:** If Not you wont be able to reach the cluster
```bash
aws configure
```

##### Confirm User Credential Config (Make sure the USER matches the user used to create the cluster)
**NOTE:*** *Make sure the User Name you get reflects the User that Provisioned the EKS Cluster*
```bash
aws sts get-caller-identity
```

### 3) INSTALL AWS EKS `eksctl` On The `EKS-Setup-Env` VM Instance. The Commands Support `arm64`, `armv6` or `armv7`
##### The Installation Link
- https://eksctl.io/installation/

##### Installation Commands
```bash
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH

curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"

curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check

tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz

sudo mv /tmp/eksctl /usr/local/bin

eksctl version
```

### 4) Installing Kubernetes `kubectl` Command Line in the `EKS-Setup-Env` Instance
##### Installing Kubernetes kubectl command line Link
- https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html

##### `kubectl` Installation Commands
```bash
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.28.2/2023-10-17/bin/linux/amd64/kubectl

curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.28.2/2023-10-17/bin/linux/amd64/kubectl.sha256

sha256sum -c kubectl.sha256

openssl sha1 -sha256 kubectl

chmod +x ./kubectl

mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$HOME/bin:$PATH

echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc

kubectl version --client
```

### 4) Installation Google Cloud SDK for Linux
- https://cloud.google.com/sdk/docs/install#linux

##### Cloud SDK Installation and Configuration Steps (For Cloud Anthos)
```bash
curl -O https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/PACKAGE_NAME

tar -xf PACKAGE_NAME

./google-cloud-sdk/install.sh

sudo cp -rf google-cloud-sdk /usr/lib/

export PATH="/usr/lib/google-cloud-sdk/bin:$PATH"

gcloud version  # or
gcloud --version
```

##### Initialize Cloud SDK and Login
```bash
gcloud init
```

##### Configure Your User and Application Auths and Set Project ID
```bash
gcloud auth login
gcloud config set project PROJECT_ID
gcloud auth application-default login
```

##### Enable The Following APIs on GCP For Anthos
```bash
gcloud services enable gkemulticloud.googleapis.com
gcloud services enable gkeconnect.googleapis.com
gcloud services enable connectgateway.googleapis.com
gcloud services enable cloudresourcemanager.googleapis.com
gcloud services enable anthos.googleapis.com
gcloud services enable logging.googleapis.com
gcloud services enable monitoring.googleapis.com
gcloud services enable opsconfigmonitoring.googleapis.com
```

### 5) Access Cluster: Update the `Kube Config` Configuration 
This file is used to managed K8S Cluster User Authorization Definition
```bash
aws eks --region CLUSTER_REGION update-kubeconfig --name CLUSTER_NAME
```

##### Get the Nodes and Pods to confirm Cluster Access
```bash
kubectl get nodes

eksctl get cluster --name eks-anthos-managed-cluster --region us-west-2

kubectl get pods --all-namespaces
```

### 6) Configure/Complete EKS Cluster Membership Prequisites
##### Setup The `OIDC` URL and `KUBE_CONFIG_CONTEXT` Context Variables
```bash
# Create OIDC URL Variable
OIDC_URL=$(aws eks describe-cluster --name <Cluster_Name> --region <Cluster_Region> --query "cluster.identity.oidc.issuer" --output text)
echo $OIDC_URL

# Create KubeConfig Context Variable
KUBE_CONFIG_CONTEXT=$(kubectl config current-context)
echo $KUBE_CONFIG_CONTEXT
kubectl get ns     (After Running the Mmbership Register Command, A new namespace will get created for the Anthos Connect Agent)
```

### 7) Register The EKS Cluster As A Member To The Anthos Hub
##### Confirm Your Auth and Project Configurations Are All Set
```bash
gcloud auth list
gcloud config configurations list
```

##### Register The EKS Cluster To Anthos Hub
```bash
gcloud container hub memberships register eks-anthos-managed-cluster \
--context=$KUBE_CONFIG_CONTEXT \
--public-issuer-url=$OIDC_URL \
--kubeconfig="~/.kube/config" \
--project=PROVIDE_PROJECT_ID \
--enable-workload-identity
```

##### Confirm That The Anthos Connect Agent Was Deployed Successfully And It's Running
```bash
kubectl get ns
kubectl get pods -n gke-connect
```

### 8) Login To The Attached EKS Cluster (For Anthos To Manage)
#### 8.1) Configure Authentication/Authorization From AWS EKS To Anthos with S.A Tokens
```bash
kubectl create serviceaccount -n kube-system anthos-admin-sa
kubectl get serviceaccount anthos-admin-sa -n kube-system
kubectl create clusterrolebinding anthos-admin-sa-binding --clusterrole cluster-admin --serviceaccount kube-system:anthos-admin-sa

# Here We're setting a Variable for The Secret Name of the above Service Account
SECRET_NAME=$(kubectl get serviceaccount -n kube-system anthos-admin-sa -o jsonpath='{$.secrets[0].name}')
echo $SECRET_NAME

# Here's We're Setting a Variable for The Secret Token (The Encoded Format, We have to Decode it)
BASE64_ENCODED_TOKEN=$(kubectl get secret -n kube-system $SECRET_NAME -o jsonpath='{$.data.token}')
echo $BASE64_ENCODED_TOKEN
```

#### 8.2) Decode above token and use it
- Go to: https://www.base64decode.org/
    - Paste the Encoded Version 
    - Click on `DECODE`
    - Navigate to `Anthos LOGIN Page`

#### 8.3) We Have To Now Authorize The EKS Cluster Access From Cloud Anthos (To Proide Access To Anthos)
- Navigate tho the `Anthos Dashboard` or `Login Page`
    - Click on `Clusters`
    - Click on your cluster name `eks-anthos-managed-cluster`
        - **NOTE::** You'll see, `Authenticate into the cluster to see more details`
        - **NOTE::** You can as well Go to GKE --> Click Cluster Name
        - Click on `LOGIN` 
        
        - **NOTE::** *(You can use any of these options GCP Identity, K8S Service Account Token, IDP etc)*
        - Select `Token`
        - PASTE the `DECODED Version` of the `TOKEN` in the `BOX`
        - Click on `LOGIN` 

        - Congratulations, `you are now LOGGED IN`
        - Navigate Back to `Anthos UI/Dashboard`

### 



















