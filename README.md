# Architectural Decision Log

This log lists the architectural decisions for hamlet. This log aims to document core architectural concepts and conventions that we use within hamlet.
The log is intended to cover conceptual ideas and conventions and should not include implementation or operational documentation

## The Log

The records below are accepted Design records which we follow as part of hamlet

<!-- adrlog -- Regenerate the content by using "adr-log -i". You can install it via "npm install -g adr-log" -->

* [ADR-0000](adr/0000-use-markdown-architectural-decision-records.md) - Use Markdown Architectural Decision Records
* [ADR-0003](adr/0003-provide-implementation-tracking-on-records.md) - Include Implementation Log as part of Decision Records
* [ADR-0004](adr/0004-provide-error-codes-on-handled-exception.md) - Provide Users an Error Code on Fatal Error

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

## Generating the Log

To Update the log above run the following commands from the root of the repo

```
npm install
npx adr-log
npx adr-log -i README.md -d adr/
```

This will update the ADR log entry above with any new ADRs which have been created
