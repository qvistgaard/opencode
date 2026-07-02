---
name: antora-docs
description: Use when writing Antora module documentation for Java or Go projects in a FreeBSD Handbook-inspired technical style, especially pages, nav.adoc, antora.yml, procedures, concepts, and reference docs.
---

# FreeBSD-Style Antora Documentation

Use this skill to write Antora module documentation for Java or Go projects with a FreeBSD Handbook-inspired style: practical, accurate, task-oriented, and calm.

## Style Model

Write like FreeBSD Handbook documentation:

- Prefer direct, instructional prose over marketing language.
- Explain what feature does, when to use it, and what command or code changes are required.
- Start chapters or pages with a short purpose statement.
- Use a `Synopsis` section for longer pages.
- Include prerequisites before procedures when setup or background matters.
- Use "Read this page to learn:" followed by bullets for long tutorials.
- Move from concept to procedure to verification.
- Use active voice where clear: "Run this command", "Add this dependency".
- Use passive voice only when actor is irrelevant: "The service is started by...".
- Define terms before using them heavily.
- Keep paragraphs short, usually one to three sentences.
- Avoid hype, humor, sales copy, rhetorical questions, and vague claims.
- Avoid unexplained abbreviations.
- Prefer concrete nouns over pronouns when pronouns could be unclear.
- Tell reader what output or behavior to expect after each major step.
- Link to authoritative reference docs for deeper API, CLI, or platform detail.

## Tone

Use calm expert tone:

- Precise, neutral, and helpful.
- Assumes capable reader, not expert in this project.
- Warns clearly about risk without dramatizing.
- Uses "must", "should", and "can" with exact meanings.
- Does not say "easy", "simple", "just", or "obviously".

Modal meanings:

- `must`: required for correctness, safety, or compatibility.
- `should`: recommended default with valid exceptions.
- `can`: optional capability.
- `may`: permission or uncertain external behavior; avoid where `can` is clearer.

## Antora Structure

Prefer standard Antora module layout:

```text
docs/
  antora.yml
  modules/
    ROOT/
      nav.adoc
      pages/
        index.adoc
        getting-started.adoc
        configuration.adoc
        operations.adoc
        reference.adoc
      examples/
      partials/
      images/
```

Use page structure like this for substantial pages:

```asciidoc
= Page Title
:page-description: One sentence describing page purpose.
:keywords: java, go, service, configuration

[[page-slug-synopsis]]
== Synopsis

This page explains ...

Read this page to learn:

* How to ...
* How to ...
* How to verify ...

[[page-slug-prerequisites]]
== Prerequisites

Before using this page:

* Install ...
* Understand ...

[[page-slug-main-task]]
== Main Task

Procedure text.
```

For short reference pages, omit `Synopsis` when title plus opening paragraph is enough.

## AsciiDoc Rules

Use Antora-compatible AsciiDoc:

- Page title uses one `= Title`.
- Major sections use `==`, subsections use `===`.
- Add explicit anchors for stable inbound links: `[[module-topic-section]]`.
- Use `xref:page.adoc[]` for same-module links.
- Use `xref:module:page.adoc[]` for cross-module links.
- Use `xref:component:module:page.adoc[]` for cross-component links.
- Use `link:https://example.com[Label]` for external links.
- Use unordered lists for choices and learned outcomes.
- Use ordered lists for procedures.
- Use description lists for terms and configuration keys.
- Use tables only when comparison or lookup is clearer than prose.
- Use callouts for code explanations when code has multiple important lines.
- Use admonitions sparingly: `[NOTE]`, `[TIP]`, `[IMPORTANT]`, `[CAUTION]`, `[WARNING]`.

Admonition meanings:

- `NOTE`: useful context that does not change procedure.
- `TIP`: optional improvement or shortcut.
- `IMPORTANT`: required condition that prevents failure.
- `CAUTION`: data loss, downtime, compatibility break, or costly mistake possible.
- `WARNING`: security, irreversible destructive action, or production outage risk.

## Code And Commands

Use fenced AsciiDoc source blocks:

```asciidoc
[source,shell]
----
$ command --flag value
----
```

Prompt convention:

- `$` for normal user commands.
- `#` for commands requiring root or administrator privileges.
- No prompt when command is meant for copy-paste in automation.

Use language tags:

- `[source,java]` for Java.
- `[source,go]` for Go.
- `[source,yaml]` for YAML.
- `[source,toml]` for TOML.
- `[source,json]` for JSON.
- `[source,shell]` for shell commands.
- `[source,text]` or `[.programlisting]` for output.

After commands, state expected result:

```asciidoc
The output should include the service name and a `RUNNING` status.
```

## Java Documentation Patterns

For Java projects, cover:

- Supported JDK versions.
- Build tool: Maven, Gradle, or both.
- Dependency coordinates.
- Main configuration keys and defaults.
- Startup lifecycle.
- Threading and resource ownership.
- Error handling and exceptions.
- Logging categories.
- Metrics and health checks.
- Testing with unit tests and integration tests.
- Packaging: JAR, container image, native image if relevant.

Prefer examples that compile:

```asciidoc
[source,java]
----
public final class Example {
    public static void main(String[] args) {
        Service service = Service.create();
        service.start();
    }
}
----
```

Document Maven dependencies like this:

```asciidoc
[source,xml]
----
<dependency>
  <groupId>com.example</groupId>
  <artifactId>example-client</artifactId>
  <version>1.0.0</version>
</dependency>
----
```

Document Gradle dependencies like this:

```asciidoc
[source,kotlin]
----
implementation("com.example:example-client:1.0.0")
----
```

## Go Documentation Patterns

For Go projects, cover:

- Supported Go versions.
- Module path.
- Installation with `go get` or `go install`.
- Import paths.
- Context cancellation behavior.
- Error return semantics.
- Goroutine ownership and shutdown.
- Configuration sources.
- Logging and metrics.
- Testing with `go test`.
- CLI usage if project provides commands.

Prefer examples that compile:

```asciidoc
[source,go]
----
package main

import (
    "context"
    "log"

    "example.com/project/client"
)

func main() {
    c := client.New()
    if err := c.Start(context.Background()); err != nil {
        log.Fatal(err)
    }
}
----
```

Document module installation like this:

```asciidoc
[source,shell]
----
$ go get example.com/project@latest
----
```

## Page Types

### Overview Page

Use for `index.adoc`:

- What project is.
- Main problems it solves.
- Main components.
- Supported environments.
- Links to getting started, configuration, operations, and reference pages.

### Getting Started Page

Use this order:

1. State goal.
2. List prerequisites.
3. Install dependency or binary.
4. Create minimal configuration.
5. Run minimal example.
6. Verify result.
7. Link to next topics.

### Configuration Page

Use description lists or tables:

```asciidoc
`server.port`::
TCP port used by the HTTP server. Default: `8080`.

`server.host`::
Address to bind. Default: `127.0.0.1`.
```

For each option, document:

- Name.
- Type.
- Default.
- Required or optional.
- Valid values.
- Runtime reload behavior.
- Example.

### Operations Page

Cover:

- Startup and shutdown.
- Health checks.
- Metrics.
- Logs.
- Backups or state handling when relevant.
- Upgrade procedure.
- Troubleshooting.

### Reference Page

Use stable structure:

- API surface or CLI commands.
- Parameters.
- Return values or output.
- Errors.
- Examples.
- Compatibility notes.

## nav.adoc Pattern

Use clear hierarchy and short titles:

```asciidoc
* xref:index.adoc[]
* xref:getting-started.adoc[]
* xref:configuration.adoc[]
* Operations
** xref:operations.adoc[]
** xref:troubleshooting.adoc[]
* Reference
** xref:api-reference.adoc[]
** xref:cli-reference.adoc[]
```

## antora.yml Pattern

Use minimal component metadata:

```yaml
name: project-name
title: Project Name
version: '1.0'
start_page: ROOT:index.adoc
nav:
  - modules/ROOT/nav.adoc
```

## Rewrite Checklist

Before finishing documentation:

- Opening paragraph says what page is for.
- Reader can tell whether page applies to Java, Go, or both.
- Prerequisites are explicit.
- Commands include expected privilege level.
- Examples compile or are marked as illustrative.
- Each procedure has verification.
- Warnings identify exact risk.
- Config keys include defaults.
- Links use Antora `xref` where possible.
- No marketing language remains.
- No unexplained acronyms remain.

## Common Rewrites

Prefer these transformations:

- "This guide will walk you through" -> "This page explains"
- "Simply run" -> "Run"
- "Make sure to" -> "Ensure that"
- "You can easily configure" -> "Configure"
- "Powerful and flexible" -> describe concrete capability
- "It is important to note" -> `NOTE` or direct sentence
- "In order to" -> "To"
- "Utilize" -> "Use"

## Output Expectations

When asked to write docs:

- Create or edit Antora files directly when workspace is available.
- Preserve existing project terminology and structure.
- Keep changes small and coherent.
- If source behavior is unclear, inspect code before writing claims.
- If behavior still cannot be confirmed, write conditional wording or ask one focused question.


Overview
Getting Started
Features
  Domain 1 Manual & usage guide best practice
  Domain 2 Manual & usage guide
Architecture
Operations
   FluxCD Deployment
Reference
   Configuration
   API Reference
   Events
      Domain
        Domain event 1
        Domain event 2
   Well-Known Tags
   IAM Actions
Troubleshooting
Architecture Decision Records
