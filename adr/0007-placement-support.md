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
* links have proven very flexible so expect the porposed changes to be repurposed for other things in the future
* straightforward transition process once infrastructure in place

### Negative Consequences <!-- optional -->

* Expansion of semantics of existing concepts may be harder to document, understand and explain than dedicated new concepts

## Design Detail

### New Link Semantics

Currently links are used extensively in product solutions. A link goes from an occurrence in one solution to an occurrence in another (possibly same) solution. This option builds on existing link processing as follows;

1. A link targets an occurrence within a solution.
1. A link uses attributes to identify a solution and the specific occurrence within that solution.
1. Occurrences within a solution are identified by a standard set of link attributes
     - `Tier`
     - `Component`
     - `Subcomponent`
     - `Instance`
     - `Version`

   Thus these attributes must be present on a link, or be inherited from the occurrence on which the link is configured.
1. The `Subcomponent` attribute replaces use of type specific attributes such as `Port`.
1. An optional `Type` attribute is added to ensure links to same named subcomponents can be resolved. Its value is the component type of the link target.
1. Each solution has a scope and an identifier within the scope.
1. A link must have a `Scope` attribute representing the solution scope. The default, representing existing usage, is the `segment` scope.
1. The scope value determines what other ordered attributes of the link are used to identify the specific solution required.
1. The attributes to identify the particular solution must either be present on the link, or inherited from the occurrence on which
the link is configured.
1. For the `segment` scope, the ordered attributes used to identify a specific solution are `Tenant`, `Product`, `Environment` and `Segment`.
1. Link indirection is supported by an optional `LinkRef` attribute within a link object. If present, no other link attributes should be explicitly provided.
1. The value of a LinkRef attribute is used as the attribute name in a `LinkRefs` configuration object, with the attribute value
being the desired link definition.
1. LinkRef attributes provide a convenient mechanism to centralise qualification of links shared across multiple components, as is commonly the case with placements.

### New Components

This option also introduces two new components - `Subscription` and `HostingPlatform`.

A `Subscription` component represents provider specific mechanism for purchasing hosting capability, such as an `account` with AWS or a `subscription` with Azure.

Subscription components will typically be used with a `tenant` scoped solution. Instances of a tenant scoped solution will be identified by the `Tenant` link attribute.

A `HostingPlatform` component represents the place within a subscription where resources can physically be hosted. An example for AWS or Azure would be a region within a `Subscription`.

HostingPlatform components will typically be used within an `account` scoped solution, optionally linked to the Subscription component with which they are associated. Where the subscription is externally created, the HostingPlatform explicitly carries the provider information for the subscription. Mixed usage are expected to be common, e.g. a master account for AWS is created externally with all subsequent accounts then being created via Subscription components.

Instances of an account scoped solution will be identified by the `Tenant` and `Account` link attributes. They will typically contain an occurrence of the HostingPlatform component for each region in which resources are required.

### Locations Occurrence Attribute

All components will now support a `Locations` attribute, the purpose of which is to provide the mechanism for a component to document its
requirements for information related to the placement of other components.

Each location is represented as a key within the Locations attribute, with eack key having a **mandatory** `Link` attribute.

Locations will be included in the metadata when defining a component type, and will specify if the link is mandatory or optional, and what component types can be targetted by the link. This will permit Location information to be checked for completeness.

If a location entry for the desired location can't be found, processing will check for a location of `_default` and apply the other checks for the desired location to this location. This processing will not apply to mandatory links.

Being an occurrence attribute, the Locations attribute can be populated directly on a component for maximum granularity, but equally be configured via `DeploymentProfiles`, and enforced via `PolicyProfiles`. These also give the ability for the Locations be be varied based on component type, for example to select different registries.

In general, it is expected that a LinkRef will be used for location links
to centralise any qualifications, for example differentiation between non-production and production environment placements.

### ResourceGroup placement
Placement of a resource group will be done by way of a location with the same name as the resource group, which **must** link to a HostingPlatform occurrence.

A transition from the existing, single `default` resource group arrangement can easily be accommodated by defining the `_default` location using a LinkRef that switches HostingPlatform on the basis of environment.

In turn, the HostingPlatform will provide the provider subscription id, region id and DeploymentFramework.

### Other Location Uses

Over time, other uses of locations are expected, such as the selection of registries from which code should be sourced.

The current baseline and network processing could also be merged into location processing, perhaps with predefined values of `_baseline` and `_network`. A component requiring these would still need to declare them for validation and documentation purposes.

* Good, because it reuses existing concepts
* Good, because it improves documentation of dependency requirements between components
* Good, becuase account and placement information is contined within the CMDB
* Bad, because a degree of expertise in deployment profiles (a more advanced fature) is required.

