</details>

******

<details>
<summary>Exercise 1: Create Terraform project to spin up EKS cluster </summary>
 <br />
 
##### This project provisions an EKS cluster with the following configuration

- **S3 bucket** as a storage for Terraform state
- K8s cluster with **3 nodes** and **1 fargate profile** for "my-app" namespace
- **Mysql** chart with 3 replicas
- K8s version **1.21**
- AWS region for VPC, EKS and S3 bucket: **"eu-west-3**" 

:warning: Make sure to change the region for your cluster in all relevant places!

:information_source: Check **README.md** file for the exact versions used in the projects for: 
- _Terraform_ 
- _Terraform modules_
- _Terraform providers_

##### To execute the project
- set variables values in the **"dev.tfvars"** file
- set **"bucket name"** and **"bucket region"** values in the terraform configuration in the **"vpc.tf"** file
- `terraform init` - installs all the providers and modules used in the project
- `terraform apply` - executes the Terraform script

##### To access the cluster with kubectl, once it's configured 
- `aws eks update-kubeconfig --name {cluster-name} --region {your-region}`

_ex: `aws eks update-kubeconfig --name my-cluster --region eu-west-3`_

:information_source: This will configure the kubeconfig file in the ~/.kube/ folder

##### To verify the cluster access
- `kubectl get nodes`
- `eksctl get fargateprofile --cluster my-cluster`

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


