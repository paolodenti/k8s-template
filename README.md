# Basic K8s Template (for minikube)

Features

* Vue.js SPA frontend ([sources here](https://github.com/paolodenti/k8s-template-client))
* Node.js Express REST API backend ([sources here](https://github.com/paolodenti/k8s-template-server))
* Mongo db
* Mongo Express admin
* Persistent volume claim for mongodb
* Configmap to store app configuration
* Opaque secret to store mongodb credentials
* Ingress multihost and multipath
* Namespaced
* Autoscaling on the rest api pod

![node diagram](docs/node.svg?raw=true "Node Diagram")

## How to run

```bash
# minikube and kubectl must be installed
minikube start

# create custom namespace
kubectl apply -f namespace.yaml

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

# enable ingress on minikube
minikube addons enable ingress
kubectl delete -A ValidatingWebhookConfiguration ingress-nginx-admission
kubectl apply -f ingress.yaml

# set up autoscaling
kubectl apply -f metrics-server.yaml
kubectl apply -f autoscaler.yaml
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

to check the autoscaling in real time (it takes at least 1 minute to see valid CPU data after applying the autoscaling descriptor)

```bash
get hpa server-deployment -n k8s-template --watch
```

Example of autoscaling during a burst of requests to the API.

![Autoscaling](docs/autoscaling.png?raw=true "Autoscaling")

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

## Windows notes

The ingress ip cannot be reached if you are not using hyperv. Enable hyperv using the instructions below.

```bash
minikube config unset vm-driver
minikube config unset driver
minikube config set vm-driver hyperv
minikube delete
minikube start
```

## Useful tools

If you want to set the k8s-template as default, you can use kubectx

```bash
brew install kubectx
kubens k8s-template
```

## Additional notes

metrics-server is taken from here: `https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml` and customized changing the namespace and startup args
