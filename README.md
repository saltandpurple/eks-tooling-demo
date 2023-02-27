# EKS + k8s-Tooling

## AWS EKS ###

### Managed control plane
- Basic Info
- How to update

### Integration with other AWS services
- Compute/EC2
- SecGroups + Network Interfaces ??
- ECR
- IAM

## Kubectl ###
### Setup .kubeconfig
Copy kubeconfig from .kubeconfig directory to ~/.kube/config.
Retrieve credentials via AWS EKS.


### Basic commands

### Handling kubeconfig

	kubectl config get-contexts		# Show all available contexts
	kubectl config current-context		# Show currently active context
	kubectl config use-context heimspiel	# Switch context to heimspiel

### Creating objects

	kubectl create namespace nginx					# Create a namespace 
	kubectl apply -f nginx/nginx.yaml -n nginx			# Apply a given manifest in the namespace heimspiel-nginx
	kubectl create deployment nginx2 -n nginx --image=nginx:latest	# Shortcut to create a deployment without a manifest

### Describing objects

	kubectl get pods --all-namespaces		# List pods in all namespaces
	kubectl get pods -n nginx -o wide		# List pods in a certain namespace but with more output
	kubectl describe deployment nginx -n nginx	# Get a detailed description of an object

### Scaling a deployment

	kubectl scale deployment nginx -n nginx --replicas=10	# Adjust number of desired replicas of a deployment (works with statefulset just the same)

### Connecting to a container

	kubectl exec nginx-{HASH} -c nginx --tty -i -n nginx -- bash 	# Connect to a running container - find out pod name with kubectl get pods beforehand

### Creating and using port-forwardings

	kubectl port-forward service/nginx -n nginx 8080:80 	# Create local port-forwarding to a service - Port specification is local:remote
	curl localhost:8080					# If this requires an explanation, you're wrong here.

### Deleting objects

	kubectl delete deployment nginx2 -n nginx	# Delete a resource by specifying it directly
	kubectl delete -f nginx/nginx.yaml		# Delete resources via their manifest

#### There is much more depth to this, obviously. You can fully handle your cluster and all its resources with kubectl.

## Helm

## Lens

