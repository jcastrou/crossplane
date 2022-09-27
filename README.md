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


