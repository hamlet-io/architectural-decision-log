# Architectural Decision Log

This log lists the architectural decisions for hamlet. This log aims to document core architectural concepts and conventions that we use within hamlet.
The log is intended to cover conceptual ideas and conventions and should not include implementation or operational documentation.

## The Log

The records below are accepted Design records which we follow as part of hamlet

<!-- adrlog -- Regenerate the content by using "adr-log -i". You can install it via "npm install -g adr-log" -->

* [ADR-0000](adr/0000-use-markdown-architectural-decision-records.md) - Use Markdown Architectural Decision Records
* [ADR-0001](adr/0001-executor-engine-separation.md) - Executor Engine Separation
* [ADR-0002](adr/0002-layer-deployments.md) - Layer Level Deployments
* [ADR-0003](adr/0003-provide-implementation-tracking-on-records.md) - Include Implementation Log as part of Decision Records
* [ADR-0004](adr/0004-provide-error-codes-on-handled-exception.md) - [Provide Users an Error Code on Fatal Error]
* [ADR-0005](adr/0005-convention-based-logging-classes.md) - [Convention-based Logging Classes]
* [ADR-0006](adr/0006-hamlet-docker-layer-release-mgmt.md) - Hamlet Engine distribution through docker image layers
* [ADR-0007](adr/0007-placement-support.md) - Placement Support

<!-- adrlogstop -->

For new ADRs, please use [template.md](template.md) as basis.
More information on MADR is available at <https://adr.github.io/madr/>.
General information about architectural decision records is available at <https://adr.github.io/>.

## Creating an ADR

1. Copy the [template.md](template.md) file into the `adr/` directory
1. Rename the file to NNNN-title-with-dashes.md, where NNNN indicates the next number in sequence and title is the tile of the ADR based on the description in the template
1. Update the template content to outline the decision you want to define
1. [Update the Log](#generating-the-log)
1. Raise a PR with your ADR to this repo and add the engine-maintainers as reviewers
1. Once approved, update the approval details at the start of the ADR then merge the PR

## Tracking implementation progress once the ADR is approved

1. Raise an issue using the [Implementation Log](https://github.com/hamlet-io/architectural-decision-log/issues/new/choose) issue template
1. Link any PRs raised during implementation to this issue
1. Refine the ADR where necessary with any details that arise during implementation
1. Close the issue when the maintainers are satisfied that the ADR has been implemented based on its accepted record 
## Generating the Log

To Update the log above run the following commands from the root of the repo

```
npm install
npx adr-log -i README.md -d adr/
```

This will update the ADR log entry above with any new ADRs which have been created
