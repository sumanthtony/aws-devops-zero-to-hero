
#STEPS TO COMPLETE THIS PROJECT:

1. Install AWS CLI, kubectl, and eksctl

Note: Give IAM permission to integate AWS services with the server.

Install AWS CLI

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
TO SET PATH: vim .bashrc
export PATH=$PATH:/usr/local/bin/
source .bashrc

Install KUBECTL:

curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kops-linux-amd64 kubectl
mv kubectl /usr/local/bin/kubectl
kubectl version

INSTALL EKSCTL :

curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version

2. Command to install eksctl: eksctl create cluster --name eks-1 --region ap-south-1 --zones=ap-south-1a,ap-south-1b --fargate 

Note: If we give FARGATE by default it will create 2 FARGATE profiles in 2 namespaces (default, kube-system).

| Concept             | What it controls                      |
| ------------------- | ------------------------------------- |
| Fargate Profile     | WHERE pods can run (namespace/labels) |
| Deployment Replicas | HOW MANY pods run                     |

---> Fargate profiles define where pods run, while the number of pods is controlled by deployment replicas. The 2 pods you see are due to high availability, not multiple Fargate profiles.

<img width="808" height="396" alt="image" src="https://github.com/user-attachments/assets/59f83db2-648c-417d-9068-556f5ea6549f" />

3. Once it is created we can go and check in EKS & CloudFormation (stack) will be created and we can see full details.

<img width="936" height="154" alt="image" src="https://github.com/user-attachments/assets/654cc6c1-b37a-494d-8025-de293ba3ddda" />

<img width="933" height="204" alt="image" src="https://github.com/user-attachments/assets/5324b34e-ef1e-4945-85b5-f6bcf222c6b6" />

4. Once EKS cluster is created we can see OPENID connector provider URL is created because of this AWS allows any Identity Providers or IAM to integrate and work on EKS

--->USECASE: To integrate/communicate one service to another service in AWS we have IAM (Users, Roles) but for other resources like pods or any other resource components needs to communicate we can't give access for these, so for this only we have openID connector based on this we can communicate easily with other services in AWS. 

<img width="909" height="302" alt="image" src="https://github.com/user-attachments/assets/def3807c-4608-4582-9166-9c6985ab439c" />

Note: If we need to maintain serverless-we will use fargate if we need to maintain in servers (EC2) we can mention it based on our requirement.

5. We can see fargate profile is created and we can check in server as well.

<img width="849" height="362" alt="image" src="https://github.com/user-attachments/assets/be9aa6c5-452e-4c95-ad68-fb48d0d2e320" />

<img width="618" height="273" alt="image" src="https://github.com/user-attachments/assets/96103df1-8552-4dde-b2c1-31ddf141ce3e" />

<img width="650" height="304" alt="image" src="https://github.com/user-attachments/assets/c6183fc4-3550-413e-b3a4-1a9cfa234c8b" />

---> ABOVE FARGATE NODES ARE: 
Kubernetes virtual nodes
Backed by Fargate compute internally
One node per pod (in many cases)

---> IMPORTANT DIFFERENCE

| Feature                | EC2 Nodes | Fargate Nodes |
| ---------------------- | --------- | ------------- |
| Visible in EKS Console | ✅ Yes     | ❌ No          |
| Part of Node Group     | ✅ Yes     | ❌ No          |
| Visible in kubectl     | ✅ Yes     | ✅ Yes         |
| Managed by             | You       | AWS           |


6. While creating EKS if we get warning like below screenshot follow this:

<img width="903" height="161" alt="image" src="https://github.com/user-attachments/assets/b54df4cb-5d8a-40ef-8ed8-45cac6ca7a2c" />


What AWS is telling you with that button

👉 “Create access entry” =
Add your IAM user/role into Kubernetes RBAC

This is the new way (Access Entry API) replacing aws-auth ConfigMap.

✅ How to fix it (2 ways)
🔹 Option 1: Quick fix from Console (recommended)

Click:
👉 Create access entry

Then:

Select your IAM user/role
Give permission:
AmazonEKSClusterAdminPolicy

EKS uses IAM for authentication and Kubernetes RBAC for authorization. Even if a user can access the cluster in AWS, they must be explicitly mapped using access entries or aws-auth to interact with Kubernetes resources.

or with script:

Using CLI (if you have access via creator)

aws eks create-access-entry \
  --cluster-name eks-1 \
  --principal-arn <your-iam-arn> \
  --type STANDARD

aws eks associate-access-policy \
  --cluster-name eks-1 \
  --principal-arn <your-iam-arn> \
  --policy-arn arn:aws:eks::aws:cluster-access-policy/AmazonEKSClusterAdminPolicy \
  --access-scope type=cluster

7. If we need to get EKS resources to our server we will give below command: (it will take some time to reflect in the server)

aws eks update-kubeconfig --name eks-1 --region ap-south-1

<img width="539" height="56" alt="image" src="https://github.com/user-attachments/assets/38f060fb-6216-4686-9835-df79334885bf" />


---> DEPLOYMENT PROCESS:

===> Here we are creating a new FARGATE PROFILE because using default namespaces which EKS has created (default, kube-system) also we can do but for proper understanding creating NEW FARGATE PROFILE WITH OUR CUSTOM NAMESPACE

eksctl create fargateprofile \
    --cluster eks-1 \
    --region ap-south-1 \
    --name alb-sample-app \
    --namespace game-2048

<img width="539" height="87" alt="image" src="https://github.com/user-attachments/assets/4c77aea3-9880-4f62-a676-aa73fcd03da8" />
<img width="729" height="289" alt="image" src="https://github.com/user-attachments/assets/2084272c-ce9d-406d-ab56-ec1724db1f0d" />

===> this is the deployment file path written to deploy the application:

https://github.com/kubernetes-sigs/aws-load-balancer-controller/blob/main/docs/examples/2048/2048_full.yaml

Note: we can make changes according to our requirement (replicas, images, labels etc..)

===> We can see all the resources are created

<img width="507" height="194" alt="image" src="https://github.com/user-attachments/assets/6f8b9692-5156-4c76-b3fb-ebd9f4958720" />

===> Now we need to access the application using Ingress controller --- it will use our ingress-2048 and create LOAD BALANCER for this we need to configure load balancer.

===> To create ALB Ingress controller/Ingress controller we need to create IAM OIDC provider because for the controller we have created need to access ALB, to communicate we need IAM OIDC to communicate to other AWS services so are creating OIDC provider:

Note: We have even create using console, but same we are executing with command:

eksctl utils associate-iam-oidc-provider --cluster eks-1 --region ap-south-1 --approve

<img width="681" height="43" alt="image" src="https://github.com/user-attachments/assets/ed2d6b6a-3900-4935-87ac-a6dc48ee0c39" />


===> After creating follow below README.MD file and complete it

Note: In Kubernetes any controller is a pod we are trying to grant ACCESS to AWS services such as ALB to talk to API.

How to setup alb add on: (While creating serviceaccount just change CLUSTER NAME & AWS ACCOUNT NAME, REGION NAME), If we get error for below script execute again it will fix.

eksctl create iamserviceaccount \
  --cluster=eks-1 \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam:032621928724:policy/AWSLoadBalancerControllerIAMPolicy \
  --region ap-south-1 \
  --approve

  

# AWS EKS 

## Introduction

## Table of Contents:

1. [Understanding Kubernetes Fundamentals](#understanding-kubernetes-fundamentals)
   - 1.1 [EKS vs. Self-Managed Kubernetes: Pros and Cons](#eks-vs-self-managed-kubernetes-pros-and-cons)

2. [Setting up your AWS Environment for EKS](#setting-up-your-aws-environment-for-eks)
   - 2.1 [Creating an AWS Account and Setting up IAM Users](#creating-an-aws-account-and-setting-up-iam-users)
   - 2.2 [Configuring the AWS CLI and kubectl](#configuring-the-aws-cli-and-kubectl)
   - 2.3 [Preparing Networking and Security Groups for EKS](#preparing-networking-and-security-groups-for-eks)

3. [Launching your First EKS Cluster](#launching-your-first-eks-cluster)
   - 3.1 [Using the EKS Console for Cluster Creation](#using-the-eks-console-for-cluster-creation)
   - 3.2 [Launching an EKS Cluster via AWS CLI](#launching-an-eks-cluster-via-aws-cli)
   - 3.3 [Authenticating with the EKS Cluster](#authenticating-with-the-eks-cluster)

4. [Deploying Applications on EKS](#deploying-applications-on-eks)
   - 4.1 [Containerizing Applications with Docker](#containerizing-applications-with-docker)
   - 4.2 [Writing Kubernetes Deployment YAMLs](#writing-kubernetes-deployment-yamls)
   - 4.3 [Deploying Applications to EKS: Step-by-step Guide](#deploying-applications-to-eks-step-by-step-guide)

## Understanding Kubernetes Fundamentals

### 1.1 EKS vs. Self-Managed Kubernetes: Pros and Cons

1.1.1 EKS (Amazon Elastic Kubernetes Service)
Pros:

    Managed Control Plane: EKS takes care of managing the Kubernetes control plane components, such as the API server, controller manager, and etcd. AWS handles upgrades, patches, and ensures high availability of the control plane.

    Automated Updates: EKS automatically updates the Kubernetes version, eliminating the need for manual intervention and ensuring that the cluster stays up-to-date with the latest features and security patches.

    Scalability: EKS can automatically scale the Kubernetes control plane based on demand, ensuring the cluster remains responsive as the workload increases.

    AWS Integration: EKS seamlessly integrates with various AWS services, such as AWS IAM for authentication and authorization, Amazon VPC for networking, and AWS Load Balancers for service exposure.

    Security and Compliance: EKS is designed to meet various security standards and compliance requirements, providing a secure and compliant environment for running containerized workloads.

    Monitoring and Logging: EKS integrates with AWS CloudWatch for monitoring cluster health and performance metrics, making it easier to track and troubleshoot issues.

    Ecosystem and Community: Being a managed service, EKS benefits from continuous improvement, support, and contributions from the broader Kubernetes community.

Cons:

    Cost: EKS is a managed service, and this convenience comes at a cost. Running an EKS cluster may be more expensive compared to self-managed Kubernetes, especially for large-scale deployments.

    Less Control: While EKS provides a great deal of automation, it also means that you have less control over the underlying infrastructure and some Kubernetes configurations.

1.1.2 Self-Managed Kubernetes on EC2 Instances
Pros:

    Cost-Effective: Self-managed Kubernetes allows you to take advantage of EC2 spot instances and reserved instances, potentially reducing the overall cost of running Kubernetes clusters.

    Flexibility: With self-managed Kubernetes, you have full control over the cluster's configuration and infrastructure, enabling customization and optimization for specific use cases.

    EKS-Compatible: Self-managed Kubernetes on AWS can still leverage various AWS services and features, enabling integration with existing AWS resources.

    Experimental Features: Self-managed Kubernetes allows you to experiment with the latest Kubernetes features and versions before they are officially supported by EKS.

Cons:

    Complexity: Setting up and managing a self-managed Kubernetes cluster can be complex and time-consuming, especially for those new to Kubernetes or AWS.

    Maintenance Overhead: Self-managed clusters require manual management of Kubernetes control plane updates, patches, and high availability.

    Scaling Challenges: Scaling the control plane of a self-managed cluster can be challenging, and it requires careful planning to ensure high availability during scaling events.

    Security and Compliance: Self-managed clusters may require additional effort to implement best practices for security and compliance compared to EKS, which comes with some built-in security features.

    Lack of Automation: Self-managed Kubernetes requires more manual intervention and scripting for certain operations, which can increase the risk of human error.

## Setting up your AWS Environment for EKS

Sure! Let's go into detail for each subsection:

## 2.1 Creating an AWS Account and Setting up IAM Users

Creating an AWS account is the first step to access and utilize AWS services, including Amazon Elastic Kubernetes Service (EKS). Here's a step-by-step guide to creating an AWS account and setting up IAM users:

1. **Create an AWS Account**:
   - Go to the AWS website (https://aws.amazon.com/) and click on the "Create an AWS Account" button.
   - Follow the on-screen instructions to provide your email address, password, and required account details.
   - Enter your payment information to verify your identity and set up billing.

2. **Access AWS Management Console**:
   - After creating the account, you will receive a verification email. Follow the link in the email to verify your account.
   - Log in to the AWS Management Console using your email address and password.

3. **Set up Multi-Factor Authentication (MFA)** (Optional but recommended):
   - Once you are logged in, set up MFA to add an extra layer of security to your AWS account. You can use MFA with a virtual MFA device or a hardware MFA device.

4. **Create IAM Users**:
   - Go to the IAM (Identity and Access Management) service in the AWS Management Console.
   - Click on "Users" in the left-hand navigation pane and then click on "Add user."
   - Enter a username for the new IAM user and select the access type (Programmatic access, AWS Management Console access, or both).
   - Choose the permissions for the IAM user by adding them to one or more IAM groups or attaching policies directly.
   - Optionally, set permissions boundary, tags, and enable MFA for the IAM user.

5. **Access Keys (for Programmatic Access)**:
   - If you selected "Programmatic access" during user creation, you will receive access keys (Access Key ID and Secret Access Key).
   - Store these access keys securely, as they will be used to authenticate API requests made to AWS services.

## 2.2 Configuring the AWS CLI and kubectl

With IAM users set up, you can now configure the AWS CLI and kubectl on your local machine to interact with AWS services and EKS clusters:

1. **Installing the AWS CLI**:
   - Download and install the AWS CLI on your local machine. You can find installation instructions for various operating systems [here](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html).

2. **Configuring AWS CLI Credentials**:
   - Open a terminal or command prompt and run the following command:
     ```
     aws configure
     ```
   - Enter the access key ID and secret access key of the IAM user you created earlier.
   - Choose a default region and output format for AWS CLI commands.

3. **Installing kubectl**:
   - Install kubectl on your local machine. Instructions can be found [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

4. **Configuring kubectl for EKS**:
   - Once kubectl is installed, you need to configure it to work with your EKS cluster.
   - In the AWS Management Console, go to the EKS service and select your cluster.
   - Click on the "Config" button and follow the instructions to update your kubeconfig file. Alternatively, you can use the AWS CLI to update the kubeconfig file:
     ```
     aws eks update-kubeconfig --name your-cluster-name
     ```
   - Verify the configuration by running a kubectl command against your EKS cluster:
     ```
     kubectl get nodes
     ```

## 2.3 Preparing Networking and Security Groups for EKS

Before launching an EKS cluster, you need to prepare the networking and security groups to ensure proper communication and security within the cluster:

1. **Creating an Amazon VPC (Virtual Private Cloud)**:
   - Go to the AWS Management Console and navigate to the VPC service.
   - Click on "Create VPC" and enter the necessary details like VPC name, IPv4 CIDR block, and subnets.
   - Create public and private subnets to distribute resources in different availability zones.

Sure! Let's go into detail for each of the points:

2. **Configuring Security Groups**

**Security Groups** are a fundamental aspect of Amazon Web Services (AWS) that act as virtual firewalls for your AWS resources, including Amazon Elastic Kubernetes Service (EKS) clusters. Security Groups control inbound and outbound traffic to and from these resources based on rules you define. Here's a step-by-step guide on configuring Security Groups for your EKS cluster:

1. **Create a Security Group**:
   - Go to the AWS Management Console and navigate to the Amazon VPC service.
   - Click on "Security Groups" in the left-hand navigation pane.
   - Click on "Create Security Group."
   - Provide a name and description for the Security Group.
   - Select the appropriate VPC for the Security Group.

2. **Inbound Rules**:
   - Define inbound rules to control incoming traffic to your EKS worker nodes.
   - By default, all inbound traffic is denied unless you explicitly allow it.
   - Common inbound rules include allowing SSH (port 22) access for administrative purposes and allowing ingress traffic from specific CIDR blocks or Security Groups.

3. **Outbound Rules**:
   - Define outbound rules to control the traffic leaving your EKS worker nodes.
   - By default, all outbound traffic is allowed unless you explicitly deny it.
   - For security purposes, you can restrict outbound traffic to specific destinations or ports.

4. **Security Group IDs**:
   - After creating the Security Group, you'll receive a Security Group ID. This ID will be used when launching your EKS worker nodes.

5. **Attach Security Group to EKS Worker Nodes**:
   - When launching the EKS worker nodes, specify the Security Group ID in the launch configuration. This associates the Security Group with the worker nodes, allowing them to communicate based on the defined rules.

Configuring Security Groups ensures that only the necessary traffic is allowed to and from your EKS worker nodes, enhancing the security of your EKS cluster.

3. **Setting Up Internet Gateway (IGW)**

An **Internet Gateway (IGW)** is a horizontally scaled, redundant, and highly available AWS resource that allows communication between your VPC and the internet. To enable EKS worker nodes to access the internet for tasks like pulling container images, you need to set up an Internet Gateway in your VPC. Here's how to do it:

1. **Create an Internet Gateway**:
   - Go to the AWS Management Console and navigate to the Amazon VPC service.
   - Click on "Internet Gateways" in the left-hand navigation pane.
   - Click on "Create Internet Gateway."
   - Provide a name for the Internet Gateway and click "Create Internet Gateway."

2. **Attach Internet Gateway to VPC**:
   - After creating the Internet Gateway, select the Internet Gateway in the list and click on "Attach to VPC."
   - Choose the VPC to which you want to attach the Internet Gateway and click "Attach."

3. **Update Route Tables**:
   - Go to "Route Tables" in the Amazon VPC service.
   - Identify the Route Table associated with the private subnets where your EKS worker nodes will be deployed.
   - Edit the Route Table and add a route with the destination `0.0.0.0/0` (all traffic) and the Internet Gateway ID as the target.

By setting up an Internet Gateway and updating the Route Tables, you provide internet access to your EKS worker nodes, enabling them to interact with external resources like container registries and external services.

4. **Configuring IAM Policies**

**Identity and Access Management (IAM)** is a service in AWS that allows you to manage access to AWS resources securely. IAM policies define permissions that specify what actions are allowed or denied on specific AWS resources. For your EKS cluster, you'll need to configure IAM policies to grant necessary permissions to your worker nodes and other resources. Here's how to do it:

1. **Create a Custom IAM Policy**:
   - Go to the AWS Management Console and navigate to the IAM service.
   - Click on "Policies" in the left-hand navigation pane.
   - Click on "Create policy."
   - Choose "JSON" as the policy language and define the permissions required for your EKS cluster. For example, you might need permissions for EC2 instances, Auto Scaling, Elastic Load Balancing, and accessing ECR (Elastic Container Registry).

2. **Attach the IAM Policy to IAM Roles**:
   - Go to "Roles" in the IAM service and select the IAM role that your EKS worker nodes will assume.
   - Click on "Attach policies" and search for the custom IAM policy you created in the previous step.
   - Attach the policy to the IAM role.

3. **Update EKS Worker Node Launch Configuration**:
   - When launching your EKS worker nodes, specify the IAM role ARN (Amazon Resource Name) of the IAM role that includes the necessary IAM policy.
   - The IAM role allows the worker nodes to authenticate with the EKS cluster and access AWS resources based on the permissions defined in the attached IAM policy.

By configuring IAM policies and associating them with IAM roles, you grant specific permissions to your EKS worker nodes, ensuring they can interact with AWS resources as needed while maintaining security and access control.

By completing these steps, your AWS environment is ready to host an Amazon EKS cluster. You can proceed with creating an EKS cluster using the AWS Management Console or AWS CLI as described in section 3.

