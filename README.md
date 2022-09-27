# Crossplane project.

The following project describes how infrastructure can be created through crossplane, using AWS services.
Before starting with the explanation it is important to clarify the following assumptions.
- There must be a previous kubernetes cluster where Crossplane can be installed. (It can be through terraform, cloudformation or even use minikube)
- You must have previously installed helm, kubectl,eksctl.
- The user who will use crossplane must have the necessary permissions to create infrastructure in AWS.

## Main Diagram.
![](https://github.com/[username]/[reponame]/blob/[branch]/image.jpg?raw=true)
The solution is to create a password, user and endpoint in Secrets Manager, at the same time an EKS cluster and deploy within this cluster a MySql database and a Wordpress site. Both pods used credentials obtained from the secret manager, and the template that we will use is a secretProviderClass that is responsible for mapping the necessary values. The pods will use a volume that is where the passwords will be accessed, for this functionality we use CSI.
It is important to mention that from Crossplane we will create services such as VPC (subnets, internetgateway, routes, security groups), Secrets Manager, EKS (node-groups), IAM (Policies, Role), among others.

## Installation

We will use kustomize and HELM chart files for the installation. The order is as follows.

- **Crossplane**: will be installed and configured in the previously built cluster.
- **VPC**: kustomize will be executed to enable VPC, subnets, internet-gateway and route-table.
- **Policies**: the kustomize will be executed and the policies will be enabled.
- **Security Groups**: the kustomize will be executed and the SG will be enabled.
- **Roles for EKS**: the kustomize will be executed, it will enable the role that the EKS cluster will use.
- **Secret Manager**: the kustomize will be executed, it will enable the secret manager that contains the user, password and endpoint that the microservices will use.
- **EKS Cluster**: the kustomize that will create the eks cluster will be executed and will make use of the previously created AWS services.
- **Roles for SA**: kustomize will be executed, it will enable the role that the service account will use to read the definition of Secret Manager in the app.
- **App** contains a wordpress site along with a mysql database, being a laboratory they do not have persistence in the volumes, however, they fulfill the objective of reading passwords from secrets-manager.

### Crossplane
1. Create crossplane-system namespace
```
kubectl create namespace crossplane-system
```
2. Add and install the Crossplane related helm chart.
```
helm repo add crossplane-stable https://charts.crossplane.io/stable
helm repo update
helm install crossplane --namespace crossplane-system crossplane-stable/crossplane
```
3. Install crossplane cli and install the respective component.
```
curl -sL https://raw.githubusercontent.com/crossplane/crossplane/master/install.sh | sh
sudo mv kubectl-crossplane /usr/local/bin
kubectl crossplane install configuration registry.upbound.io/xp/getting-started-with-aws:v1.9.1
```
4. Create the file related to the AWS account.
```
AWS_PROFILE=default && echo -e "[default]\naws_access_key_id = $(aws configure get aws_access_key_id --profile $AWS_PROFILE)\naws_secret_access_key = $(aws configure get aws_secret_access_key --profile $AWS_PROFILE)" > creds.conf
```
5. Create the secret that contains the AWS credentials.
# Crossplane project.

The following project describes how infrastructure can be created through crossplane, using AWS services.
Before starting with the explanation it is important to clarify the following assumptions.
- There must be a previous kubernetes cluster where Crossplane can be installed. (It can be through terraform, cloudformation or even use minikube)
- You must have previously installed helm, kubectl.
- The user who will use crossplane must have the necessary permissions to create infrastructure in AWS.

## Main Diagram.
![](https://github.com/jcastrou/crossplane/blob/main/main-diagram.jpg?raw=true)
The solution is to create a password, user and endpoint in Secrets Manager, at the same time an EKS cluster and deploy within this cluster a MySql database and a Wordpress site. Both pods used credentials obtained from the secret manager, and the template that we will use is a secretProviderClass that is responsible for mapping the necessary values. The pods will use a volume that is where the passwords will be accessed, for this functionality we use CSI.
It is important to mention that from Crossplane we will create services such as VPC (subnets, internetgateway, routes, security groups), Secrets Manager, EKS (node-groups), IAM (Policies, Role), among others.

## Installation

We will use kustomize and HELM chart files for the installation. The order is as follows.

- **Crossplane**: will be installed and configured in the previously built cluster.
- **VPC**: kustomize will be executed to enable VPC, subnets, internet-gateway and route-table.
- **Policies**: the kustomize will be executed and the policies will be enabled.
- **Security Groups**: the kustomize will be executed and the SG will be enabled.
- **Roles for EKS**: the kustomize will be executed, it will enable the role that the EKS cluster will use.
- **Secret Manager**: the kustomize will be executed, it will enable the secret manager that contains the user, password and endpoint that the microservices will use.
- **EKS Cluster**: the kustomize that will create the eks cluster will be executed and will make use of the previously created AWS services.
- **Roles for SA**: kustomize will be executed, it will enable the role that the service account will use to read the definition of Secret Manager in the app.
- **App** contains a wordpress site along with a mysql database, being a laboratory they do not have persistence in the volumes, however, they fulfill the objective of reading passwords from secrets-manager.

### Crossplane
1. Create crossplane-system namespace
```
kubectl create namespace crossplane-system
```
2. Add and install the Crossplane related helm chart.
```
helm repo add crossplane-stable https://charts.crossplane.io/stable
helm repo update
helm install crossplane --namespace crossplane-system crossplane-stable/crossplane
```
3. Install crossplane cli and install the respective component.
```
curl -sL https://raw.githubusercontent.com/crossplane/crossplane/master/install.sh | sh
sudo mv kubectl-crossplane /usr/local/bin
kubectl crossplane install configuration registry.upbound.io/xp/getting-started-with-aws:v1.9.1
```
4. Create the file related to the AWS account.
```
AWS_PROFILE=default && echo -e "[default]\naws_access_key_id = $(aws configure get aws_access_key_id --profile $AWS_PROFILE)\naws_secret_access_key = $(aws configure get aws_secret_access_key --profile $AWS_PROFILE)" > creds.conf
```
5. Create the secret that contains the AWS credentials.
```
kubectl create secret generic aws-creds -n crossplane-system --from-file=creds=./creds.conf
```
6. Create the AWS provider, to be able to create the resources in AWS.
```
kubectl apply -f provider-cnf.yaml 
```

### AWS resources
Go to the crossplane/templates path and run the following commands.
1. Create VPC and components
```
kubectl apply -k vpc
```
2. Create policies
```
    kubectl apply -k policies   
```
3. Create EKS roel
```
    kubectl apply -k role-eks
```
4. Create security group
```
    kubectl apply -k security-groups
```
5. Create Secret Manager
```
    kubectl apply -k secret-manager
```
6. Create EKS cluster
```
    kubectl apply -k eks-cluster
```

### App

First we must prepare two files, Both are in crossplane/templates/role-sa/, in both files the account number must be placed and in the same way the OIDC number of the previously created cluster. Once you change them you can continue with the following steps.
1. Apply the role of the service account.
```
kubectl apply -k role-sa
```
In the EKS cluster that was created in the previous steps, the following command must be executed to create the application resources (Wordpress and MySQL).
2.  The following components must be installed in the cluster where the application will be used, which will allow us to communicate with the Secrets Manager service.
```
helm repo add secrets-store-csi-driver https://kubernetes-sigs.github.io/secrets-store-csi-driver/charts

helm install csi-secrets-store secrets-store-csi-driver/secrets-store-csi-driver --set syncSecret.enabled=true --namespace kube-system

kubectl apply -f https://raw.githubusercontent.com/aws/secrets-store-csi-driver-provider-aws/main/deployment/aws-provider-installer.yaml

```
3. Create an IAM OIDC provider for the cluster if you don't already have one
```
eksctl utils associate-iam-oidc-provider --region=us-east-1 --cluster=app-cluster --approve 
```
4.  Go to the root of the project and kustomize the project with the following command.
```
kubectl apply -k app
```
5. After a few minutes you can access the wordpress site by getting the loadbalancer with the following command and check the external address.
```
kubectl get svc
```