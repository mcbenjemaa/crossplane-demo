
### Configure and install

Start with a Self-Hosted Crossplane

```
kubectl create namespace crossplane-system

helm repo add crossplane-stable https://charts.crossplane.io/stable
helm repo update

helm install crossplane --namespace crossplane-system crossplane-stable/crossplane --version 1.3.0

```

Check Crossplane Status
```
helm list -n crossplane-system

kubectl get all -n crossplane-system
```

#### Install CLI

```
curl -sL https://raw.githubusercontent.com/crossplane/crossplane/master/install.sh | sh
```
