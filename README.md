# crossplane-demo

A list of hands-on with crossplane.

### Crossplane in theory

**TL;DR**

Crossplane provides Declarative approach to infrastructure configuration and infrastructure management  that can be used to provision and manage any infrastructure on top of the K8s API.

#### Crossplane
Crossplane extends your Kubernetes cluster, providing you with CRDs for any infrastructure or managed service. Compose these granular resources into higher level abstractions that can be versioned, managed, deployed and consumed using your favorite tools and existing processes you've already integrated with your clusters.


#### Quick start

To get started with crossplane, you should install Crossplane controller first [install-crossplane](install-crossplane.md)


As quick start guide you can head over [sample-gcp](sample-gcp.md)

Yet, Another guide with helm provider [crossplane-helm-provider](crossplane-helm-provider.md)


#### Crossplane composite resources

**TL;DR**
The idea is to create custom resources with the composite resources (XRs), and the corresponding team could just identify a composite resource claim (XRC)
that will satisfy the definition of the XR.

This model has similarities to Persistent Volumes (PV) and Persistent Volume Claims (PVC) in Kubernetes.



Composite resources (XRs) are always cluster scoped 
it enables you to define new custom resources with schemas of your choosing. We call these “composite resources” (XRs). Composite resources compose managed resources – Kubernetes custom resources that offer a high fidelity representation of an infrastructure primitive, like an SQL instance or a firewall rule.

two special Crossplane resources to define and configure these new custom resources:

A `CompositeResourceDefinition` (XRD) defines a new kind of composite resource, including its schema. An XRD may optionally offer a claim (XRC).
A `Composition` specifies which resources a composite resource will be composed of, and how they should be configured. You can create multiple Composition options for each composite resource.


[A simple demo to composite](composition.md)



#### Crossplane configuration

`XRDs` and `Compositions` may be packaged and installed as a configuration. A configuration is a `package` of composition configuration that can easily be installed to Crossplane by creating a declarative `Configuration` resource, or by using `kubectl crossplane install configuration`.


TBD
