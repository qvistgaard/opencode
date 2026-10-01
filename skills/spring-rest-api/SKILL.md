---
name: spring-rest-api
description: Use when designing, implementing, reviewing, or migrating Spring Boot REST APIs, endpoints, controllers, OpenAPI contracts, or REST URL paths, especially Source2Sea APIs.
---

# Spring Boot REST API Design

Use this skill for new Spring Boot REST APIs and migrations of existing APIs to this controller pattern. Inspect target codebase first. Preserve established framework, package, reactive/imperative, error, security, and media-type conventions unless this skill gives an explicit Source2Sea rule.

## Scope

Apply Source2Sea-specific rules only when target is Source2Sea platform or user asks for them. For other projects, use resource naming and contract/controller separation, but do not invent Source2Sea media types, SRNs, IAM annotations, or CQRS factories.

## Public URLs

Source2Sea public surfaces use `source2sea.com`.

Reserved top-level paths:

- `/`: public landing page or authenticated entry point.
- `/auth`: authentication routes.
- `/apps`: user-facing applications.
- `/api`: machine-facing APIs.
- `/developer`: developer documentation.
- `/internal`: intentionally exposed operational/admin surfaces.

Do not add another top-level prefix without architectural approval. Do not put APIs under `/developer` or applications under `/api`.

Public API route format:

```
https://source2sea.com/api/v{version}/{service}/{resource}
```

Rules:

- Version every public API in path. Never publish `/api/{service}/...`.
- Service versions evolve independently. Publish only versions service supports.
- Use lowercase nouns. Use hyphens for multi-word resources. Use plural collection names.
- Use singular names only for singleton resources/subresources, for example `/users/{id}/profile`.
- Avoid path verbs for CRUD. Use HTTP method and resource route: `POST /users`, not `/createUser`.
- Action paths allowed only for non-CRUD actions: `/reports/{id}/export`, `/jobs/{id}/retry`.

Examples:

- Valid: `/api/v1/iam/users`
- Valid: `/api/v1/orders/order-items`
- Invalid: `/api/iam/users`
- Invalid: `/api/v1/iam/getUsers`
- Invalid: `/api/v1/iam/OrderItems`

## Design Workflow

Before code:

1. Inspect existing routes, controllers/specs, request/response DTOs, security annotations, error handling, tests, and OpenAPI configuration.
2. Identify resource, owning service, API version, collection/item/singleton shape, operations, response representations, and authorization actions.
3. Propose route table and migration mapping before edits when design is non-trivial.
4. Preserve public compatibility. Treat route, HTTP method, status, media type, request schema, and response schema changes as breaking unless compatibility is explicitly planned.

Use standard CRUD shape where domain fits:

| Intent | Method | Route | Success |
| --- | --- | --- | --- |
| List | `GET` | collection | `200` |
| Read | `GET` | item | `200` |
| Create | `POST` | collection | `201` |
| Replace/update | `PUT`/`PATCH` | item | `200` |
| Delete | `DELETE` | item | `204` |

Do not expose persistence entities directly. Use operation-specific request DTOs and stable response DTOs.

## Source2Sea Controller Pattern

Use two HTTP-boundary layers:

- `FeatureSpec`: canonical HTTP contract.
- `Features` or `FeatureController`: thin orchestration implementation.

### Spec Interface

Put all endpoint contract details in `*Spec` interface:

- `@GetMapping`, `@PostMapping`, `@PutMapping`, `@PatchMapping`, `@DeleteMapping`
- `consumes`, `produces`, and `@ResponseStatus`
- `@IamRequireAction`
- OpenAPI annotations: `@Operation`, `@ApiResponses`, request body/schema, path parameter docs
- `@Valid` request bodies
- method signatures

Every endpoint needs explicit authorization. Action names describe business capability, not HTTP verb.

### Controller Implementation

Put only orchestration in implementation:

- `@RestController`, class `@RequestMapping`, constructor-injected dependencies
- command/query creation and parameter binding
- edge conversion between API DTOs and domain values
- reactive composition and response metadata attachment
- `@Override` endpoint methods

Do not put business rules, persistence logic, domain validation, or duplicated mapping in controller.

### CQRS And Reactive Rules

Use `CommandFactory` for mutations and `QueryFactory` for reads where application uses these factories.

- `Flux<T>`: collection/event streams.
- `Mono<T>`: one resource response.
- `Publisher<T>`: only when existing use case requires it.
- Resolve and execute resources through existing SRN model attributes, such as `@ModelAttribute UserSrn`.
- Keep route path variable for routing/docs even if SRN is execution handle. Mark unused only when required by compiler/style.

### Mutation Representations

For Source2Sea mutations, expose event and model variants on same route where product design does not give clear exception:

- Event: `APPLICATION_VND_SOURCE2SEA_EVENT_JSON_VALUE`; execute command and return events directly.
- Model: `APPLICATION_VND_SOURCE2SEA_JSON_VALUE`; execute command, collect events, reload authoritative projection, attach emitted events in explicit metadata if API exposes them.

Never construct post-mutation model from request DTO. Query after command.

```java
@Override
public Mono<FeatureResponse> createAsModel(AccountSrn account,
                                           CreateFeatureRequest request) {
  return createFeature(account, request)
      .execute()
      .collectList()
      .flatMap(events -> getFeature(resolveFeature(events), events));
}
```

### Helpers

Command builder helper:

- Creates configured command with `commandFactory.create(...)`.
- Maps request/route data.
- Returns unexecuted command.
- Does not branch business logic, execute command, query, or construct response.

Projection loader helper:

- Runs query.
- Converts domain projection to response DTO.
- Attaches optional response metadata.
- Shared by endpoints needing same model projection.

Public endpoint method does one thing: run query; run command for events; or run command, collect events, then load model.

## New API Checklist

- Route uses `/api/v{version}/{service}/{resource}` for Source2Sea public API.
- Resource path lowercase, hyphenated, noun-based, plural for collections.
- Contract resides in `*Spec`; implementation remains thin.
- Request DTOs have `@Valid`; response DTOs are stable models.
- OpenAPI documents summary, description, IDs where project uses them, request schema, success, common failures, and path variables.
- Authorization explicit for every endpoint.
- Correct status codes and content negotiation declared.
- Event/model mutation variants implemented when applicable.
- Model response after mutation reloaded from query.
- Tests cover route, auth, validation, status, representation, and migration compatibility.
- API documentation updated with Antora rules when the project uses Antora docs.
- Deprecated or compatibility endpoints visibly marked in endpoint indexes and navigation, not only in OpenAPI annotations.
- Content negotiation documented when multiple `Accept` media types produce different response shapes.
- Migration notes document renamed query parameters, fallback aliases, and precedence rules.
- Domain behavior affecting responses, counts, eligibility, filtering, or visibility linked from API docs to feature/behavior docs.

## Migration Workflow

1. Inventory old routes: method, path, request/response schema, media type, auth, status, consumers, and tests.
2. Design target resource routes and version strategy. Do not silently repurpose old public routes.
3. Add `FeatureSpec` with full new contract and OpenAPI/security/validation metadata.
4. Add thin implementation with command builders and shared projection loader.
5. Migrate read endpoints to direct queries. Migrate writes to event/model variants and query-after-command model responses.
6. Keep legacy routes only when documented compatibility window requires them. Mark deprecated in OpenAPI, test both surfaces, and remove after migration deadline.
7. Test old and new behavior, then update API documentation and clients.
8. Do not describe intentional design changes as parity gaps. Track only required client-impacting differences as gaps.

## API Documentation Handoff

When implementing or migrating Spring APIs in a project with Antora docs:

- Use the `api-docs` skill for endpoint reference page structure, parameter cards, media type documentation, raw HTTP examples, response code cards, and response format switchers.
- Update endpoint page, API index, nav, examples, and IAM reference if affected.
- Put deprecation or replacement information near top of deprecated endpoint pages.
- Keep unrelated documentation advisories in separate admonition blocks.
- Use `[IMPORTANT]` for deprecation or required migration warnings.
- Use `[NOTE]` for contextual links, replacement details, and non-blocking behavior notes.
- Link API docs to behavior pages for domain rules that affect inclusion, visibility, counts, filtering, or eligibility.
- Use stable Antora anchors for rule links.
- Keep docs and OpenAPI consistent for parameter names, media types, status codes, and response shapes.

## Review Rules

Flag:

- unversioned public Source2Sea API
- non-`source2sea.com` public Source2Sea service
- undeclared top-level prefix
- camelCase, uppercase, verb-based, or singular collection path
- business logic or persistence in controller
- route/security/OpenAPI/validation annotations in implementation instead of spec
- missing explicit auth
- direct model construction after mutation
- repeated command mapping or response conversion
- missing event/model mutation representation without product reason
- breaking migration without compatibility plan
- missing API documentation updates for changed public route, media type, parameter, status, response shape, deprecation, or domain rule link
- deprecated endpoints hidden in docs index/nav or marked only on detail pages
- mixed-topic admonition blocks that combine deprecation, migration, response format, and domain behavior notes
