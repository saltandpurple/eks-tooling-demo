# EKS + k8s-Tooling

## Tools used in this workshop

### awscli

> https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

### kubectl

> https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html

### Lens

> https://docs.k8slens.dev/getting-started/install-lens/

### helm

> https://helm.sh/docs/intro/install/

---

## AWS EKS


- Basically a managed control plane
- EKS console shows k8s internals as well as related AWS resources.
- EKS provides managed ha control plane as essential service.
- Well integrated with the other AWS services. 
  - Compute/EC2
    - SecGroups, ENIs all work as expected.
  - Loadbalancers, EBS Volumes will be provisioned dynamically via integrated controllers/drivers.
    - Abstract resource (eg PersistentVolumeClaim, LoadBalancer) is created in k8s, the driver/controller does the rest.
  - IAM
    - Nodegroup Role + Cluster Role
    - Permissions can be provided to individual workloads via ServiceAccount-Annotations or to all workloads via nodegroup role.
  - ECR
    - Repository Permissions integrated with IAM
    - All image repositories can be used, ECR is optional.
  - Vendor lock-in: partial problem, AWS-specified annotations etc are not portable, the rest is (if you make use of IaC).
- Barely used in daily business, except for updates and nodegroup adjustments. Everything else is done with the tools below.
- Can be updated with a 'one-click-solution'. Very easy to use, exceedingly reliable.
---

## Kubectl ###

- Powerful, versatile, easy-to-use CLI. Provides extensive control over resources running on kubernetes as well as the cluster itself. 
- Like most shells, it's accessible on different levels: basic commands are easy and abundant. (Much) More complex operations are equally possible but it quickly becomes hard to read/write.

### Getting access to the cluster - setting up your kubeconfig

We will retrieve our kubeconfig directly from AWS EKS - therefore we setup access to AWS first:

Add the following to your ~/.aws/config (don't forget to add the actual credentials!):

    [profile nzc-k8s-profile]
    region = eu-central-1
    aws_access_key_id = ASK ME FOR THIS
    aws_secret_access_key = ASK ME FOR THIS
    
    [profile k8s-workshop]
    source_profile = nzc-k8s-profile
    role_arn = arn:aws:iam::020746439882:role/eks-admin
    role_session_name = k8s-workshop


Now retrieve your kubeconfig via `awscli`. 

    aws eks update-kubeconfig --region eu-central-1 --name k8s-workshop --profile k8s-workshop

The required config should be written directly to your ~/.kube/config. Check it:

    cat ~/.kube/config

It should look something like this:

```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMvakNDQWVhZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJek1ESXlOekUxTURneU1Wb1hEVE16TURJeU5ERTFNRGd5TVZvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTjZ2ClU3dEEyRTFOa1IybTJFcUFhMzY0Vm5VeWJrRmpSalNrN3pxd2NzRmpmSG9WaUFVUzNVOThveTVvZmhmTjM0V2oKMStUb3pKelVKQjdUNFhjQWxmeDJ1ekYwV2tOczhDS3NHTlZFRDJWQWRROWN4KzZLQ01GVi9YM2VlVlpEZlV2aApMM2crTDA5OUE3RTFzMS9CdHYxTHZMYlZnc2hHcGl3YTNKT0k1WVIrNCtXWnZnYTZGNVdZS3MwQ2srbDNPcGZ6ClVaMXhKa3JKZXlwQlRFN0s5UERoSUlJc0tPNWpQYkNuUTBTclVodm9RUDMwekZoL3J6SlppWm93cWdyNGVpQnoKTnBlT0tIajc3YTE4dFlXMnJGSFRxNzFLcXZtNTNCQklsbXRpNjRBL0JXZFVHZ0QvOHVCRmsxclV4V21sN09zVApZOXNCOU5wTis4emhobUJCV2RFQ0F3RUFBYU5aTUZjd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZHL3NoZXIwUkc1S2ljTVZBT0c1eFpQNlFmSFFNQlVHQTFVZEVRUU8KTUF5Q0NtdDFZbVZ5Ym1WMFpYTXdEUVlKS29aSWh2Y05BUUVMQlFBRGdnRUJBTkpUVVVZVlY1MFV0c2s2aGQzdQp0SXFWWGJJU1RHOHQvNlZ1d0FtTEErWXBsNEdwVDIra0NnMVRndURhWm8wU0VMRmtEaVc0UWVmam5xNktlcWp0CkVpT28vWVF4eHR6R2M2S0w3NFlnMExoKzdkdEJUZGN4Qzc4SmxWZHZYMTVhTWpaRkZ3K0svKzRQNVNpcnlldjUKbUt2VG5WVzRBVms2WHJVR0xSdWxqZjdncnJlWE9tS1Zvb3Y5ME1VR3IweDc5SEZIV3BmUDI4dkphenNqdWdFcAplV293SVJqazVna2wxbnpWNndoVXd1ZkpibjY3QnUrREo0K0pPbHRmb1ViSWhMOVpZWkRwZThXL1dLRE1ZYVNxCnROUE51c3dtaXk5bWlrK04zUVpYb3FwcS9hbkdWSllwdUt4WCtGbmlZWWNaZ2ZBSHV3aFhHejlFbnVtOFJ1di8KSEFzPQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    server: https://DF4D5CD3063DA5115814ED0F43691268.gr7.eu-central-1.eks.amazonaws.com
  name: k8s-workshop
contexts:
- context:
    cluster: k8s-workshop
    user: k8s-workshop
  name: k8s-workshop
currentContext: k8s-workshop
kind: Config
preferences: {}
users:
- name: k8s-workshop
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1beta1
      args:
      - --region
      - eu-central-1
      - eks
      - get-token
      - --cluster-name
      - k8s-workshop
      command: aws
      env:
      - name: AWS_PROFILE
        value: k8s-workshop
      interactiveMode: IfAvailable
      provideClusterInfo: false
```

That's it.

### Basic commands

### Handling kubeconfig

	kubectl config get-contexts		# Show all available contexts
	kubectl config current-context		# Show currently active context
	kubectl config use-context k8s-workshop	# Switch context to heimspiel

### Checking permissions

    kubectl auth can-i --list                                # Get an overview of all permissions for my user
    kubectl auth can-i get pods --as=serviceaccount:default  # Check for permissibility of a specific action (and a different user/sa than myself)

### RBAC related checks can be easily performed with kubectl.


### Creating objects

	kubectl create namespace nginx					# Create a namespace
    kubectl get namespaces                                          # Check that is has been created
    kubectl config set-context --current --namespace=nginx  # Set namespace to nginx for less typing
	kubectl apply -f nginx/nginx.yaml			# Apply a given manifest in the namespace heimspiel-nginx
	kubectl create deployment nginx2 -n nginx2 --image=nginx:latest	# Shortcut to create a deployment without a manifest (namespace must exist)

### In k8s, namespaces are a central mechanism for separation of concerns.


### Describing objects

	kubectl get pods --all-namespaces		# List pods in all namespaces
	kubectl get pods -o wide		# List pods in a certain namespace but with more output
	kubectl describe service nginx	# Get a detailed description of an object

### Scaling a deployment

	kubectl scale deployment nginx -n nginx --replicas=10	# Adjust number of desired replicas of a deployment (works with statefulset just the same)

### Connecting to a container

	kubectl exec nginx-{HASH} -c nginx --tty -i -n nginx -- bash 	# Connect to a running container - find out pod name with kubectl get pods beforehand

### Creating and using port-forwardings

	kubectl port-forward service/nginx 8080:80 	# Create local port-forwarding to a service - Port specification is local:remote
	curl localhost:8080					# If this requires an explanation, you're wrong here.

### You're going to use port-forwardings a LOT when debugging.

### Deleting objects

	kubectl delete -f nginx/nginx.yaml		# Delete resources via their manifest
	kubectl delete deployment nginx2 -n nginx	# Delete a resource by specifying it directly

There is much more depth to this, obviously. You can fully handle your cluster and all its resources with kubectl.

---

## Helm

- "Package manager for k8s"
- More like a release manager, versioning, rollbacks etc - on k8s level
- Release can be any combination of manifests
- We use it primarily for its excellent templating capabilities
- In reality, most manifests are fairly complex but also contain a lot of elements already existent in other manifests
- Chart = the central "unit" for helm
  - basically a parametrized collection of k8s-manifests
- Template + default-values + custom-values + inline-values = Deployable manifest(s)
- Is also based on kubectl - uses your existing kubeconfig
- Doesn't require anything to be installed in the cluster, just the tool on your local machinew


### Doings
```
- Take a look at the templates and explain a few different k8s concepts used
- Statefulset vs. Deployment
```

### Printing out the resulting manifest with variables filled in

    helm template mariadb/ --debug
    helm template mariadb/ --debug -f helm/dev-values.yaml --set secret.rootPassword=unsafePassword123   # Override with file and inline value


### Installing/upgrading helm releases
helm install/upgrade | Release-Name Chart | Namespace | Value-Overrides (with files or direct inputs) | Create NS if not exists | Install if not exists

    helm install/upgrade mariadb mariadb/ -n mariadb -f helm/dev-values.yaml --set secret.rootPassword=unsafePassword123 --create-namespace --install  

### Value hierarchy:
Values are determined by looking at:
- default values, specified in the chart-internal `values.yaml`
- values specified via `-f` parameter from left to right
- values specified via `--set` parameter from left to right
- later values take precedence over earlier ones - the chart-internal values have the LOWEST priority, the last specified value-(file) on the CLI has the HIGHEST priority

### Rolling back/removing releases

    helm rollback mariadb -n mariadb   # Reverts release to previous version
    helm uninstall mariadb -n mariadb  # Removes release

---

## Lens

- Lens is basically a visualized version of kubectl
- Kubectl is awesome! However, in reality we use it mostly for debugging, hacking and specialized lookups. Everything else is done via Lens, due to easy of use and clarity.
- Displays all the standard k8s resources - nothing in the GUI is "Lens-exclusive"
- Uses kubectl in background - therefore, no extra config is required
- Supports CRDs as well

### Doings:
```
- Provide overview
- Show nginx deployment - what we did with kubectl in Lens
- Show MariaDB statefulset created with helm
- Create port-forwarding and access db directly
- Show PVC + LB and corresponding resources in AWS
- Access nginx via Loadbalancer URL
- Show secret
- Rescale nginx
- Delete resources
```

---

## Hands-on / Hacking / everybody breaks stuff

- Please make sure to use your own namespace (specified with the `-n` param ) to avoid conflicts!
