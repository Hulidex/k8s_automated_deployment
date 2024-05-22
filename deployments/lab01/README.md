# Instructions


## Ex 01

Using the commands:

`kubectl api-resources`
`kubectl explain`
`kubectl apply` (don't forget --dry-run= flag)

1. Create a Manifest `pod.yaml` that will contain a pod with the name `hello-world010`
and container image `gcr.io/google-samples/hello-app:1.0`
2. Deploy the pod in the cluster
3. Finally delete de pod manually

> [!NOTE]
> Use a text editor for creating the file.

## Ex 02 

Use `kubectl create deployment` with flag --dry-run=client
to create a deployment with name 'nginx' and using the docker image 'nginx'
without modifying a single line of code manually with a text editor. The file
will be called `deployment.yaml`

Solution:

```bash
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > deployment.yaml
```
## Final boss

1. Create a deployment (text editor is not allowed) with the following characteristics:

- name: `replicated-hello`
- image `gcr.io/google-samples/hello-app:1.0`
- replicas: `3`

Use --dry-run=client and -o yaml flags for storing the API object in a file.

2. Deploy

3. Change the deployment to only two replicas and upgrade image version: 

- replicas: `1`
- image: `gcr.io/google-samples/hello-app:2.0`

4. Before applying the changes use command `kubectl diff` so have an idea of the changes that will be applied
in the cluster
5. Apply the changes
6. Check that the changes are applied
7. Destroy the deployment

Solution:

```bash
# Step 1
kubectl create deployment replicated-hello --image=gcr.io/google-samples/hello-app:1.0 --replicas 3 --dry-run=client -o yaml > replicated-hello.yaml

# Step 2
kubectl apply -f replicated-hello.yaml

# Step 3
# You know how to do it
nvim replicated-hello.yaml

# Step 4
kubectl diff -f replicated-hello.yaml  | less

# Step 5
kubectl apply -f replicated-hello.yaml

# Step 6
kubectl get rs -o wide

# Step 7
kubectl delete -f replicated-hello.yaml
```




