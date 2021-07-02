# Placement Support

* Status: proposed
* Deciders: [list everyone involved in the decision] <!-- optional -->
* Date: 2021-06-18

## Context and Problem Statement

For some time hamlet has had the idea of a `placement` - a mechanism to dynamically determine where resources should be hosted.

Placements are currently part of the hierarchy of the state tree, and the resource groups within the occurrence structure were intended to allow groups of resources to be placed into different provider accounts.

However to date, a number of limitations and inefficiencies stemming from hamlet's original design has prevented full exploitation of the placement concept;
- the assembly of the solution before the engine is run
- the focus of a template run on a single segment
- the generation of account templates uses different concepts to those of product templates
- the maintenance of critical configuration information such as environment to account mappings external to the cmdb.

The introduction of dynamic cmdb loading and the input pipeline processor with full cmdb qualification presents an opportunity to address these long standing issues and complete the implementation of placement support. The question is how.

## Decision Drivers <!-- optional -->

* reuse existing concepts and ways of doing things as much as possible
* given a cmdb, no other configuration information should be necessary to deploy a product

## Considered Options

This ADR extends on the decision made in [ADR-0002](../0002-layer-deployments.md) - Layer Level Deployments and provides more design detail, primarily by the enrichment of link semantics.

## Decision Outcome

Chosen option: "Extend link semantics", because it addresses the decision drivers best (see below) and aligns with the decision previously made.

### Positive Consequences <!-- optional -->

* reduced configuration complexity - everything in one place
* better validation of configurations
* links have proven very flexible so expect the proposed changes to be re-purposed for other things in the future
* straightforward transition process once infrastructure in place

### Negative Consequences <!-- optional -->

* Expansion of semantics of existing concepts may be harder to document, understand and explain than dedicated new concepts

## Design Detail

### New Link Semantics

Currently , a link goes from one occurrence in a solution to another _within_ the same solution solution.

This option builds on existing link processing by adding support for a link to target an occurrence in _another_ solution as follows;

1. A link targets an occurrence within a solution.
1. A link consists of two parts;
- those attributes that identify a solution within a `District`
- those attributes that identify an occurrence within the solution.
1. Occurrences within a solution are identified by a standard set of link attributes
     - `Tier`
     - `Component`
     - `Subcomponent`
     - `Instance`
     - `Version`

   Thus these attributes must be present on a link, or be inherited from the occurrence on which the link is configured.
1. The `Subcomponent` attribute replaces use of type specific attributes such as `Port`.
1. An optional `Type` attribute is added to ensure links to same named subcomponents can be resolved. Its value is the desired component type of the link target. A side effect of this for any link, the `Type` attribute can be added to document an expectation of the type of the intended target allowing misconfigured links to be identified in some cases.
1. Districts will be configured via a `districts` plugin directory, with each directory having an `id.ftl` file to define the district.
1. Each `District` has a name and defines an ordered set of layers used to identify solutions within the `District`. In turn, the input filter attribute associated with each layer is used as the attribute in the link used to identify the layer within a district.
1. In order to resolve a link, the desired district must be known, and a value for each of the layers identifying solutions in the distict must be known.
1. To identify the district, a link must have a `District` attribute whose value matches the name of one of the known districts. It has a default of `self` which means the link will use the district associated with the solution from which the link originates.
1. As an initial implementation, the following districts will be defined in the shared provider with the indicated layer ordering;
    - `segment` = `Product`, `Environment`, `Segment`
    - `account` = `Account`
    - `tenant` = `Tenant`
1. In the same way that a link can inherit occurrence identifiers from the source component, it inherits layer identifiers from the source solution.
1. Link indirection is supported by an optional `LinkRef` attribute within a link object. If present, no other link attributes should be explicitly provided.
1. The value of a LinkRef attribute is used as the attribute name in a `LinkRefs` configuration object, with the attribute value
being the desired link definition.
1. LinkRef attributes provide a convenient mechanism to centralise qualification of links shared across multiple components, as is commonly the case with placements.

### New Components

This option also introduces two new components - `Subscription` and `HostingPlatform`.

A `Subscription` component represents provider specific mechanism for purchasing hosting capability, such as an `account` with AWS or a `subscription` with Azure.

Subscription components will typically be used with a `tenant` district solution.

A `HostingPlatform` component represents a place within a subscription where resources can physically be deployed.

HostingPlatform components will typically be used within an `account` district solution, optionally linked to the `Subscription` component with which they are associated. Where the subscription is externally created, the HostingPlatform explicitly carries the provider information for the subscription. Mixed usage are expected to be common, e.g. a master account for AWS is created externally with all subsequent accounts then being created via Subscription components.

As an example of usage, for AWS or Azure it would be normal to see this component in each `account` district solution, with instances for each region that is active in the account. In the initial implementation, this component would not have resources of its own but would act somewhat like the external component and simply provide key attributes (like providerId and region in the case of AWS or Azure). However it is also likely that other components within an `account` district solution, such as registries, would also link to this component in order to establish where account solution resources should be deployed. So when deploying an S3 based registry, a region would need to be provided as a command line option to select the desired template to generate, with the HostingPlatform then being used to determine if a given registry should be included (see below on ResourceGroup placement).

### Locations Occurrence Attribute

All components will now support a `Locations` attribute, the purpose of which is to provide the mechanism for a component to document its requirements for information related to the placement of other components.

Each location is represented as a key within the Locations attribute, with eack key having a **mandatory** `Link` attribute.

Locations will be included in the metadata when defining a component type, and will specify if the link is mandatory or optional, and what component types can be targetted by the link. This will permit Location information to be checked for completeness.

If a location entry for the desired location can't be found, processing will check for a location of `_default` and apply the other checks for the desired location to this location. This processing will not apply to mandatory links.

Being an occurrence attribute, the Locations attribute can be populated directly on a component for maximum granularity, but equally be configured via `DeploymentProfiles`, and enforced via `PolicyProfiles`. These also give the ability for the Locations be be varied based on component type, for example to select different registries.

In general, it is expected that a LinkRef will be used for location links
to centralise any qualifications, for example differentiation between non-production and production environment placements.

### ResourceGroup placement
Placement of a resource group will be done by way of a location with the same name as the resource group, which **must** link to a `HostingPlatform` occurrence. The occurrence will be expected to offer `PROVIDER_ID`, `REGION` and `DEPLOYMENT_FRAMEWORK` attributes.

A transition from the existing, single `default` resource group arrangement can easily be accommodated by defining the `_default` location using a LinkRef that switches HostingPlatform on the basis of environment.

### Other Location Uses

Over time, other uses of locations are expected, such as the selection of registries from which code should be sourced.

The current baseline and network processing could also be merged into location processing, perhaps with predefined values of `_baseline` and `_network`. A component requiring these would still need to declare them for validation and documentation purposes.

### Assessment

* Good, because it reuses existing concepts (layers, districts, solutions)
* Good, because it improves documentation of dependency requirements between components
* Good, because placement information is contained within the CMDB
* Bad, because a degree of expertise in deployment profiles (a more advanced fature) is required for more advanced use cases.
