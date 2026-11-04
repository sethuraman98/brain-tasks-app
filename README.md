# Brain Tasks – End-to-End DevOps Project

## Project Overview

Brain Tasks is a React-based task management application deployed on Amazon EKS using an automated CI/CD pipeline built with AWS CodePipeline and AWS CodeBuild.

The project demonstrates containerization, image management, Kubernetes deployment, continuous integration, continuous deployment, and cloud infrastructure monitoring.

## Project Objectives

- Containerize the Brain Tasks application using Docker.
- Store Docker images in Amazon ECR.
- Deploy the application to Amazon EKS.
- Automate application builds using AWS CodeBuild.
- Automate deployment using AWS CodePipeline.
- Expose the application through a Kubernetes LoadBalancer service.
- Monitor application and infrastructure health.
- Document deployment, troubleshooting, and cleanup procedures.

## Architecture

```text
                    GitHub Repository
                           |
                           v
                    AWS CodePipeline
                           |
                           v
                     AWS CodeBuild
                    (Docker Build)
                           |
                           v
                      Amazon ECR
                  (Container Images)
                           |
                           v
                    Amazon EKS Cluster
                           |
                  Kubernetes Deployment
                      (2 Replicas)
                           |
                           v
                 Kubernetes LoadBalancer
                           |
                           v
                   Brain Tasks Application
```

## Technologies Used

| Technology | Purpose |
|---|---|
| Git and GitHub | Source code management |
| Docker | Containerization |
| Amazon ECR | Container image repository |
| AWS CodeBuild | Automated image build |
| AWS CodePipeline | CI/CD orchestration |
| Amazon EKS | Managed Kubernetes |
| Kubernetes | Application deployment and service management |
| AWS EC2 | Project administration and compute |
| AWS IAM | Permissions and access management |
| AWS CloudWatch | Monitoring and logging |

## AWS Environment

| Configuration | Value |
|---|---|
| AWS Region | `ap-southeast-2` (Sydney) |
| EKS Cluster | `brain-tasks-cluster` |
| EKS Node Group | `brain-tasks-workers` |
| Worker Nodes | 2 |
| Worker Instance Type | `t3.small` |
| ECR Repository | `brain-tasks-app` |
| Pipeline Name | `brain-tasks-pipeline` |
| Kubernetes Deployment | `brain-tasks` |
| Kubernetes Service | `brain-tasks-service` |
| Application Port | 80 |

## Application URL

The deployed application is accessible through the AWS LoadBalancer:

[Open Brain Tasks Application](http://a01987df151a7498d9db77f8bad712ff-1140526709.ap-southeast-2.elb.amazonaws.com)

> This URL is associated with the current AWS deployment and may stop working after resource cleanup.

## Repository Structure

```text
Brain-Tasks-App/
│
├── Dockerfile
├── .dockerignore
├── buildspec.yml
├── deployment.yaml
├── service.yaml
└── README.md
```

The application source and generated build files are maintained in the GitHub repository.

## Prerequisites

Before starting the deployment, ensure the following are available:

- AWS account with required permissions
- AWS CLI
- Docker
- Git
- kubectl
- eksctl
- GitHub repository
- AWS CodeBuild project
- AWS CodePipeline
- Amazon ECR repository
- Amazon EKS cluster

## Step 1: Clone the Repository

```bash
git clone https://github.com/Vennilavanguvi/Brain-Tasks-App.git
cd Brain-Tasks-App
```

Replace the repository URL with your own GitHub repository URL.

## Step 2: Build the Docker Image

Build the application image:

```bash
docker build -t brain-tasks-app:v1 .
```

Verify the image:

```bash
docker images
```

## Step 3: Create an Amazon ECR Repository

The ECR repository used for this project is:

```text
brain-tasks-app
```

Repository URI:

```text
353452028567.dkr.ecr.ap-southeast-2.amazonaws.com/brain-tasks-app
```

Authenticate Docker with Amazon ECR:

```bash
aws ecr get-login-password --region ap-southeast-2 | docker login --username AWS --password-stdin 353452028567.dkr.ecr.ap-southeast-2.amazonaws.com
```

Tag the image:

```bash
docker tag brain-tasks-app:v1 353452028567.dkr.ecr.ap-southeast-2.amazonaws.com/brain-tasks-app:v1
```

Push the image:

```bash
docker push 353452028567.dkr.ecr.ap-southeast-2.amazonaws.com/brain-tasks-app:v1
```

## Step 4: Configure Amazon EKS

The EKS cluster used in this project:

```text
Cluster Name: brain-tasks-cluster
Region: ap-southeast-2
Node Group: brain-tasks-workers
```

Verify the cluster:

```bash
aws eks describe-cluster \
  --name brain-tasks-cluster \
  --region ap-southeast-2
```

Configure kubectl:

```bash
aws eks update-kubeconfig \
  --region ap-southeast-2 \
  --name brain-tasks-cluster
```

Verify cluster connectivity:

```bash
kubectl get nodes
```

Expected result: two worker nodes in `Ready` status.

## Step 5: Kubernetes Deployment

The Kubernetes deployment is configured in `deployment.yaml`.

The deployment uses:

- Deployment name: `brain-tasks`
- Replicas: 2
- Container port: 80
- Amazon ECR image: `brain-tasks-app:v1`

Apply the deployment:

```bash
kubectl apply -f deployment.yaml
```

Verify:

```bash
kubectl get deployments
kubectl get pods -o wide
```

Expected result: two application pods in `Running` status.

## Step 6: Configure Kubernetes LoadBalancer

The Kubernetes service is defined in `service.yaml`.

The service configuration:

- Service name: `brain-tasks-service`
- Type: `LoadBalancer`
- Port: 80
- Target port: 80

Apply the service:

```bash
kubectl apply -f service.yaml
```

Verify:

```bash
kubectl get svc
```

Retrieve the application URL:

```bash
kubectl get svc brain-tasks-service
```

Open the LoadBalancer DNS address in a browser to access the application.

## Step 7: AWS CodeBuild

AWS CodeBuild automates the application build and Docker image creation.

The CodeBuild project is configured to:

1. Retrieve source code from GitHub through CodePipeline.
2. Execute the build commands defined in `buildspec.yml`.
3. Build the Docker image.
4. Authenticate with Amazon ECR.
5. Push the image to the ECR repository.

The build stage was verified successfully through AWS CodePipeline.

## Step 8: AWS CodePipeline

Pipeline name:

```text
brain-tasks-pipeline
```

The pipeline consists of three stages:

### Source Stage

- Source provider: GitHub through AWS CodeConnections/GitHub App.
- Retrieves the latest application code.

### Build Stage

- Build provider: AWS CodeBuild.
- Builds the Docker image.
- Pushes the image to Amazon ECR.

### Deploy Stage

- Deployment provider: Amazon EKS.
- Deployment configuration: Kubernetes manifests.
- Manifest files:
  - `deployment.yaml`
  - `service.yaml`

The pipeline was manually triggered using **Release change**.

### Successful Pipeline Execution

The final successful pipeline execution confirmed:

| Stage | Status |
|---|---|
| Source | Succeeded |
| Build | Succeeded |
| Deploy | Succeeded |

The application was successfully deployed to Amazon EKS.

## Step 9: Verify Deployment

Check Kubernetes resources:

```bash
kubectl get deployments
kubectl get pods
kubectl get svc
```

Check deployment rollout:

```bash
kubectl rollout status deployment/brain-tasks
```

Expected result:

- Deployment available
- Two pods running
- LoadBalancer service active
- Application accessible through browser

## Troubleshooting

### 1. SSH Connection Reset During EKS Creation

**Error:**

```text
client_loop: send disconnect: Connection reset
```

**Cause:**

The SSH connection was interrupted while the EKS cluster was being created.

**Resolution:**

1. Reconnect to the EC2 instance.
2. Check the CloudFormation stack status.
3. Verify whether the EKS cluster creation completed.
4. Continue after confirming the stack reached `CREATE_COMPLETE`.

The SSH session interruption did not prevent the cluster creation from completing.

### 2. EKS Node Group IAM Permission Error

**Error:**

```text
iam:ListAttachedRolePolicies
```

**Cause:**

The EC2 IAM role lacked permissions required during node group creation.

**Resolution:**

- Updated the IAM permissions for the lab environment.
- Removed the failed node group stack.
- Recreated the node group.
- Verified that both worker nodes reached `Ready` status.

> Administrator-level permissions were temporarily used during troubleshooting. They should be removed after the project is completed. Avoid using broad administrator permissions in production.

### 3. CodePipeline EKS Deployment Authentication Error

**Error:**

```text
the server has asked for the client to provide credentials
```

**Cause:**

The CodePipeline deployment role was not authorized to access the Kubernetes cluster.

**Resolution:**

Created an EKS access entry for the CodePipeline service role and associated an EKS access policy.

After correcting access permissions, the pipeline deployment succeeded.

> The lab used cluster administrator access for the pipeline role. In production, use least-privilege permissions and restrict deployment access to the required Kubernetes namespace and resources.

### 4. CodePipeline Public Subnet Error

**Error:**

```text
Public subnets detected in the EKS cluster or action configuration.
```

**Cause:**

The EKS deployment action was configured with public subnet IDs.

**Resolution:**

Reviewed the EKS node group subnet configuration and deployment action networking settings.

The deployment action was updated and successfully retried.

## AWS Resource Cleanup

AWS EKS, EC2, and LoadBalancer resources can generate charges even when the application is not actively used.

Before cleanup, capture the required screenshots and ensure your project documentation is saved.

### Step 1: Delete the Kubernetes LoadBalancer Service

```bash
kubectl delete service brain-tasks-service
```

Wait until the associated AWS LoadBalancer is deleted.

### Step 2: Delete the Kubernetes Deployment

```bash
kubectl delete deployment brain-tasks
```

### Step 3: Delete the EKS Node Group

```bash
eksctl delete nodegroup \
  --cluster brain-tasks-cluster \
  --name brain-tasks-workers \
  --region ap-southeast-2
```

Wait for deletion to complete.

### Step 4: Delete the EKS Cluster

```bash
eksctl delete cluster \
  --name brain-tasks-cluster \
  --region ap-southeast-2
```

This deletes the cluster and associated managed infrastructure.

### Step 5: Review Remaining AWS Resources

Check the following services for resources that may continue to incur charges:

- EC2 instances
- EBS volumes and snapshots
- Elastic Load Balancers
- Amazon ECR images and repositories
- NAT Gateways
- CloudWatch log groups
- CodeBuild projects
- CodePipeline pipelines

Delete only resources created for this project that you no longer need.

Do not delete shared or default VPC resources without verifying their dependencies.

### Step 6: Remove Temporary IAM Permissions

Review the EC2 IAM role and CodePipeline IAM role.

Remove temporary AdministratorAccess permissions if they were added for troubleshooting.

Keep only the permissions required for the intended workflow.

## Project Outcome

This project successfully demonstrates an end-to-end DevOps deployment workflow.

The Brain Tasks application was:

- Containerized using Docker
- Stored in Amazon ECR
- Deployed to Amazon EKS
- Exposed using a Kubernetes LoadBalancer
- Automated using AWS CodeBuild and AWS CodePipeline
- Verified through successful pipeline execution and browser access

## Key Learning Outcomes

- Docker image creation and registry management
- Amazon ECR integration
- Kubernetes deployments and services
- Amazon EKS cluster management
- AWS IAM and Kubernetes access management
- CI/CD pipeline configuration
- AWS troubleshooting
- Application deployment verification
- Cloud resource cleanup and cost management

---

**Project Status:** Successfully deployed and verified through AWS CodePipeline.

**Author:** Sethu Raman C

**GitHub Repository:** https://github.com/sethuraman98/brain-tasks-app
