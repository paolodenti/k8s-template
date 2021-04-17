# Basic K8s Template

Example with

* mongodb backend
* mongo express on mongo db backend
* loadbalancer on mongoexpress on port 8080
* persistent volume claim below mongodb
* configmap to store configuration
* opaque secret to store mongodb credentials

## How to run

```bash
# enable ingress on minikube
minikube addons enable ingress
kubectl delete -A ValidatingWebhookConfiguration ingress-nginx-admission
```

```bash
# create mongo credentials
kubectl create secret generic mongo-secret \
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

# configure ingress
kubectl apply -f ingress.yaml 
```

### to access the service

```bash
minikube ip
```

and navigate to ip

To access the service directly

```bash
minikube service mongo-express-service
```
