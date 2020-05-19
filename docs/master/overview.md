---
title: Overview
toc: true
weight: 400
indent: true
---

# Overview

Crossplane introduces multiple building blocks that enable you to provision,
publish, and consume infrastructure using the Kubernetes API. These individual
concepts work together to allow for powerful separation of concern between
different personas in an organization, meaning that each member of a team
interacts with Crossplane at an appropriate level of abstraction.

![Crossplane Concepts]

The diagram above illustrates a common workflow using most of Crossplane's
functionality.

An infrastructure operator...
1. Installs Crossplane and one or more [providers] (in this case
   [provider-azure]) as [packages]. This enables provisioning of external
   infrastructure from the Kubernetes cluster.
2. Creates an `InfrastructureDefinition` to define a new type of resource (in
   this case a `MySQLInstance`). Crossplane creates a cluster-scoped CRD of kind
   `MySQLInstance` in response.
3. Creates an `InfrastructurePublication` to make provisioning a `MySQLInstance`
   possible at the namespace scope. Crossplane creates a namespace-scoped CRD of
   kind `MySQLInstanceRequirement` in response.
4. Creates a `Composition` that instructs Crossplane how to render one or more
   infrastructure primitives installed by providers in response to the creation
   of a `MySQLInstance` or `MySQLInstanceRequirement`. In this case the
   `Composition` specifies that Azure `MySQLServer` and `MySQLFirewallRule`
   [managed resources] should be created.

An application developer...
1. Creates an [OAM] `Component` for their service that specifies that they wish
   to be run as an OAM `ContainerizedWorkload`.
2. Creates an OAM `Component` for their MySQL database that can be satisfied by
   the published `MySQLInstanceRequirement` type.

An application operator...
1. Creates an OAM `ApplicationConfiguration`, which is comprised of the two
   `Component` types that were defined by the application developer, and a
   `ManualScalerTrait` trait to modify the replicas in the
   `ContainerizedWorkload`. In response, Crossplane translates the OAM types
   into Kubernetes-native types, in this case a `Deployment` and `Service` for
   the `ContainerizedWorkload` component, and a `MySQLServer` and
   `MySQLFirewallRule` for the `MySQLInstanceRequirement` component.
2. Crossplane provisions the external infrastructure and makes the connection
   information available to the application, allowing it to connect to and
   consume the MySQL database on Azure.

The concepts used in this workflow are explained in greater detail below and in
their individual documentation.

## Packages

Packages allow Crossplane to be extended to include new functionality. This
typically looks like bundling a set of Kubernetes [CRDs] and [controllers] that
represent and manage external infrastructure (i.e. a provider), then installing
them into a cluster where Crossplane is running. Crossplane handles making sure
any new CRDs do not conflict with existing ones, as well as manages the RBAC and
security of new packages. Packages are not strictly required to be providers,
but it is the most common use-case for packages at this time.

## Providers

Providers are packages that enable Crossplane to provision infrastructure on an
external service. They bring CRDs (i.e. managed resources) that map one-to-one
to external infrastructure resources, as well as controllers to manage the
life-cycle of those resources. You can read more about providers, including how
to install and configure them, in the [providers documentation].

## Managed Resources

Managed resources are Kubernetes custom resources that represent infrastructure
primitives. Managed resources with an API version of `v1beta1` or higher support
every field that the cloud provider does for the given resource. You can find
the Managed Resources and their API specifications for each provider on
[doc.crds.dev] and learn more in the [managed resource documentation]. 

## Composition

Composition refers the machinery that allows you to bundle managed resources
into higher-level infrastructure units, using only the Kubernetes API. New
infrastructure units are defined using the `InfrastructureDefinition`,
`InfrastructurePublication`, and `Composition` types, which result in the
creation of new CRDs in a cluster. Creating instances of these new CRDs result
in the creation of one or more managed resources. You can learn more about all
of these concepts in the [composition documentation].

## OAM

Crossplane supports application management as the Kubernetes implementation of
the [Open Application Model]. As such, Crossplane currently implements the
following OAM API types as Kubernetes custom resources.

* `WorkloadDefinition`: defines the kind of components that an application
  developer can use in an application, along with the component's schema.
  * Crossplane also implements the core `ContainerizedWorkload` type.
    Infrastructure owners may define any resource as a workload type by
    referencing it in a `WorkloadDefinition`.
* `Component`: describe functional units that may be instantiated as part of a
  larger distributed application. For example, each micro-service in an
  application is described as a `Component`.
* `Trait`: a discretionary runtime overlay that augments a component workload
  type with additional features. It represents an opportunity for those in the
  application operator role to make specific decisions about the configuration
  of components, without having to involve the developer.
  * Crossplane also implements the core `ManualScalerTrait` type.
* `ApplicationConfiguration`: includes one or more component instances, each
  represented by a component definition that defines how an instance of a
  component spec should be deployed.

For more information, take a look at the [OAM documentation].

<!-- Named Links -->

[Crossplane Concepts]: crossplane-concepts.png
[providers]: #providers
[provider-azure]: https://github.com/crossplane/provider-azure
[packages]: #packages
[managed resources]: #managed-resources
[OAM]: #oam
[CRDs]: https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/
[controllers]: https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/#custom-controllers
[providers documentation]: providers.md
[doc.crds.dev]: https://doc.crds.dev
[managed resources documentation]: managed-resources.md
[composition documentation]: composition.md
[Open Application Model]: https://oam.dev/
[OAM documentation]: oam.md