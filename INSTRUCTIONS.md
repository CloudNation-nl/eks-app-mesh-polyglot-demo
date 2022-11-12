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

The rest of the labs can be followed at https://catalog.workshops.aws/eks-immersionday/en-US

### Lab 1: Install the Helm chart
```
helm install workshop helm-chart/
```
You should see below output
```
NOTES:
1. Get the application URL by running these commands:
     NOTE: It may take a few minutes for the LoadBalancer to be available.
           You can watch the status of by running 'kubectl get --namespace workshop svc -w frontend'
  export LB_NAME=$(kubectl get svc --namespace workshop frontend -o jsonpath="{.status.loadBalancer.ingress[*].hostname}")
  echo http://$LB_NAME:9000
 ```

### Get the LoadBalancer url. 
```
export LB_NAME=$(kubectl get svc frontend -n workshop -o jsonpath="{.status.loadBalancer.ingress[*].hostname}") 
echo $LB_NAME:9000
```
Go to the browser and paste this url, you should see below screen
![fronteend - note path is optimized for cloud9](/home/ec2-user/environment/eks-app-mesh-polyglot-demo/images/workshopui.png)

You can add products and see the below details
![fronteend - note path is optimized for cloud9](/home/ec2-user/environment/eks-app-mesh-polyglot-demo/images/addproducts.png)