# Kubernetes Ingress Lab using Kind and Traefik

## Project Overview

This project demonstrates how to expose multiple applications running inside a Kubernetes cluster using **Kubernetes Ingress and Traefik**.

I created a local Kubernetes cluster using **Kind (Kubernetes in Docker)** on an AWS EC2 instance. I deployed two application workloads, `blue-app` and `green-app`, exposed them using Kubernetes ClusterIP Services, and configured Traefik Ingress to route HTTP traffic based on URL paths.

The objective was to understand how external traffic reaches applications running inside a Kubernetes cluster.

## Architecture

```text
                  User / Browser
                        |
                        |
                EC2 Public IP:80
                        |
                        |
                 Traefik Ingress
                  Controller
                        |
                 Ingress Resource
                   /         \
                  /           \
              /blue           /green
                |               |
          blue-service     green-service
            ClusterIP        ClusterIP
                |               |
             blue-app        green-app
               Pod              Pod
```

## Technologies Used

- AWS EC2 – Hosted the Kubernetes lab environment.
- Docker – Built and managed the application container image.
- Docker Hub – Stored the application image.
- Kind – Created a Kubernetes cluster using Docker containers.
- Kubernetes – Managed Deployments, Pods, Services, Namespaces and Ingress.
- Traefik – Ingress Controller for HTTP traffic routing.
- Helm – Installed Traefik in the Kubernetes cluster.
- YAML – Defined Kubernetes resources and configuration.

## Project Structure

```text
k8s-ingress-lab/
│
├── README.md
├── kind-config.yaml
├── traefik-values.yaml
│
├── manifests/
│   ├── namespace.yml
│   ├── apps.yml
│   └── ingress.yml
│
└── screenshots/
```

## Implementation Steps

### 1. Created an AWS EC2 Instance

Launched an EC2 instance to host Docker, Kind, kubectl and Helm.

### 2. Installed the Required Tools

Installed and configured:

- Docker
- Kind
- kubectl
- Helm

### 3. Created a Kind Cluster

Created a Kubernetes cluster with port mappings to expose HTTP and HTTPS traffic through the EC2 instance.

```bash
kind create cluster --name ingress-lab --config kind-config.yaml
```

### 4. Installed Traefik Ingress Controller

Added the Traefik Helm repository and installed the Ingress Controller.

```bash
helm repo add traefik https://traefik.github.io/charts

helm repo update

helm upgrade --install traefik traefik/traefik \
  --namespace traefik \
  --create-namespace \
  --values traefik-values.yaml
```

### 5. Created a Kubernetes Namespace

Created a separate namespace named `ingress-lab` to organize the application resources.

```bash
kubectl apply -f manifests/namespace.yml
```

### 6. Deployed Two Applications

Created two Kubernetes Deployments:

- `blue-app`
- `green-app`

Both applications use a container image stored on Docker Hub.

```bash
kubectl apply -f manifests/apps.yml
```

### 7. Created ClusterIP Services

Created two internal Kubernetes Services:

- `blue-service`
- `green-service`

These Services provide stable internal endpoints for the application Pods.

### 8. Configured Ingress Routing

Created an Ingress resource using the Traefik Ingress Controller.

Traffic routing:

| Request path | Destination |
|---|---|
| `/blue` | blue-service |
| `/green` | green-service |

```bash
kubectl apply -f manifests/ingress.yml
```

## Verification Commands

Check the Kubernetes resources:

```bash
kubectl get nodes

kubectl get pods -n ingress-lab

kubectl get services -n ingress-lab

kubectl get ingress -n ingress-lab

kubectl get pods -n traefik

kubectl get services -n traefik
```

Inspect the Ingress configuration:

```bash
kubectl describe ingress -n ingress-lab
```

## Application Testing

After configuring the EC2 Security Group to allow HTTP traffic on port 80, test the following URLs:

```text
http://<EC2-PUBLIC-IP>/blue

http://<EC2-PUBLIC-IP>/green
```

Each path should route traffic to its respective application.

## Screenshots

### 1. Kind Cluster

![Kind Cluster](screenshots/01-kind-cluster.png)

### 2. Traefik Ingress Controller

![Traefik Running](screenshots/02-traefik-running.png)

### 3. Application Pods

![Application Pods](screenshots/03-apps-running.png)

### 4. Ingress Rules

![Ingress Rules](screenshots/04-ingress-rules.png)

### 5. Browser Testing

![Browser Testing](screenshots/05-browser-test.png)

## Key Concepts Learned

Through this hands-on project, I practised:

- Creating a Kubernetes cluster using Kind.
- Understanding the role of an Ingress Controller.
- Difference between Kubernetes Service and Ingress.
- Deploying multiple applications using Deployments.
- Exposing applications internally using ClusterIP Services.
- Configuring path-based routing using Ingress.
- Installing Kubernetes applications using Helm.
- Configuring Kind port mappings.
- Exposing a Kubernetes application through an AWS EC2 instance.
- Troubleshooting image-pull and Kubernetes networking issues.

## Cleanup

To remove the Kubernetes resources and Traefik installation:

```bash
kubectl delete -f manifests/ingress.yml

kubectl delete -f manifests/apps.yml

helm uninstall traefik -n traefik

kind delete cluster --name ingress-lab
```

## Learning Outcome

This project helped me understand how Kubernetes Ingress can route external HTTP traffic to multiple applications through a single entry point, instead of exposing each application separately.

This is a hands-on learning project created for practising Kubernetes networking and Ingress concepts.

**Note:** Update the screenshots and testing results to reflect your actual successful deployment. Do not publish AWS credentials, SSH private keys, Kubernetes secrets or other sensitive information.
