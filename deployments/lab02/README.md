# Introduction

Namespaces

## Ex 01

Create a namespace called `dev`

### solution

I will create it declaratively so that we can follow Configuration as Code practises:

```bash
# Create the object and save it to a file DON'T APPLY THE CHANGES
kubectl create ns dev --dry-run=client -o yaml > ns-dev.yaml
```


```bash
# Apply the changes
kubectl apply -f ns-dev.yaml
```

```bash
# List all namespaces and check that it was created
kubectl get ns -v6
```

## Ex 02 

Deploy the hello app from lab01 in the namespace created in the previous exercise

### Solution:


```bash
# Create deployment
kubectl create deployment replicated-hello --image=gcr.io/google-samples/hello-app:1.0 --replicas 3 --namespace dev --dry-run=client -o yaml > replicated-hello.yaml
```

```bash
# apply the deployment
kubectl apply -f replicated-hellow.yaml
```

```bash
# Check the deployment
kubectl get deploy -n dev
```

## Ex 03

Clean the environment

### Solution

```bash
kubectl --delete ns dev
```

