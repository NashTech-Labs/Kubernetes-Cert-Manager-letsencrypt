# Kubernetes ACME Certificate Setup using Let's Encrypt

This project sets up an ACME certificate for your application deployed in a Kubernetes cluster using **cert-manager** and **Let's Encrypt**. It leverages the `ClusterIssuer` and `Ingress` resources to automate SSL/TLS certificate provisioning for your application.

## Prerequisites

Before using the `issuer.yml` and `ingress.yml`, ensure you have the following components set up:

### 1. **Kubernetes Cluster**

You need a running Kubernetes cluster. You can use any Kubernetes provider, such as Google Kubernetes Engine (GKE), Amazon EKS, Azure AKS.

### 2. **kubectl**

Ensure that you have the `kubectl` command-line tool installed and configured to interact with your Kubernetes cluster. You can verify the connection to your cluster with:

```bash
kubectl get nodes
```
## 3. Install cert-manager
Cert-manager is a Kubernetes tool that automates the management and issuance of SSL/TLS certificates. Install cert-manager using the following steps:

Add the cert-manager Helm repository:
```
helm repo add jetstack https://charts.jetstack.io
helm repo update
```
Install cert-manager using Helm:

```helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace```

Verify that cert-manager is installed and running:

```kubectl get pods --namespace cert-manager```
You should see pods running for cert-manager.

## 4. Install NGINX Ingress Controller
The Ingress resource defined in ingress.yml uses the NGINX ingress controller. If you haven't already installed the NGINX ingress controller, follow these steps:

**Add the official NGINX Ingress Controller Helm repository:**
```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```
**Install the NGINX Ingress Controller with Helm:**

```helm install nginx-ingress ingress-nginx/ingress-nginx --namespace ingress-nginx --create-namespace```
This command will install the NGINX Ingress Controller in the ingress-nginx namespace.

Verify the installation of the NGINX Ingress Controller:
```
kubectl get pods -n ingress-nginx
```
You should see pods running for the NGINX Ingress Controller. If the installation was successful, you should see a pod like nginx-ingress-controller.

## 5. Domain Name
You must have a domain or subdomain pointing to your Kubernetes ingress controller's external IP. You can check the external IP of the ingress controller with:

```kubectl get svc -n ingress-nginx```
You should update your DNS provider to point your domain (e.g., example.com) to this external IP address.

## 6. Deploy Your Application
Ensure that your application is already deployed in the Kubernetes cluster before setting up cert-manager for SSL/TLS certificate provisioning. The Ingress resource (ingress.yml) will point to your service, and the certificate will be issued for the domain used by your application.

## 7. Helm Values
If you're using Helm to deploy the ingress resource, ensure that your Helm values are configured correctly. Update values.yaml as follows:

```yaml
namespace: your-namespace
ingress:
  host: example.com
service:
  port: 80
```
Replace your-namespace with the namespace where your app is deployed and example.com with your actual domain.

# Usage
Once the prerequisites are in place, follow these steps:

## 1. Deploy ClusterIssuer
The issuer.yml file defines a ClusterIssuer for Let's Encrypt, which is used to issue SSL/TLS certificates. To apply the ClusterIssuer, run:

```kubectl apply -f issuer.yml```
This creates the ClusterIssuer resource with the necessary configurations to interact with Let's Encrypt.

## 2. Deploy Ingress Resource
The ingress.yml file defines an Ingress resource that uses cert-manager to request and store the SSL/TLS certificate from Let's Encrypt. The Ingress resource will expose your application via HTTP/HTTPS and apply the certificate.

To deploy the Ingress resource, run:

```kubectl apply -f ingress.yml```<br>
Ensure that the following values are replaced with your actual values:

{{ .Values.namespace }}: The Kubernetes namespace where your application is deployed.<br>
{{ .Values.ingress.host }}: The domain you want to use for the application.<br>
{{ .Values.service.port }}: The port number of the service you are exposing.<br>
name-of-certi-secret: The name of the Kubernetes secret where the certificate will be stored.<br>

## 3. Verify Certificate Creation
Once the Ingress resource is created, cert-manager will automatically request a certificate from Let's Encrypt. To verify that the certificate has been successfully issued, run:

```kubectl get certificate -A```
You should see the certificate details, including its validity period.

## 4. Access Your Application
Once the certificate is issued, the application should be accessible via HTTPS. You can test the connection by navigating to https://example.com (replace with your actual domain) in a browser.

## Conclusion
This setup automates the process of obtaining and renewing SSL/TLS certificates for your application using Let's Encrypt and cert-manager. It leverages Kubernetes' built-in resources like Ingress and cert-manager for secure and seamless certificate management.
