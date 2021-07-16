### Crossplane helm provider

If you don't have crossplane installed yet, then head over to the installation guide [install-crossplane](install-crossplane.md)

#### Install Helm provider
Install


```
kubectl crossplane install provider crossplane/provider-helm:master
```

You may also manually install provider-helm by creating a Provider directly:

```
apiVersion: pkg.crossplane.io/v1alpha1
kind: Provider
metadata:
  name: provider-helm
spec:
  package: "crossplane/provider-helm:master"

```

Check status

```
kubectl get provider
```


###### Usage Helm

Helm provider is meant to deploy Helm releases on `Workload clusters` (The clusters that are being created and managed by crossplane).

For instance, when you apply a `Release` resource to the `Management cluster` (The cluster that manages all the infrastructure)
The Release will be deployed to the `Workload cluster`,

Helm provider uses a custom resource `ProviderConfig` to identify the workload cluster, by the secret that has the underlying kubeconfig credentials.
(These secrets are being created automatically when you provision the clusters using crossplane)
Example:


If working on external cluster:
Create a secret of external cluster and use it in Mgmt cluster

```
kubectl --context <mgmt-cluster> create secret generic cluster-config --from-literal=kubeconfig="$(kubectl --context <workload-cluster> config view --minify --raw)"
```

Create `ProviderConfig`

```
apiVersion: helm.crossplane.io/v1beta1
kind: ProviderConfig
metadata:
  name: helm-provider
spec:
  credentials:
    source: Secret
    secretRef:
      name: cluster-config
      namespace: default
      key: kubeconfig
```

Apply to cluster Mgmt cluster
``` 
 k apply -f manifests/provider-helm/provider-config-external.yaml
```

If you want to manage helm releases on the same cluster.
you need to apply 

```
SA=$(kubectl -n crossplane-system get sa -o name | grep provider-helm | sed -e 's|serviceaccount\/|crossplane-system:|g')
kubectl create clusterrolebinding provider-helm-admin-binding --clusterrole cluster-admin --serviceaccount="${SA}"
kubectl apply -f manifests/provider-helm/provider-config.yaml
```

Create a `Release` resource

```
kubectl apply -f manifests/provider-helm/release.yaml
```


Workflow for external cluster:
```
In cluster B:

Install provider-helm
Create a Kubernetes secret containing kubeconfig of the cluster A
Create a ProviderConfig for provider helm pointing the secret you created in previous step
Create a HelmRelease with providerConfigRef to the ProviderConfig you created in previous step.
Provider helm will go and deploy the helm release into cluster A.
```



Release Resource:
```
KIND:     Release
VERSION:  helm.crossplane.io/v1beta1

DESCRIPTION:
     A Release is an example API type

FIELDS:
   apiVersion	<string>
   kind	<string>
   metadata	<Object>
      annotations	<map[string]string>
      clusterName	<string>
      creationTimestamp	<string>
      deletionGracePeriodSeconds	<integer>
      deletionTimestamp	<string>
      finalizers	<[]string>
      generateName	<string>
      generation	<integer>
      labels	<map[string]string>
      managedFields	<[]Object>
         apiVersion	<string>
         fieldsType	<string>
         fieldsV1	<map[string]>
         manager	<string>
         operation	<string>
         time	<string>
      name	<string>
      namespace	<string>
      ownerReferences	<[]Object>
         apiVersion	<string>
         blockOwnerDeletion	<boolean>
         controller	<boolean>
         kind	<string>
         name	<string>
         uid	<string>
      resourceVersion	<string>
      selfLink	<string>
      uid	<string>
   spec	<Object>
      connectionDetails	<[]Object>
         apiVersion	<string>
         fieldPath	<string>
         kind	<string>
         name	<string>
         namespace	<string>
         resourceVersion	<string>
         toConnectionSecretKey	<string>
         uid	<string>
      deletionPolicy	<string>
      forProvider	<Object>
         chart	<Object>
            name	<string>
            pullSecretRef	<Object>
               name	<string>
               namespace	<string>
            repository	<string>
            url	<string>
            version	<string>
         namespace	<string>
         patchesFrom	<[]Object>
            configMapKeyRef	<Object>
               key	<string>
               name	<string>
               namespace	<string>
               optional	<boolean>
            secretKeyRef	<Object>
               key	<string>
               name	<string>
               namespace	<string>
               optional	<boolean>
         set	<[]Object>
            name	<string>
            value	<string>
            valueFrom	<Object>
               configMapKeyRef	<Object>
                  key	<string>
                  name	<string>
                  namespace	<string>
                  optional	<boolean>
               secretKeyRef	<Object>
                  key	<string>
                  name	<string>
                  namespace	<string>
                  optional	<boolean>
         skipCreateNamespace	<boolean>
         values	<>
         valuesFrom	<[]Object>
            configMapKeyRef	<Object>
               key	<string>
               name	<string>
               namespace	<string>
               optional	<boolean>
            secretKeyRef	<Object>
               key	<string>
               name	<string>
               namespace	<string>
               optional	<boolean>
         wait	<boolean>
      providerConfigRef	<Object>
         name	<string>
      providerRef	<Object>
         name	<string>
      rollbackLimit	<integer>
      writeConnectionSecretToRef	<Object>
         name	<string>
         namespace	<string>
   status	<Object>
      atProvider	<Object>
         releaseDescription	<string>
         revision	<integer>
         state	<string>
      conditions	<[]Object>
         lastTransitionTime	<string>
         message	<string>
         reason	<string>
         status	<string>
         type	<string>
      failed	<integer>
      patchesSha	<string>
      synced	<boolean>
 ```     