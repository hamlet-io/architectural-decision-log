# Hamlet Engine distribution through docker image layers

* Status: proposed
* Deciders: roleyfoley
* Date: 2020-21-04

## Context and Problem Statement

The hamlet engine is made up of a collection of software components which are based on a number of different platforms ( bash, freemarker, java ).
Our current distribution process collects these components into a multipurpose docker image that can handle most build and deploy tasks.
The collection process uses git to pull down the code from the software components and includes this in the image

The problem with this process is that it really only works in a controlled docker environment, making this image multipurpose has resulted in a large docker image and installing other versions of hamlet requires pulling a new image.

## Decision Drivers

* Having a simple installation process for user local installations or integration in existing processes
* Simple version management support with a focus on handling the usage of features under active development within deployment pipelines
* A tested and verified release process

## Considered Options

* Docker image layer as component artefacts with support for installing these images
* Include all software components in the hamlet-cli python package

## Decision Outcome

Chosen option: "[option 1]", because [justification. e.g., only option, which meets k.o. criterion decision driver | which resolves force force | … | comes out best (see below)].

### Positive Consequences <!-- optional -->

* [e.g., improvement of quality attribute satisfaction, follow-up decisions required, …]
* …

### Negative Consequences <!-- optional -->

* [e.g., compromising quality attribute, follow-up decisions required, …]
* …

## Pros and Cons of the Options

### Docker image layer as component artefacts with support for installing these images

In this process we would create docker images that contain the contents of the repository and would be based on the "scratch" image

I.e.

```Dockerfile
FROM scratch
COPY ./ /
```

This would essentially create a single layer image that just contains the files in the repository. This docker image would be tagged and pushed to a docker repository
The docker manifest lists the layers that are used in the docker image and the layers are tag.gz files when stored by the registry.

Pulling this layer down essentially provides an artefact that can be tagged and treated like docker images as required.

We would add processes to the executors which would pull the appropriate images for the components locally, store them locally based on their version and use a shim based approach for setting the active version of the components

This could be incorporated into our exiting bootstrap process to enable the bash executor to use the process.

We could also add a consolidated layer of the components as the primary release artefact, this would be a single fully tested artefact based on all of the components combined at a given point.

* Good, because docker images are agnostic from their software platform and can handle storing the different file types we want
* Good, because the images can be retrieved without specific dependencies on other package managers
* Good, because we can have both immutable and mutable releases, dev, nightly, stable and also have tagged releases
* Good, because the layers can integrate with our existing docker images as required using the `COPY --from` syntax
* Bad, because it is an uncommon pattern and does require a depth of docker image knowledge to maintain
* Bad, because it requires executor specific implementation of the pull process

### Include all software components in the hamlet-cli python package

We currently have a published pypi package for the hamlet-cli which just publishes the hamlet cli and doesn't include the underlying software components that are required for the cli to do actually do anything
We could extend the package build process to include the underlying software components inside the package so they are available on installation

* Good, because this would provide a single command to install the cli and all the components
* Good, because it uses a standard known installation method ( pip ) for installation processes
* Bad, because it wouldn't allow for different version of the engine components independent from the cli to be used. They could be done through virtual envs but would require publishing packages on each push to a component
* Bad, because it doesn't accomodate the idea of different executors running the engine. This would need to be implemented in each executor
