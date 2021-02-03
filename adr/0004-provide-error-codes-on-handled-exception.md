# [Provide Users an Error Code on Fatal Error]

* Status: proposed
* Deciders: tba
* Date: 2021-02-03

## Context and Problem Statement

Hamlet Deploy is not a compiled binary file nor is it written in a single programming or scripting language. It is a combination of all of the above; 
 * Java for the Freemarker wrapper, 
 * Freemarker has its own DSL, 
 * the original Hamlet Deploy Executor is presently primarily Bash scripts and 
 * the "Click" CLI Executor is written in Python. 
 
When a handled exception occurs, there is no consistency in the error information provided. A handled error in Freemarker is likely to provide a large JSON object for review, whilst an error in the Bash Executor will provide a brief summary of the problem. Because these summaries are not always unique-per-error, many provide only a generic error message. 


## Considered Options

* create a manually-maintained library of error codes & assign one to each handled error
* [for consideration in addition to the above] group like-errors by convention 

## Decision Outcome

Pending discussion

<!-- Chosen option: "[option 1]", because [justification. e.g., only option, which meets k.o. criterion decision driver | which resolves force force | … | comes out best (see below)]. -->

### Positive Consequences

* provide a method for additional documentation on individual error messages
  * which may provide an avenue to see which errors are encountered / searched most frequently
* improved reporting / communication around errors experienced
* improved clarity around what constitutes a "handled" and "unhandled" error message
* greater consistency between the different parts of Hamlet Deploy

### Negative Consequences

* increased maintenance overhead
