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
# minikube, kubectl and helm v3 must be installed
minikube start

# enable ingress on minikube
minikube addons enable ingress
kubectl delete -A ValidatingWebhookConfiguration ingress-nginx-admission

helm install k8s-template ./k8s-template
```

### Helm information

To re-apply the configuration after the first install

```bash
helm upgrade --install k8s-template ./k8s-template
```

The sequence of files applied by helm is

1. Namespace (namespace.yaml)
1. ServiceAccount (metrics-server.yaml)
1. Secret (secret.yaml)
1. ConfigMap (configmap.yaml)
1. PersistentVolumeClaim (persistentvolumeclaim.yaml)
1. ClusterRole (metrics-server.yaml)
1. ClusterRoleBinding (metrics-server.yaml)
1. RoleBinding (metrics-server.yaml)
1. Service (client.yaml, metrics-server.yaml, mongo-express.yaml, mongo.yaml, server.yaml)
1. Deployment (client.yaml, metrics-server.yaml, mongo-express.yaml, mongo.yaml, server.yaml)
1. HorizontalPodAutoscaler (autoscaler.yaml)
1. Ingress (ingress.yaml)
1. APIService (metrics-server.yaml)

### How access the service

#### Wait for ip ready on ingress

```bash
kubectl get ingress -n k8s-template --watch
```

when you get the ip, in `/etc/hosts` set

```text
<ingress ip> admin-k8s-template.io app-k8s-template.io
```

#### Autoscaling

To check the autoscaling in real time (it takes at least 1 minute to see valid CPU data after applying the autoscaling descriptor)

```bash
kubectl get hpa server-deployment -n k8s-template --watch
```

Example of autoscaling during a burst of requests to the API.

![Autoscaling](docs/autoscaling.png?raw=true "Autoscaling")

## Customize installation values

All the customizable values are in `k8s-template/values.yaml`. You can either change the static values or run helm overriding command line, e.g.

```bash
helm install --set namespace=mynewnamespace k8s-template ./k8s-template
```

### Customize values in secret

The default mongo credentials are in cleartext, encoded base64, inside `k8s-template/values.yaml`.

The (decoded) default values are

```yaml
mongoRootUsername: "username"
mongoRootPassword: "password"
```

If you want to set new credentials you can pass values on the helm command line, after base64 encoding:

```bash
HELM_USERNAME="$(echo -n '<my new username>' | base64)"
HELM_PASSWORD="$(echo -n '<my new password>' | base64)"

helm install \
  --set mongoRootUsername="${HELM_USERNAME}" \
  --set mongoRootPassword="${HELM_PASSWORD}" \
  k8s-template ./k8s-template
```

Even if base64 encoded, your credentials are in cleartext inside values.yaml; **Do not commit your credentials in values.yaml**.

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

metrics-server is taken from here:  
`https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml`  
and customized changing the namespace and startup args
