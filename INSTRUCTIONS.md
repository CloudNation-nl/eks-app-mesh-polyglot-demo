# Welcome!

## Kubernetes mini-lab (kubectl & manifests):
We will start interacting with kubernetes on a basic level
To get started run the following command (note the period and the space in front!)

```
cd /home/ec2-user/environment/eks-app-mesh-polyglot-demo/
. minilab.sh
```
This will install all prerequisites for interacting with kubernetes locally. We use `minikube` for this.

To start the cluster run:
```
minikube start
```
## Using kubectl
We will now create a pod and expose it using `kubectl` command line interface.
```
minikubectl create deployment nginx --image=nginx
minikubectl expose deployment nginx --type=NodePort --port=80
```
This creates a deployment of 1 `nginx` container.

You can see service information by running
```
minikubectl get services
```

If you see the service listed, can get the url of this service by running `minikube service nginx --url`. 
You can use it to get the server response by running:
```
curl $(minikube service nginx --url)
```

And that's it! You've created your first kubernetes deployment and service!.

### Using manifest files
In the previous section we have created a service using the `kubectl` CLI.
In this section, we will create a new file callled `manifest.yaml` with the following content:
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-manifest
  labels:
    app: nginx-manifest
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-manifest
  template:
    metadata:
      labels:
        app: nginx-manifest
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: nginx-index-file
          mountPath: /usr/share/nginx/html/
      volumes:
      - name: nginx-index-file
        configMap:
          name: index-html-configmap
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-manifest
spec:
  type: NodePort
  selector:
    app: nginx-manifest
  ports:
  - 
    protocol: TCP
    port: 80 # For convenience, port equals to targetPort. Note however that in type=Nodeport an ephermal port is automatically chosen instead
    targetPort: 80
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: index-html-configmap
data:
  index.html: |
    <html>
    <h1>Welcome</h1>
    </br>
    <h1>Hi there! Welcome to the EKS workshop! </h1>
    </html

```
!!Don't forget to SAVE the file after creating!!

Note that this file contains the exact same information as passed via the command line interface.
However, we've added one configuration: a ConfigMap (basically a dataset) with custom front-end!
You can apply this configuration by running:

```bash
minikubectl apply -f "./manifest.yaml"
```

Check the output of the `nginx-manifest` service by running
```
curl $(minikube service nginx-manifest --url) 
```

And that's it! Now you can deploy both using the `kubectl` CLI & using manifest files!

To free up some memory, you can run
```
minikube delete
```


## 1. Workshop on Polyglot Microservices in EKS

To Run this workshop,follow the below steps: 

### Run the setup script
```
. setup.sh
```


### Lab 1: Installing Helm charts
Follow the first lab at https://catalog.workshops.aws/eks-immersionday/en-US/helm/deploy

### Lab 2: Using Iam Roles for Service Accounts (IRSA)
Follow the lab at https://catalog.workshops.aws/eks-immersionday/en-US/irsa

### Lab 3: Observability


NOTE:  Due to a bug, after the setup, run
```
kubectl rollout restart ds/fluent-bit -n amazon-cloudwatch
```
Also, check out the container insights container map!

### Lab 4: Autoscaling
Follow the lab at https://catalog.workshops.aws/eks-immersionday/en-US/autoscaling

#### Kube ops view bugs
NOTE: Due to a bug, instead of installing `kube-ops-view` via helm, use the following commands

```
cd ~/environment
git clone https://codeberg.org/hjacobs/kube-ops-view.git
cd kube-ops-view
cat <<EoF> ~/environment/kube-ops-view/deploy/service.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    application: kube-ops-view
    component: frontend
  name: kube-ops-view
spec:
  selector:
    application: kube-ops-view
    component: frontend
  type: LoadBalancer
  ports:
  - port: 80
    protocol: TCP
    targetPort: 8080
EoF

kubectl apply -k deploy

```


#### Horizontal pod scaling bugs
Second bug: Use version 0.6.0 of metrics server

```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.6.0/components.yaml
```

If you accidentally installed the latest version already, run:
```
wget https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.5.0/components.yaml
kubectl delete -f components.yaml
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.6.0/components.yaml
```


#### Karpenter bugs

To do the tagging, instead of running the commands listed in the workshop, run:
```
SUBNET_IDS=$(aws ec2 describe-subnets --query 'Subnets[?MapPublicIpOnLaunch==`false`].SubnetId' --output text)  
aws ec2 create-tags \
    --resources $(echo $SUBNET_IDS | tr ',' '\n') \
    --tags Key="alpha.eksctl.io/cluster-name",Value="${CLUSTER_NAME}"                                                                                                  

SG_ID=$(aws ec2 describe-security-groups  --filter  Name=tag:Name,Values=eksworkshop-eksctl-node     --query "SecurityGroups[*].GroupId" --output text)

aws ec2 create-tags \
    --resources $SG_ID \
    --tags Key="alpha.eksctl.io/cluster-name",Value="${CLUSTER_NAME}"                                                                                                                                                            
```
eksworkshop-eksctl
Validate the setup using
```
VALIDATION_SUBNETS_IDS=$(aws ec2 describe-subnets --filters Name=tag:"alpha.eksctl.io/cluster-name",Values="${CLUSTER_NAME}" --query "Subnets[].SubnetId" --output text | sed 's/\t/,/')
echo "$SUBNET_IDS == $VALIDATION_SUBNETS_IDS"
```

### Lab 5: Fargate
Follow the lab at https://catalog.workshops.aws/eks-immersionday/en-US/fargate

No bugs!
