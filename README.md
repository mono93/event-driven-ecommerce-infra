# Event-Driven Ecommerce Infrastructure

This repository contains the infrastructure-as-code (IaC) setup for deploying an event-driven ecommerce application on AWS using Amazon EKS (Elastic Kubernetes Service). The infrastructure includes a Kubernetes cluster, IAM policies, and application deployments for backend and frontend services.

## Architecture Overview

The infrastructure consists of:
- **Amazon EKS Cluster**: Managed Kubernetes cluster for running containerized applications.
- **Kubernetes Manifests**: Deployments, services, and ingress for backend and frontend microservices.
- **IAM Policies**: AWS Identity and Access Management policies for necessary permissions (e.g., ELB and EC2 access).

The application follows an event-driven architecture, likely using services like Amazon SNS, SQS, or EventBridge for asynchronous communication between components.

## Prerequisites

Before deploying this infrastructure, ensure you have the following:

- **AWS Account**: With appropriate permissions to create EKS clusters, IAM roles, and other resources.
- **AWS CLI**: Installed and configured with your AWS credentials.
  ```bash
  aws configure
  ```
- **kubectl**: Kubernetes command-line tool.
  ```bash
  # Install via Homebrew (macOS)
  brew install kubectl
  ```
- **eksctl**: CLI tool for creating and managing EKS clusters.
  ```bash
  # Install via Homebrew
  brew tap weaveworks/tap
  brew install weaveworks/tap/eksctl
  ```
- **Helm** (optional): For managing Kubernetes packages, if needed for additional components.
- **Docker**: If you need to build or push container images locally.

## Setup and Deployment

### 1. Clone the Repository

```bash
git clone <repository-url>
cd event-driven-ecommerce-infra
```

### 2. Create EKS Cluster

Use eksctl to create the EKS cluster as defined in `eks/cluster.yaml`.

```bash
eksctl create cluster -f eks/cluster.yaml
```

This will provision:
- EKS control plane
- Worker nodes (EC2 instances)
- VPC, subnets, and security groups

**Note**: Cluster creation can take 10-15 minutes. Monitor the progress with:
```bash
eksctl get cluster
```

### 3. Configure kubectl

After the cluster is created, update your kubeconfig to connect to the new cluster.

```bash
aws eks update-kubeconfig --region <your-region> --name <cluster-name>
```

Verify connection:
```bash
kubectl get nodes
```

### 4. Apply IAM Policies

The `iam-policy.json` contains the necessary IAM policy for the cluster and services. Attach this policy to the IAM role used by the EKS worker nodes or service accounts as needed.

For example, if using IAM roles for service accounts (IRSA):
- Create an IAM policy with the contents of `iam-policy.json`.
- Associate it with the appropriate service accounts in your Kubernetes deployments.

### 5. Deploy Kubernetes Resources

Apply the Kubernetes manifests in the `k8s/` directory.

```bash
# Deploy backend
kubectl apply -f k8s/deployments/backend-deployment.yaml
kubectl apply -f k8s/services/backend-service.yaml

# Deploy frontend
kubectl apply -f k8s/deployments/frontend-deployment.yaml
kubectl apply -f k8s/services/frontend-service.yaml

# Apply ingress
kubectl apply -f k8s/ingres/ingres.yaml
```

**Note**: Ensure your container images are available in a registry (e.g., ECR) and update the image references in the deployment YAMLs if necessary.

### 6. Verify Deployment

Check the status of your deployments and services:

```bash
kubectl get deployments
kubectl get services
kubectl get ingress
```

Access the application via the ingress URL once it's provisioned.

## Configuration

- **Cluster Configuration**: Modify `eks/cluster.yaml` to adjust cluster size, node types, or networking.
- **Application Deployments**: Update `k8s/deployments/` YAMLs for image versions, environment variables, or resource limits.
- **Ingress**: Configure `k8s/ingres/ingres.yaml` for custom domains, TLS certificates, or load balancer settings.

## Monitoring and Logging

- Use AWS CloudWatch for cluster and application logs.
- Monitor with Kubernetes Dashboard or tools like Prometheus/Grafana.
- Set up alerts for resource usage and errors.

## Security Considerations

- Ensure IAM policies follow the principle of least privilege.
- Use secrets management (e.g., AWS Secrets Manager or Kubernetes secrets) for sensitive data.
- Regularly update cluster and node versions for security patches.
- Enable network policies in Kubernetes for pod-to-pod communication.

## Troubleshooting

- **Cluster Creation Issues**: Check AWS CloudFormation stacks in the console for errors.
- **Pod Failures**: Use `kubectl describe pod <pod-name>` and `kubectl logs <pod-name>` for diagnostics.
- **Network Issues**: Verify security groups and VPC configurations.
- **IAM Permissions**: Ensure your AWS user/role has the necessary permissions.

## Contributing

1. Fork the repository.
2. Create a feature branch.
3. Make changes and test thoroughly.
4. Submit a pull request with a detailed description.

## License

This project is licensed under the MIT License. See LICENSE file for details.

## Support

For issues or questions, please open an issue in this repository or contact the maintainers.