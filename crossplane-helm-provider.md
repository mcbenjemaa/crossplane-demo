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

Apply to Mgmt cluster
``` 
 k apply -f manifests/provider-helm/provider-config-external.yaml
```

If you want to manage helm releases on the same cluster.
you need to execute 

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



The `Release` resource is great and flexible for extending and defining multiple sources for helm values

Example:
```
apiVersion: helm.crossplane.io/v1alpha1
kind: Release
metadata:
  name: wordpress-example
spec:
  forProvider:
    chart:
      name: wordpress
      repository: https://charts.bitnami.com/bitnami
      version: 9.3.19
    namespace: wordpress
    values:
      mariadb:
        enabled: false
      externaldb:
        enabled: true
    valuesFrom:
    - configMapKeyRef:
        name: wordpress-defaults
        namespace: prod
        key: values.yaml
        optional: false
    set:
    - name: wordpressBlogName
      value: "Hello Crossplane"
    - name: externalDatabase.host
      valueFrom:
        secretKeyRef:
          name: dbconn
          key: host
    - name: externalDatabase.user
      valueFrom:
        secretKeyRef:
          name: dbconn
          key: username
    - name: externalDatabase.password
      valueFrom:
        secretKeyRef:
          name: dbconn
          key: password
    patchesFrom:
    - configMapKeyRef:
        name: labels
        namespace: prod
        key: patches.yaml
        optional: false
    - configMapKeyRef:
        name: wordpress-nodeselector
        namespace: prod
        key: patches.yaml
        optional: false
    - secretKeyRef:
        name: image-pull-secret-patch
        namespace: prod
        key: patches.yaml
        optional: false
  providerConfigRef: 
    name: cluster-1-provider
  reclaimPolicy: Delete
```

##### Value Overrides
There are multiple ways to provide value overrides and final values will be composed with the following precedence:

* `spec.forProvider.valuesFrom` array, items with increasing precedence
* `spec.forProvider.values`
* `spec.forProvider.set` array, items with increasing precedence


##### Post Rendering Patches
It will be possible to provide post rendering patches which will make last mile configurations using post rendering option of Helm. `spec.forProvider.patchesFrom` array will be used to specify patch definitions satisfying kustomizes patchTransformer interface.

Example:

``` 
patches:
- patch: |-
    - op: replace
      path: /some/existing/path
      value: new value
      target:
      kind: MyKind
      labelSelector: "env=dev"
- patch: |-
    - op: add
      path: /spec/template/spec/nodeSelector
      value:
      node.size: really-big
      aws.az: us-west-2a
      target:
      kind: Deployment
      labelSelector: "env=dev"
```

Release Resource Description:
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