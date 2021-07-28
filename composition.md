### Composition

API Reference: https://doc.crds.dev/github.com/crossplane/crossplane 


### Create the definition 

Ref: https://crossplane.io/docs/v1.3/concepts/composition.html

Create an XRD resource that will be used to establish new Custom Resouce.

my example create an definition for opendistro.

```
kubectl apply -f manifests/composite/opendsitro.yaml
```

```
kubectl get xrd

NAME                                      ESTABLISHED   OFFERED   AGE
compositeopendistroinstances.renault.fr   True          True      4m46s
```


### Create a Composition 

A Composition resource used to compose multiple resources to form the final desired state,

For example you can compose multiple cloud resources together, e.g. GKE, CloudSQL, Firewall etc..

for demo purpose, 
I created a simple Composition that has only one resource.

```
kubectl apply -f manifests/composite/composition.yaml
```

```
kubectl get composition
```

### Create The claim

Now, as a Consumer you can define your own resource

```
kubectl apply -f manifests/composite/opendsitro.yaml
```

```
k get opendistroinstances.renault.fr

NAME                     READY   CONNECTION-SECRET   AGE
my-opendistro-instance   True                        2m1s
```

Crossplane will spinup the requested resources specified in the composition 

to check all managed resources by crossplane

```
kubectl get managed
```


If you update the resource, crossplane will handle the change for you.


Change something and apply

```
kubectl apply -f manifests/composite/opendsitro.yaml
```


```
kubectl get managed
```