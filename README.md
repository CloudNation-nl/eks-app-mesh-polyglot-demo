# Welcome!

## Kubernetes mini-lab:
We will start interacting with kubernetes on a basic level
To get started run the following command (note the period and the space in front!)

```
. minilab.sh
```
This will install all prerequisites for interacting with kubernetes locally. We use `minikube` for this.

To start the cluster run:
```
minikube start
```

We will now create a pod and expose it using `kubectl` command line interface.
```
minikubectl create deployment nginx --image=nginx
minikubectl expose deployment nginx --type=NodePort --port=80
```
This creates a deployment of 1 `nginx` container.
You can get the url of this service by running `minikube service nginx --url`. 
You can use it to get the server response by running:
```
curl $(minikube service nginx --url)
```

And that's it! You've created your first kubernetes deployment and service!
To free up some memory, you can run
```
minikube delete
```


This repository is used for many AWS EKS workshops in https://catalog.workshops.aws/eks-immersionday/en-US

## 1. Workshop on Polyglot Microservices in EKS

![frontend](workshop/images/lbui.png)

To Run this workshop,follow the below steps: 

### Run the setup script
```
. setup.sh
```


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
![fronteend](workshop/images/workshopui.png)

You can add products and see the below details
![fronteend](workshop/images/addproducts.png)