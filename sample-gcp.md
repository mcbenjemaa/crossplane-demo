#### sample GCP

Quick-start guide with crossplane on `gcp-provider` 

If you don't have crossplane installed on your `Mgmt cluster` Then please install crossplane before you get started [install-crossplane](install-crossplane.md)


#### Install GCP Provider

Crossplane needs a provider configuration in order to be able to create resources in that specific provider (behind the scene it communicate with provider API). 

Crossplane community Already made a getting started `Configuration`

```
kubectl crossplane install configuration registry.upbound.io/xp/getting-started-with-gcp:v1.3.0
```

Wait until all packages become healthy:
```
kubectl get pkg --watch
```

Get GCP Account Keyfile
``` 
# replace this with your own gcp project id and the name of the service account
# that will be created.
PROJECT_ID=my-project
NEW_SA_NAME=test-service-account-name

# create service account
SA="${NEW_SA_NAME}@${PROJECT_ID}.iam.gserviceaccount.com"
gcloud iam service-accounts create $NEW_SA_NAME --project $PROJECT_ID

# enable cloud API
SERVICE="sqladmin.googleapis.com"
gcloud services enable $SERVICE --project $PROJECT_ID

# grant access to cloud API
ROLE="roles/cloudsql.admin"
gcloud projects add-iam-policy-binding --role="$ROLE" $PROJECT_ID --member "serviceAccount:$SA"

# create service account keyfile
gcloud iam service-accounts keys create creds.json --project $PROJECT_ID --iam-account $SA
```

Create a Provider Secret
```
kubectl create secret generic gcp-creds -n crossplane-system --from-file=creds=./creds.json
```

Configure the Provider

We will create the following `ProviderConfig` object to configure credentials for GCP Provider:

```
# replace this with your own gcp project id
PROJECT_ID=my-project
echo "apiVersion: gcp.crossplane.io/v1beta1
kind: ProviderConfig
metadata:
  name: default
spec:
  projectID: ${PROJECT_ID}
  credentials:
    source: Secret
    secretRef:
      namespace: crossplane-system
      name: gcp-creds
      key: creds" | kubectl apply -f -
```

Now, that you have configured crossplane to use GCP provider.

For example, you can check the available custom resource `GkeCluster` that is being created. 

```
$ k explain gkecluster

KIND:     GKECluster
VERSION:  container.gcp.crossplane.io/v1beta1

DESCRIPTION:
     A GKECluster is a managed resource that represents a Google Kubernetes
     Engine cluster.

...
```


You can apply a resource and crossplane will provision a GKE cluster in GCP.

```
kubectl apply -f manifests/sample-gcp/gke-cluster.yaml
```

Verify the resource creation by 
```
kubectl get managed
```

or 
```
kubectl get gkecluster,nodepool
```

Cleanup

```
kubectl delete -f manifests/sample-gcp/gke-cluster.yaml
```