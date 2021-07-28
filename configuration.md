### Configuration

A Basic Configuration directory that contains:


`crossplane.yaml` - Metadata about the configuration. [this file should be at the root.]
`definition.yaml` - The XRD.
`composition.yaml` - The Composition.


Crossplane can create a configuration from any directory with a valid crossplane.yaml metadata file at its root, and one or more XRDs or Compositions.



#### Build a Configuration

Create a Directory

```
mkdir crossplane-config
cd crossplane-config
```

We will use the example in `manifests/composite` for demo purpose.

First we’ll create a `CompositeResourceDefinition` (XRD) to define the schema of our `CompositeOpendistro` and its `Opendistro` resource claim.

`crossplane-config/definition.yaml`

```
apiVersion: apiextensions.crossplane.io/v1
kind: CompositeResourceDefinition
metadata:
  name: compositeopendistroinstances.renault.fr
spec:
  group: renault.fr
  # The defined kind of composite resource.
  names:
    kind: CompositeOpendistro
    plural: compositeopendistroinstances

  claimNames:
    kind: Opendistro
    plural: opendistroinstances
 
  versions:
  - name: v1alpha1
    served: true
    referenceable: true
    # Define the spec of the Resource
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
                kibanaPort:
                  description: Kibana external Port
                  type: integer
                kibanaService:
                  type: string
                  enum: ["ClusterIP", "LoadBalancer"]
            required:
            - kibanaService

          # The status subresource can be optionally defined in the XRD
          # schema to allow observed fields from the composed resources
          # to be set in the composite resource and claim.
          status:
            type: object
            properties:
              state:
                description: If the Opendistro Ready or not
                type: string
              url:
                description: Internal URL of the Opendistro client
                type: string
```


Create Compositions

Now we’ll specify which managed resources our `CompositeOpendistro` XR and its claim could be composed of, and how they should be configured

The `Composition` defines the set of resources that can satisfy the XR we defined above.

In real wolrd scenario, You could define multiple `Compositions` per Provider: 
Ref [Create Compositions](https://crossplane.io/docs/v1.3/getting-started/create-configuration.html#create-compositions)


`crossplane-config/composition.yaml`

```
apiVersion: apiextensions.crossplane.io/v1
kind: Composition
metadata:
  name: opendistro-composition
  labels:
    purpose: example
    provider: helm
spec:
  # This Composition declares that it satisfies the CompositeMySQLInstance
  # resource defined above - i.e. it patches "from" a CompositeMySQLInstance.
  # Note that the version in apiVersion must be the referenceable version of the
  # XRD.
  compositeTypeRef:
    apiVersion: renault.fr/v1alpha1
    kind: CompositeOpendistro


  patchSets:
  - name: metadata
    patches:
    - fromFieldPath: metadata.labels


  # resources.
  resources:
     - base:
        apiVersion: helm.crossplane.io/v1beta1
        kind: Release
        metadata:
          name: opendistro
        spec:
          forProvider:
            chart:
              name: opendistro-es
              repository: https://mcbenjemaa.github.io/opendistro-build
              version: 1.13.3
            namespace: default
            values:
              kibana:
                externalPort: 443
                service:
                  type: ClusterIP

          providerConfigRef:
            name: helm-provider
       patches:
        - fromFieldPath: spec.kibanaService
          toFieldPath: spec.forProvider.values.kibana.service.type
        - fromFieldPath: spec.kibanaPort
          toFieldPath: spec.forProvider.values.kibana.externalPort
```


Finally, we’ll author our metadata file then build and push our configuration so that Crossplane users may install it.

```
kubectl crossplane build configuration
```

You should see a file in your working directory with a `.xpkg` extension.  `crossplane-demo-simple-configuration*.xpkg` 


You can push the configuration using kubectl crossplane

```
REG=my-package-repo
kubectl crossplane push configuration ${REG}/getting-started-with-gcp:master
```


Now, you are ready to install the configuration:


```
kubectl crossplane install configuration ${REG}/crossplane-simple-config:v1
```

and you should be able to consume the XRD

```
kubectl apply -f manifests/composite/opendsitro.yaml
```