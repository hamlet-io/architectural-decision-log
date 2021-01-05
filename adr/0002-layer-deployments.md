# Layer Level Deployments

* Status: proposed
* Deciders: roleyfoley
* Date: 2021-01-04

Technical Story: Support for deployments of shared resources across layers defined in hamlet

## Context and Problem Statement

We want a way to define and manage the deployment of resources which are shared across multiple solutions.
A number of services offered by cloud providers, namely AWS have services which can only be deployed once per AWS Account or enable a service across an entire account
How should we manage these resources to scale with our plugin support and to support the features we have introduced for our other deployments

## Decision Drivers

* Support required for multi region deployments to allow global image registry
* Deprecation of the fragment and case style template processing which is currently used for account level deployments

## Considered Options

* Add support for a layer level component like deployment option
* Refactor existing services to use a similar approach to extensions and apply extensions to layers


## Decision Outcome

Chosen option: "Add support for a layer level component like deployment option", because this allows for greater flexibility over the resources we can deploy and aligns with our other deployment processes

### Positive Consequences

* Layer level components work in a similar way to our other components
* We can use placements, providers and plugins to support custom layer deployments
* Extends the existing account level deployments to work across user defined layers and support


### Negative Consequences

* Additional complexity in introducing the state management for these components
* Processing requires additional steps including the creation of an occurrence like structure
* The concept for the layer deployments will be similar to occurrences but won't be exactly the same

## Pros and Cons of the Options

### Add support for a layer level component like deployment option

The idea is to introduce a layer based component structure, at the moment components are only available under tiers and we use the components to in turn create occurrences. This allows for users to define instances of components and to define specific types. This would introduce the idea of Foundations which are singleton types defined in a plugin and assigned to a defined layer. Foundations are configured as part of a layers configuration and would store its state based on the layer that it has been assigned to. This allows for the state of foundations to be used by multiple solutions. In this model the foundations for a layer would be defined by a plugin and remove the need for users to know which foundations create dependencies for other components.

So instead of a user being able to deploy multiple instances of a given component type each foundation type can only be deployed once, foundations can optionally offer to be disabled or enabled as part of their configuration

* Good, because it aligns with our existing concepts around distinct deployment units which are standalone deployments
* Good, because it is similar to own component style deployment process
* Good, because we can use standardised naming and state querying ( resources, attributes etc. )
* Bad, because it adds complexity in our processing and requires the implementation of a new occurrence structure ( one which supports singleton deployments )


### Refactor existing services to use a similar approach to extensions and apply extensions to layers

In this approach add extension support to layers, in this approach each layer would have a single deployment unit and the configured extensions on the layer would determine what is created for a given deployment. The state would then be provided in a similar way that we do now. In this approach the user would need to define the layers they require and have knowledge of required and optional deployment resources

* Good, because it is simple and simplifies the management of layer based deployments, i.e. add the layer level extensions you want and deploy
* Good, because it aligns with the current approach and would require less work to implement
* Bad, because it wouldn't gain the benefits of state sharing and would require global naming functions to support
* Bad, because it could create conflicting resources as part of a shared single deployment
