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
        operations.adoc
      examples/
      partials/
      images/
    reference/
      nav.adoc
      pages/
        configuration.adoc
        api-reference.adoc
        events.adoc
        iam-actions.adoc
      examples/
      partials/
      images/
```

Put reference pages in their own Antora module, such as `modules/reference`, not in the `ROOT` module. Keep overview, getting started, feature, architecture, operations, troubleshooting, and ADR pages in `ROOT` unless the project already uses a more specific module structure.

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

## Documentation Information Architecture

Assume this top-level documentation structure unless the project already uses a clear alternative:

```text
Overview
Getting Started
Features
Architecture
Operations
Reference
Troubleshooting
Architecture Decision Records
```

Use each section for a distinct reader need:

- `Overview`: project purpose, scope, main components, supported environments, and next links.
- `Getting Started`: shortest path from prerequisites to a verified working result.
- `Features`: user-facing capabilities, domain manuals, and usage guides.
- `Architecture`: system design, component relationships, data flow, and important tradeoffs.
- `Operations`: deployment, FluxCD deployment, startup, shutdown, upgrades, backups, metrics, logs, and runtime procedures.
- `Reference`: lookup material that should be complete rather than tutorial-oriented.
- `Troubleshooting`: symptom-driven diagnosis, causes, fixes, and verification.
- `Architecture Decision Records`: decision history with context, decision, status, and consequences.

ADRs are immutable historical records. Keep old decisions intact, mark stale statements as historical proposals, and add current implementation notes so the page does not describe false current behavior.

Use this ADR page metadata pattern:

```asciidoc
= {adr-id} - Record architecture decisions
:page-description: Architecture decisions are documented using Architecture Decision Records (ADRs).
:keywords: architecture, adr, documentation
:revdate: 22-11-2022
:author: Steffen F. Qvistgaard
:adr-id: {component-id}-ADR-0001
```

ADR metadata rules:

- Use `{adr-id}` in the page title so the rendered title includes the component-specific ADR identifier.
- Define `:adr-id:` as `{component-id}-ADR-NNNN`, with a zero-padded sequence such as `0001`.
- Define `component-id` in `antora.yml` under `asciidoc.attributes`.
- Use `:revdate:` for the decision or last meaningful documentation date.
- Use `:author:` for the ADR author.
- Keep stale ADR content, but add current implementation notes instead of rewriting history.

Always organize feature and reference material under these topics:

```text
Features
  <Feature Name>      (one page per domain concept, e.g. Users, Groups, Roles)
  <Feature Name>
  ...
Reference
  Configuration
  API Reference
  Events
    Domain Events
  Well-Known Tags
  IAM Actions
```

Each domain concept gets its own dedicated page under `Features`. Do not use generic `Domain Manual` or `Domain Usage Guide` titles. Name each page after the concept it describes, such as `Users`, `Groups`, or `Roles`.

Omit topics or pages when there is no valuable information for them. For example, do not create a `Well-Known Tags` page when the project does not define tags. Preserve the information architecture as the default shape, but do not add empty or low-value placeholder pages.

Keep domain pages practical:

- Put narrative usage, best practices, and workflows under `Features`.
- Put exhaustive keys, APIs, events, tags, and IAM action names under `Reference`.
- Put reference pages in the `reference` Antora module, not under `ROOT`.
- Use `Architecture` only for design explanation, not feature walkthroughs.
- Use `Operations` for live-system work, including deployment tools such as FluxCD.

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

### Feature Page

Use one page per domain concept under `Features`. Name the page after the concept: `users.adoc`, `groups.adoc`, `roles.adoc`.

Structure each feature page in this order:

1. **Overview** — what the concept is, its purpose, and how it fits into the broader system.
2. **Concepts** — key terms, types, states, or relationships the reader must understand before using the feature.
3. **How It Works** — behavioral description: lifecycle, rules, constraints, side effects, and interactions with other features.
4. **Usage** — step-by-step procedures for the most common tasks. Use ordered lists. One procedure per heading.
5. **Permissions** — IAM actions required for each operation. Link to the IAM Actions reference page.
6. **Related** — cross-links to related feature pages, API reference, and event reference.

Use this page template:

```asciidoc
= Users
:page-description: How users are managed in the platform.
:keywords: users, identity, iam

[[users-overview]]
== Overview

Describe what a user is and its role in the system.

[[users-concepts]]
== Concepts

Define terms. Use a description list or short paragraphs.

[[users-how-it-works]]
== How It Works

Explain lifecycle, states, rules, and interactions.

[[users-usage]]
== Usage

[[users-usage-create]]
=== Create a User

. Step one.
. Step two.
. Verify result.

[[users-usage-delete]]
=== Delete a User

. Step one.
. Step two.

[[users-permissions]]
== Permissions

List IAM actions required per operation. Link to xref:reference:iam-actions.adoc[].

[[users-related]]
== Related

* xref:groups.adoc[]
* xref:reference:api-reference.adoc[]
* xref:reference:events.adoc[]
```

Rules:

- Do not merge multiple domain concepts into one feature page.
- Do not use `Domain Manual` or `Domain Usage Guide` as page or section titles.
- Include only sections that have real content. Omit empty sections.
- Keep overview short: two to four sentences.
- Keep concept definitions precise. Avoid marketing language.
- Procedures must be numbered steps. Each step states one action and its expected result.
- Link permissions to the IAM Actions reference rather than repeating action names inline.

### Configuration Page

Use a three-column table grouped by functional area. Add an explicit anchor before each group heading.

```asciidoc
[[configuration-server-management]]
== Server and Management

[cols="3,5,2", options="header"]
|===
| Property | Description | Default

| `server.port`
| HTTP port.
| `8183`

| `management.server.port`
| Management port.
| `8181`

| `management.endpoints.web.exposure.include`
| Exposed actuator endpoints.
| `*`

| `management.endpoint.health.show-details`
| Health detail visibility.
| `always`

| `management.tracing.sampling.probability`
| Trace sampling probability.
| `1`

| `management.opentelemetry.tracing.export.otlp.endpoint`
| OTLP trace export endpoint.
| `http://localhost:4318/v1/traces`

| `management.otlp.metrics.export.enabled`
| OTLP metrics export switch.
| `false`
|===
```

For each property, document:

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

### IAM Actions Reference Page

Use only when the project defines service-specific IAM or authorization actions. Omit this page when there are no IAM actions.

Include:

- Page title: `IAM Actions Reference`.
- Page description that states the page covers service-specific IAM resources, actions, and condition context keys for permission policies.
- Keywords such as `iam`, `security`, `authorization`, and `permissions` when relevant.
- Navigation title: `IAM Actions`.
- Overview that states the service name and service prefix.
- Short explanation of how actions are used in permission policies.
- One section per action group, domain, or service prefix.
- Service prefix for each group.
- Table of actions with columns for action name, description, and access level.

Use this table shape:

```asciidoc
[cols="2,4,1", options="header"]
|===
| Action | Description | Access Level

| `service:CreateResource`
| Create a new resource.
| Write
|===
```

Action entries must:

- Use fully qualified action names such as `catalog:CreateProduct`.
- Use consistent access levels such as `Read`, `Write`, `List`, `Tagging`, or project-specific values.
- Describe the exact operation authorized by the action.
- Mention destructive effects in the description, such as permanent deletion or affected related resources.
- Match application behavior verified from source code, policy definitions, API handlers, or tests.
- Avoid inventing actions from CRUD naming conventions unless the implementation defines them.

When the system supports IAM resources or condition context keys, add separate sections for them. Include resource formats, supported actions, condition key names, value types, and when each condition applies.

### Domain Events Reference Pages

Use only when the application emits domain events. Omit event reference pages when the application has no domain events.

Document emitted domain events under the `reference` Antora module with an event index page and one page per event. Group event pages by domain or aggregate.

Use this navigation shape:

```asciidoc
** xref:reference:events.adoc[Event Reference]
*** Catalog Events
**** xref:reference:events/catalog/catalog-created.adoc[]
**** xref:reference:events/catalog/catalog-updated.adoc[]
**** xref:reference:events/catalog/catalog-deleted.adoc[]
*** Product Events
**** xref:reference:events/product/product-created.adoc[]
**** xref:reference:events/product/product-deleted.adoc[]
**** xref:reference:events/product/basic-product-data-changed.adoc[]
**** xref:reference:events/product/category-added.adoc[]
**** xref:reference:events/product/category-unassigned.adoc[]
```

The event index page should include:

- Purpose of domain events in the application.
- Domains or aggregates that emit events.
- Event list grouped by domain or aggregate.
- Links to event specification pages.
- Notes about delivery semantics, ordering, versioning, and compatibility when the implementation defines them.

Each event specification page should include:

- Page title using event object name, such as `` `ProductCreated` Object Specification``.
- `:navtitle:` matching the event name.
- Short description of when the event is emitted.
- PlantUML JSON payload diagram using `[plantuml, <EventName>-diagram-json, png]`; include it for every event.
- Attribute table for top-level fields.
- Attribute tables for nested objects.
- Example JSON representation.
- Compatibility or version notes when event payloads are versioned.

Use this complete event page template. Replace names, descriptions, types, fields, payload values, and anchors with values verified from implementation:

```asciidoc
= CatalogDeleted Event
:page-description: Event emitted when a catalog is deleted from Warehouse.
:keywords: events, catalog
:navtitle: CatalogDeleted

[[catalog-deleted-overview]]
== Overview

The `CatalogDeleted` event is emitted when a catalog is deleted from the Warehouse system.

Event name::
`catalogDeleted`

Aggregate::
Catalog

[[catalog-deleted-diagram]]
== Payload Diagram

[plantuml, CatalogDeleted-diagram-json, png]
----
@startjson
!theme sandstone
{
  "CatalogDeleted": {
    "catalog": "The Source2Sea Resource Name (SRN) identifying the deleted catalog"
  }
}
@endjson
----

[[catalog-deleted-attributes]]
== Attributes

[cols="2,1,5", options="header"]
|===
| Field | Data Type | Description

| `catalog`
| CatalogSrn
| The Source2Sea Resource Name (SRN) identifying the deleted catalog.
|===

[[catalog-deleted-example]]
== Example

[source,json]
----
{
  "catalog": "srn:catalog:account:12345:catalog:67890"
}
----
```

Use this attribute table shape:

```asciidoc
[cols="2,1,5,2", options="header"]
|===
| Field | Data Type | Description | Default Value

| `name`
| String
| The full name of the product.
| `null`
|===
```

Event documentation must:

- Match events emitted by the application, verified from source code, event definitions, message schemas, tests, or configuration.
- Describe payload fields exactly as serialized.
- Include PlantUML JSON diagram for every event. Diagram root key must exactly match event object name.
- Keep diagram, attribute tables, and JSON example same payload structure and exact serialized field names.
- Use meaningful field-description strings in diagrams, not sample values, unless field needs a concrete format example.
- Include examples that match actual field names and JSON shape.
- Document whether fields are optional, nullable, defaulted, or always present.
- Document nested objects in their own subsections.
- Avoid inventing events from domain concepts unless the implementation emits them.
- Avoid saying an event is emitted for an operation unless verified from implementation.

## nav.adoc Pattern

Derive `nav.adoc` from `Documentation Information Architecture`. Keep labels short and preserve that order unless the existing project structure requires otherwise.

Use this pattern for mechanics:

```asciidoc
* xref:index.adoc[]
* xref:getting-started.adoc[]
* Features
** xref:features/domain-manual.adoc[]
** xref:features/domain-usage-guide.adoc[]
* xref:architecture.adoc[]
* Operations
** xref:operations.adoc[]
* Reference
** xref:reference:configuration.adoc[]
** xref:reference:events.adoc[]
* xref:troubleshooting.adoc[]
* xref:adr/index.adoc[Architecture Decision Records]
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
  - modules/reference/nav.adoc
asciidoc:
  attributes:
    component-id: WH
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
