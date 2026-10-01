---
name: spring-search-api
description: Use when designing, implementing, reviewing, or documenting a Spring Boot unified search API using the shared search spec: SearchRequest, SearchResponse, domains, constraints, normalized search items, CQRS queries, or Elasticsearch-backed search.
---

# Spring Unified Search API

Use this skill when a Spring Boot application implements or extends a unified search API based on this contract:

- One endpoint receives a generic search request.
- The request can search one or more domains.
- Domains can use different data sources.
- Constraints are typed and domain-targeted.
- Results are returned in one normalized item shape.

This skill is framework guidance. Preserve project-specific package names, media types, authentication, IAM annotations, CQRS factories, test style, and documentation structure.

## Load With Related Skills

Also load:

- `spring-rest-api` when adding or changing controllers/spec interfaces.
- `antora-docs` when documenting search API pages or request/response examples.

## Contract Shape

Use one request DTO for all search domains:

```java
public class SearchRequest {
  private String query;
  private Set<String> domains;
  private Integer limit;
  private String cursor;
  private String sort;
  private List<SearchRequestConstraint> constraints;
}
```

Validation:

- Require nonblank `query` or nonempty `constraints`.
- Allow `domains` to be omitted or empty. Omitted means all supported domains.
- Ignore unsupported domains.
- Treat nonpositive or missing `limit` as default page size.
- Treat invalid or mismatched cursor according to product policy. Default fallback is first page.

Use one response DTO:

```java
public class SearchResponse {
  private String query;
  private long tookMs;
  private List<SearchItem> results;
  private String nextCursor;
}
```

Use normalized items:

```java
public class SearchItem {
  private String domain;
  private String srn;
  private String id;
  private String title;
  private String description;
  private Double score;
  private Map<String, Object> metadata;
}
```

## Constraint Wire Format

Use flat constraint JSON. Do not require a nested `properties` object.

Correct:

```json
{
  "type": "location",
  "targets": ["products"],
  "srn": ["srn:port::port/AE-DXB", "srn:port::port/NL-RTM"]
}
```

Avoid:

```json
{
  "type": "location",
  "targets": ["products"],
  "properties": {
    "srn": "srn:port::port/AE-DXB"
  }
}
```

Implementation pattern:

- DTO has `type`, `targets`, and internal `JsonNode properties`.
- Custom deserializer removes `type` and `targets` from raw object.
- Remaining fields become `properties` for converter parsing.
- Unknown fields are preserved.
- Unknown constraint types are ignored unless strict mode is explicitly required.

## Constraint Design

Constraints must be domain-targeted:

```json
{
  "type": "category",
  "targets": ["products"],
  "srn": "srn:catalog:..."
}
```

Rules:

- `targets` selects domains that should apply the constraint.
- A domain ignores constraints not targeting that domain.
- Multiple values inside one constraint type usually combine with OR.
- Different constraint types usually combine with AND.
- Single-value and list-value forms should be accepted where useful.
- Prefer domain-specific field names, such as `srn`, over vague `value` when the payload has clear meaning.

Common product-style constraints:

- `products`: product SRN string or array.
- `catalog`: catalog SRN string or array.
- `target`: buyer, fleet, agreement, or commercial target SRN string or array.
- `location`: location or port SRN string or array.
- `category`: category SRN string or array.
- `state`: boolean filters such as `present`, `valid`, `deleted`.

Common geo-style constraints:

- `proximity`: `origin.latitude`, `origin.longitude`, `radius`.
- `function`: function/capability filter with `operator` plus string or array value.

Document each supported constraint:

- Purpose.
- Target domains.
- Accepted fields.
- Single vs list behavior.
- OR/AND composition rules.
- Side effects on returned metadata.
- Example request JSON.

## Spring Controller Pattern

Keep controller thin.

Preferred flow:

```java
return queryFactory.create(Search.class)
    .account(accountSrn)
    .domains(request.getDomains())
    .pageCursor(request.getCursor())
    .query(request.getQuery())
    .constraints(searchRequestToConstraintConverter.convert(request))
    .limit(request.getLimit())
    .sort(request.getSort())
    .execute()
    .map(searchResponseConverter::convert);
```

Controller responsibilities:

- Bind authentication/account context.
- Bind request body.
- Convert HTTP constraints to internal constraints.
- Create and execute one CQRS query.
- Convert internal result to API response.

Do not put search DSL construction, ranking, pagination, or domain business rules in the controller.

## Internal Query Pattern

Use one top-level search query to orchestrate all domains:

1. Record start time.
2. Resolve requested domains.
3. Resolve page size.
4. Decode cursor.
5. Select constraints per domain.
6. Run domain searches, in parallel when useful.
7. Normalize candidates to search items.
8. Merge and sort if multiple domains are active.
9. Apply pagination slice.
10. Return `SearchResult` with `tookMs`, `results`, and optional `nextCursor`.

Domain source methods should stay isolated:

```java
private Flux<SearchResultItem> searchProducts(int fetchSize, int offset, List<Constraint> constraints) { ... }
private Flux<SearchResultItem> searchLocations(int fetchSize, int offset, List<Constraint> constraints) { ... }
```

## Elasticsearch Guidance

For indexed domains:

- Use `match_all` when query is blank and constraints are present.
- Use explicit weighted fields when query is nonblank.
- Exact identifier/SRN/code match should receive highest boost.
- Prefix/autocomplete match should receive high boost.
- Name/title fields should receive medium boost.
- Description/body fields should receive lower boost.
- Apply state/tenant/security filters as filters, not scoring clauses.
- Use `terms` filters for list constraints.
- If filtering multiple fields inside arrays of objects, verify whether mapping needs `nested` queries.

Cursor policy:

- Small first pass can use opaque offset cursor.
- High-volume search should prefer `search_after` with stable sort.
- Cursor should encode enough request context to avoid reuse with incompatible requests when correctness requires it.

## Normalization Guidance

Each domain must map raw hits to normalized items.

Required mappings:

- `domain`: stable domain identifier, such as `products`.
- `srn`: stable cross-system identity.
- `id`: domain-local code or identifier when available.
- `title`: primary display string.
- `description`: secondary text when available.
- `score`: backend score or computed rank.
- `metadata`: domain-specific data needed by clients.

Metadata rules:

- Keep response stable and additive.
- Avoid dumping persistence entities blindly.
- Filter metadata by active constraints when clients expect scoped results, such as target/location price rows.
- Document metadata fields in API reference.

## Testing Checklist

Add unit tests for:

- Request validation accepts query-only.
- Request validation accepts constraints-only.
- Request validation rejects missing query and constraints.
- Flat constraint JSON deserializes into `properties`.
- Constraint converter parses single and list forms.
- Constraint converter ignores unknown types.
- Controller delegates to CQRS query and passes fields.
- Query defaults omitted domains to supported domains.
- Query drops unsupported domains.
- Query applies default state/security filters.
- Query composes OR within a constraint type and AND across types.
- Result converter maps query result to API response.

When Elasticsearch query builders are hard to assert, test converter/controller behavior and add focused query tests around helper methods or captured search request builders.

## Documentation Checklist

Update API docs with:

- Endpoint path and method.
- Request field table.
- Supported domains.
- One section per constraint.
- Constraint composition rules.
- Request examples for text search and constraint-only search.
- Response item field table.
- Domain metadata field descriptions.
- Cursor behavior.
- Authorization action.

For each constraint, document:

- What it filters.
- Which domains support it.
- Required fields.
- Whether fields accept scalar, array, or both.
- OR/AND composition.
- Example JSON.

## Review Risks

Flag these issues in review:

- Nested `properties` required in public JSON when project follows flat spec.
- Business logic in controller.
- Endpoint returns domain persistence entity directly.
- Unknown constraints fail request without explicit strict-mode requirement.
- Domain filtering omitted, so constraints apply to wrong domains.
- Multiple SRN filters use AND when OR was intended inside one constraint type.
- Different constraint types use OR when AND was intended.
- `target` and `location` filters on array/object fields assume same-row semantics without nested mapping.
- Cursor is not opaque.
- Response omits `tookMs` or stable identity.
- API docs do not document each supported constraint.
