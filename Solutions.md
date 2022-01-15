</details>

******

<details>
<summary>Exercise 1: Create Terraform project to spin up EKS cluster </summary>
 <br />

**Steps**
```sh
# create cluster with 3 EC2 instances and store access configuration to cluster in kubeconfig.my-cluster.yaml file 
eksctl create cluster --name=my-cluster --nodes=3 --kubeconfig=./kubeconfig.my-cluster.yaml

# create fargate profile in the cluster. It will apply for all K8s components in my-app namespace
eksctl create fargateprofile \
    --cluster my-cluster \
    --name my-fargate-profile \
    --namespace my-app

# point kubectl to your cluster - use absolute path to kubeconfigfile
export KUBECONFIG={absolute-path}/kubeconfig.my-cluster.yaml

# validate cluster is accessible and nodes and fargate profile created
kubectl get node
eksctl get fargateprofile --cluster my-cluster

```

</details>

******

<details>
<summary>Exercise 2: Deploy Mysql and phpmyadmin </summary>
 <br />

**General notes**
- All the k8s manifest files for the exercise are in "k8s-deployment" folder, so:
```sh
# clone this repository locally
git clone git@gitlab.com:devops-bootcamp3/bootcamp-java-mysql.git

# check out the solutions branch
git checkout feature/solutions

# change to k8s-deployment folder
cd k8s-deployment

```

- Mysql Chart link: 
https://github.com/bitnami/charts/tree/master/bitnami/mysql 

```sh
# install Mysql chart 
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install my-release bitnami/mysql -f mysql-chart-values-eks.yaml --version 8.8.6
# Note that chart version version 8.8.8+ has a bug setting the db user password incorrectly, which affects EKS installation: https://giters.com/bitnami/charts/issues/8557, that's why we are installing an older version. 


# deploy phpmyadmin with its configuration for Mysql DB access
kubectl apply -f db-config.yaml
kubectl apply -f db-secret.yaml
kubectl apply -f phpmyadmin.yaml

# access phpmyadmin and login to mysql db
kubectl port forward svc/phpmyadmin-service 8081:8081

# access in browser on
localhost:8081

# login with one of these 2 credentials
"my-user" : "my-pass"
"root" : "secret-root-pass"

```

</details>

******

<details>
<summary>Exercise 3: Deploy your Java Application with 3 replicas </summary>
 <br />

**Steps**
```sh

# Create namespace my-app to deploy our java application, because we are deploying java-app with fargate profile. And fargate profile we create applies for my-app namespace. 
kubectl create namespace my-app

# We now have to create all configuration and secrets for our java app in the my-app namespace

# Create my-registry-key secret to pull image 
DOCKER_REGISTRY_SERVER=docker.io
DOCKER_USER=your dockerID, same as for `docker login`
DOCKER_EMAIL=your dockerhub email, same as for `docker login`
DOCKER_PASSWORD=your dockerhub pwd, same as for `docker login`

kubectl create secret -n my-app docker-registry my-registry-key \
--docker-server=$DOCKER_REGISTRY_SERVER \
--docker-username=$DOCKER_USER \
--docker-password=$DOCKER_PASSWORD \
--docker-email=$DOCKER_EMAIL


# Again from k8s-deployment folder, execute following commands. By adding the my-app namespace, these components will be created with Fargate profile
kubectl apply -f db-secret.yaml -n my-app
kubectl apply -f db-config.yaml -n my-app
kubectl apply -f java-app.yaml -n my-app

```

</details>

******

<details>
<summary>Exercise 4 & 5: Automate deployment & Use ECR as Docker repository </summary>
 <br />

**Current cluster setup**

At this point, you already have an EKS cluster, where: 
- Mysql chart is deployed and phpmyadmin is running too
- my-app namespace was created
- db-config and db-secret were created in the my-app namspace for the java-app
- my-registry-key secret was created to fetch image from docker-hub
- your java app is also running 

**Steps to automate deployment for existing setup**
```sh
# Create an ECR registry for your java-app image

# Locally, on your computer: Create a docker registry secret for ECR
DOCKER_REGISTRY_SERVER=your ECR registry server - "your-aws-id.dkr.ecr.your-ecr-region.amazonaws.com"
DOCKER_USER=your dockerID, same as for `docker login` - "AWS"
DOCKER_PASSWORD=your dockerhub pwd, same as for `docker login` - get using: "aws ecr get-login-password --region {ecr-region}"

kubectl create secret -n my-app docker-registry my-ecr-registry-key \
--docker-server=$DOCKER_REGISTRY_SERVER \
--docker-username=$DOCKER_USER \
--docker-password=$DOCKER_PASSWORD


# SSH into server where Jenkins container is running
ssh -i {private-key-path} {user}@{public-ip}

# Enter Jenkins container
sudo docker exec -it {jenkins-container-id} -u 0 bash

# Install aws-cli inside Jenkins container
- Link: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
./aws/install

# Install kubectl inside Jenkins container
- Link: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

apt-get update
apt-get install -y apt-transport-https ca-certificates curl
curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | tee /etc/apt/sources.list.d/kubernetes.list
apt-get update
apt-get install -y kubectl

# Install envsubst tool
- Link: https://command-not-found.com/envsubst

apt-get update
apt-get install -y gettext-base

# create 2 "secret-text" credentials for AWS access in Jenkins: 
- "jenkins_aws_access_key_id" for AWS_ACCESS_KEY_ID 
- "jenkins_aws_secret_access_key" for AWS_SECRET_ACCESS_KEY    

# Create 4 "secret-text" credentials for db-secret.yaml:
- id: "db_user", secret: "my-user"
- id: "db_pass", secret: "my-pass"
- id: "db_name", secret: "my-app-db"
- id: "db_root_pass", secret: "secret-root-pass"

# Set the correct values in Jenkins for following environment variables: 
- ECR_REPO_URL
- CLUSTER_REGION

# Create Jenkins pipeline using the Jenkinsfile in this branch, in the root folder
Make sure the paths to the k8s manifest files in the "deploy" stage of the Jenkinsfile are all correct!!

```

</details>

******

<details>
<summary>Exercise 6: Configure Autoscaling </summary>
 <br />

You learn how to scale the cluster up and down in the **_Kubernetes on AWS_** module, video **_3 - Configure Autoscaling in EKS cluster_**


