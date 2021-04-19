# Basic K8s Template (for minikube)

Features

* Vue.js SPA frontend ([sources here](https://github.com/paolodenti/k8s-template-client))
* Node.js Express REST API backend ([sources here](https://github.com/paolodenti/k8s-template-server))
* Mongo db
* Mongo Express admin
* Persistent volume claim for mongodb
* Configmap to store app configuration
* Opaque secret to store mongodb credentials
* Ingress multihost
* Namespaced

![node diagram](docs/node.svg?raw=true "Node Diagram")

## How to run

```bash
# minikube and kubectl must be installed
minikube start

# enable ingress on minikube
minikube addons enable ingress
kubectl delete -A ValidatingWebhookConfiguration ingress-nginx-admission

# create custom namespace
kubectl create namespace k8s-template

# create mongo credentials
kubectl create secret generic secret \
  --namespace='k8s-template' \
  --from-literal=mongo-root-username='<your username>' \
  --from-literal=mongo-root-password='<your password>'

# apply all configurations
kubectl apply -f configmap.yaml
kubectl apply -f persistentvolumeclaim.yaml
kubectl apply -f mongo.yaml
kubectl apply -f mongo-express.yaml
kubectl apply -f server.yaml
kubectl apply -f client.yaml
kubectl apply -f ingress.yaml
```

### How access the service

#### Wait for ip ready on ingress

```bash
kubectl get ingress -n k8s-template --watch
```

when you get the ip, in `/etc/hosts` set

```text
<ingress ip> admin-k8s-template.io app-k8s-template.io
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
