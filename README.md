# Basic K8s Template (for minikube)

Example with

* namespaced
* mongo db backend
* mongo express on mongo db backend
* persistent volume claim below mongodb
* configmap to store configuration
* opaque secret to store mongodb credentials
* REST API for basic crud, NodeJS, Express, Mongoose
* ingress on port 80

![node diagram](docs/node.svg?raw=true "Node Diagram")

## How to run

```bash
# enable ingress on minikube
minikube addons enable ingress
kubectl delete -A ValidatingWebhookConfiguration ingress-nginx-admission

# create custom namespace
kubectl create namespace k8s-template

# create mongo credentials
kubectl create secret generic mongo-secret \
  --namespace='k8s-template' \
  --from-literal=mongo-root-username='<your username>' \
  --from-literal=mongo-root-password='<your password>'

# create app configuration
kubectl apply -f mongo-configmap.yaml

# create storage
kubectl apply -f mongo-persistentvolumeclaim.yaml

# create mongo deployment and service
kubectl apply -f mongo-deployment.yaml
kubectl apply -f mongo-service.yaml

# create mongo express deployment and service
kubectl apply -f mongo-express-deployment.yaml
kubectl apply -f mongo-express-service.yaml

# create server api deployment and service
kubectl apply -f server-deployment.yaml
kubectl apply -f server-service.yaml

# create client spa deployment and service
kubectl apply -f client-deployment.yaml
kubectl apply -f client-service.yaml

# configure ingress
kubectl apply -f k8s-template-ingress.yaml 
```

### to access the service

#### wait for ip on ingress

```bash
kubectl get ingress -n k8s-template --watch
```

when you get the ip, in `/etc/hosts` set

```text
<ingress ip> admin-k8s-template.info app-k8s-template.info
```

## MacOS notes

Ingress is not supported by default on the docker driver You need to install hyperkit to use the Ingress.

```bash
brew install hyperkit
minikube config unset vm-driver
minikube config unset driver
minikube config set vm-driver hyperkit
minikube delete
minikube start
```

## Useful tools

If you want to set the k8s-template as default, you can use kubectx

```bash
brew install kubectx
kubens k8s-template
```
