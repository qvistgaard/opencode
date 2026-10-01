---
name: api-docs
description: Use when creating or updating API reference documentation from Java, Go, OpenAPI, Swagger, REST handlers, controllers, routers, tests, or generated docs, especially Antora endpoint pages with API reference cards.
---

# API Documentation

Use this skill when documenting API endpoints from application code, tests, OpenAPI/Swagger specs, generated docs, or existing API pages. Applies to Java, Go, Spring, Gin, Chi, Echo, net/http, OpenAPI-first services, and generated Swagger2Markup/AsciiDoc sources.

## Workflow

1. Inspect application code first: routes, handlers/controllers, DTOs, media types, status codes, errors, auth, tests, and existing docs.
2. Verify actual behavior from source or tests. Do not invent parameters, response fields, filters, or status codes.
3. Compare old and new endpoints when migrating. Document breaking changes, compatibility media types, renamed parameters, and visibility/filter behavior.
4. Keep docs and OpenAPI/code aligned for method, path, parameters, headers, media types, status codes, request body, response shapes, and deprecation status.

## Endpoint Page Order

Use this order for reference pages:

1. Title and short purpose sentence.
2. Operation heading with method/path.
3. Important migration/deprecation/visibility warnings near operation.
4. Request.
5. Response.
6. Example requests.
7. Behavior.
8. Equivalent lower-level request or implementation mapping, if useful.
9. Migration from legacy endpoint.
10. Related links.

For short endpoints, omit sections that add no value.

## Operation

Show method/path near top:

```asciidoc
`POST /api/v1/curator/products`
```

Use raw HTTP examples, not curl, when showing protocol details:

```asciidoc
[source,http]
----
POST /api/v1/service/resources?target=... HTTP/1.1
Host: api.example.com
Content-Type: application/json
Accept: application/vnd.example+json

{
  "name": "example"
}
----
```

## Request Sections

Use these headings:

```asciidoc
=== Request

==== Query Parameters

==== Accept Header

==== Request Body Parameters
```

Use API reference cards, not tables, for endpoint params when current page uses card design.

Card style rules:

- Wrapper font: `system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif; font-size: 14px`.
- Param/header name: mono `15px`, `700`, color `#111827`.
- Type/value: mono `13px`, `500`, color `#64748b` for type, `#111827` for values.
- Description: system `14px`, line-height `1.55`, color `#334155`.
- Labels such as `Example`, `Response body`, `Use case`: `13px`, `700`, color `#475569`.
- Required badge: `10px`, uppercase, red text `#b42318`, bg `#fff1f3`, border `#fda29b`, pill radius.
- Default badge: green text `#0f5132`, bg `#ecfdf3`, border `#abefc6`.
- Legacy/deprecated badge: gray text `#475569`, bg `#fff` or `#f8fafc`, border `#cbd5e1`.
- Cards: border `#dfe3e8`, radius `8px`, bg `#fff`, padding `1rem 1.125rem`.
- Nested option cards: border `#e2e8f0`, radius `6px`, bg `#f8fafc`, padding `.75rem`.

Do not show the same request body example twice. If the request body card includes an example, omit a standalone duplicate JSON block unless the examples section needs a full HTTP request.

## Accept Header

When multiple response formats are selected by `Accept`, use one `Accept` card with multiple valid values inside. Do not create one header card per media type.

Each value should include:

- media type
- `default`, `legacy`, or `deprecated` badge when relevant
- response body type
- use case

## Response Sections

Use this order:

```asciidoc
=== Response

include::partial$api-examples/<endpoint>-response-codes.adoc[]

==== Response Body Formats

include::partial$api-examples/<endpoint>-response-formats.adoc[]
```

Use response-code cards instead of tables. One `Response` card contains all status code options.

Each status option should include:

- status code
- status badge: `success`, `invalid request`, `not found`, etc.
- meaning
- body

## Response Format Switcher

For multiple response body formats, use a self-contained radio switcher.

Requirements:

- Tab labels are the exact `Accept` header values.
- JSON examples use real `[source,json]` blocks so syntax highlighting works.
- Use passthrough HTML only for wrapper, CSS, radio controls, and tab labels.
- Use unique CSS class and input IDs per endpoint to avoid collisions.
- Do not put raw JSON in `<pre><code>` unless syntax highlighting is not needed.

Pattern:

```asciidoc
++++
<style>
  .api-<endpoint>-response-switch {
    margin: 1rem 0 1.5rem;
    font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
    font-size: 14px;
  }
  .api-<endpoint>-response-switch input {
    position: absolute;
    opacity: 0;
    pointer-events: none;
  }
  .api-<endpoint>-response-tabs {
    display: inline-flex;
    border: 1px solid #cbd5e1;
    border-radius: 8px;
    overflow: hidden;
    background: #f8fafc;
    margin-bottom: .75rem;
  }
  .api-<endpoint>-response-tabs label {
    cursor: pointer;
    padding: .5rem .85rem;
    font: 700 13px/1.4 system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
    color: #475569;
    border-right: 1px solid #cbd5e1;
  }
  #api-<endpoint>-response-v1:checked ~ .api-<endpoint>-response-tabs label[for="api-<endpoint>-response-v1"] {
    background: #0f172a;
    color: #fff;
  }
  .api-<endpoint>-response-panel {
    display: none;
  }
  #api-<endpoint>-response-v1:checked ~ .api-<endpoint>-response-v1 {
    display: block;
  }
  .api-<endpoint>-response-panel .title {
    font: 700 15px/1.4 ui-monospace, SFMono-Regular, Menlo, Consolas, 'Liberation Mono', monospace;
    color: #111827;
    background: #f8fafc;
    border: 1px solid #dfe3e8;
    border-bottom: 0;
    border-radius: 8px 8px 0 0;
    padding: .875rem 1rem;
    margin: 0;
  }
  .api-<endpoint>-response-panel .content {
    border: 1px solid #dfe3e8;
    border-radius: 0 0 8px 8px;
    overflow: hidden;
  }
</style>

<div class="api-<endpoint>-response-switch">
  <input id="api-<endpoint>-response-v1" name="api-<endpoint>-response-format" type="radio" checked>
  <div class="api-<endpoint>-response-tabs">
    <label for="api-<endpoint>-response-v1">application/vnd.example+json</label>
  </div>
++++

[source,json,role="api-<endpoint>-response-panel api-<endpoint>-response-v1"]
.application/vnd.example+json - ResponseType
----
{
  "id": "example"
}
----

++++
</div>
++++
```

## Examples

Put examples after response documentation unless the project explicitly follows example-first pages.

Use raw HTTP request examples:

```asciidoc
=== Example Requests

==== V1 Request

[source,http]
----
GET /api/v1/service/resources?id=123 HTTP/1.1
Host: api.example.com
Accept: application/vnd.example+json
----
```

Avoid duplicating response examples if the response format switcher already contains them.

## Migration Notes

Put critical replacement/deprecation/visibility warnings near operation, not hidden at the bottom.

Use:

- `[IMPORTANT]` for required migration, deprecation, or compatibility warnings.
- `[NOTE]` for contextual links and non-blocking behavior notes.

Migration section should include:

- old route and new route
- removed/renamed parameters
- required new parameters
- media type changes
- response shape changes
- status code behavior changes
- compatibility/legacy media type
- client update checklist

## Tables

Avoid tables for endpoint parameters, `Accept` values, and response codes when card style is used. Use tables only when comparison is clearer, especially large matrix/reference data.

## Java And Go Source Reading

For Java:

- Inspect Spring annotations, JAX-RS annotations, OpenAPI annotations, DTOs, validation annotations, tests, security annotations, and media type constants.
- If Source2Sea Spring APIs are involved, also use `spring-rest-api`.

For Go:

- Inspect router setup, middleware, handler signatures, request binding, response writers, DTO structs, validation tags, OpenAPI generation, tests, and error helpers.
- Trace route registration through framework-specific groups/muxes before documenting paths.

## Review Checklist

Before finishing API docs:

- Operation method/path matches code.
- Query/path/header/body parameters match code and tests.
- Required/optional status is correct.
- Media type negotiation is explicit.
- Response codes match code.
- Response examples match DTO fields.
- Raw HTTP examples use correct headers.
- Migration warnings are near top.
- No duplicate request/response examples.
- Related links include legacy/replacement pages and domain behavior pages.
