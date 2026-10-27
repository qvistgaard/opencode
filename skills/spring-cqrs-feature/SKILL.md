---
name: spring-cqrs-feature
description: Use when designing, implementing, or reviewing Spring Boot features using Event-Sourced CQRS architecture, especially creating domain features with commands, queries, event sourcing, and aggregate projections.
---

# Spring CQRS Feature

Use this skill when designing, implementing, or reviewing Spring Boot features using Event-Sourced CQRS architecture. This includes creating new domain features with commands, queries, event sourcing, and aggregate projections.

**Related Skills:**
- Use **spring-rest-api** skill for controller patterns, OpenAPI annotations, request/response DTOs, dual response endpoints, and API design
- Use **api-docs** skill for Antora endpoint documentation

---

## Architecture Overview

### Event-Sourced CQRS Pattern

The architecture separates **write operations** (commands) from **read operations** (queries):

- **Commands**: Business logic that emits domain events to event store (Cassandra via Evento)
- **Queries**: Read projections and materialized views from read models
- **Events**: Immutable state changes stored in event stream
- **Projections**: Aggregate state reconstructed from events
- **Aggregates**: Domain objects that ensure consistency and emit events

### Factory-Based Dependency Injection

Controllers use factories to create command/query instances with dependencies injected:

```java
@RestController
@RequiredArgsConstructor
public class Policies {
  private final CommandFactory commandFactory;  // Creates commands
  private final QueryFactory queryFactory;      // Creates queries
  
  CreatePolicy command = commandFactory.create(CreatePolicy.class);
  ListPolicies query = queryFactory.create(ListPolicies.class);
}
```

### Technology Stack

- **Spring Boot** (version 3.x or later recommended)
- **Spring WebFlux** (Project Reactor - reactive) OR **Spring Web MVC** (servlet-based)
  - Use WebFlux if already using reactive stack
  - Use Web MVC otherwise (recommended for new projects with Evento)
- **Evento** (event sourcing framework)
- **Apache Cassandra** (event store + read models via evento-cassandra)
- **Apache Kafka** (event streaming)
- **Lombok** (code generation)

---

## Feature Implementation Checklist

### Step-by-Step Guide

When adding a new feature (e.g., "Tags" resource):

#### **Phase 1: Domain Model (-internals module)**

1. Create package structure under `features/{feature}/`
2. Define aggregate identifier (SRN class)
3. Create base command class (`Reactive{Feature}Command`)
4. Implement concrete commands (Create, Update, Delete, specialized actions)
5. Define domain events (Created, Updated, Deleted, etc.)
6. Create queries (Get, List, History)
7. Define projections (`{Feature}Details`)
8. Implement ECST (AutoConfiguration, Service, Enable annotation) - see Advanced Patterns
9. Create list repositories (Cassandra)
10. Add list management commands if using async pattern (Add/Update/Remove from list)
11. Create domain models and exceptions

#### **Phase 2: API Layer (-server or -application module)**

**See spring-rest-api skill** for complete controller, OpenAPI, DTO, and endpoint patterns.

REST endpoints go in the `-server` module if the project has one, otherwise in `-application` module (historical).

11. Create controller (`{Features}.java`) implementing spec interface
12. Define request DTOs (Create/Update requests with validation)
13. Define response DTOs (Response, MutationResponse, Metadata)
14. Create converters (domain → API response)
15. Implement dual endpoints (events + model) - see spring-rest-api skill
16. Add command builder helpers and projection loader helpers
17. Add exception handlers to `ControllerAdvice`

#### **Phase 3: Event Handlers (-consumer, -server, or -application module)**

Event handlers go in the `-consumer` module if the project has one for async processing, otherwise in `-server` or `-application` module.

18. Choose list update pattern (async with event handlers OR sync with EventTransaction in commands)
19. If async: Create event handler functions (`{Feature}ListFunctions.java`)
20. If async: Wire up event handlers for Created/Updated/Deleted events
21. If async: Configure event routing and error handling

#### **Phase 4: Infrastructure**

22. Create Cassandra migration scripts (event store + list tables)
23. Add managed data (YAML) if applicable
24. Write unit tests (commands, queries, aggregates, list commands if async)
25. Write integration tests (controllers, event handlers if async)
26. Update API documentation (use api-docs skill for Antora)

---

## Package Structure Template

```
{project}-internals/src/main/java/com/{org}/{project}/features/{feature}/
├── aggregates/
│   ├── commands/
│   │   ├── Reactive{Feature}Command.java          ← Base command class
│   │   ├── Create{Feature}.java                   ← Create command
│   │   ├── Update{Feature}.java                   ← Update command
│   │   ├── Delete{Feature}.java                   ← Delete command
│   │   └── {SpecializedAction}.java              ← e.g., AttachPolicy, AssignRole
│   ├── events/
│   │   ├── {Feature}Created.java
│   │   ├── {Feature}Updated.java
│   │   ├── {Feature}Deleted.java
│   │   └── {SpecializedEvent}.java
│   ├── queries/
│   │   └── Get{Feature}.java                      ← Single item query
│   └── projections/
│       └── {Feature}Details.java                  ← Aggregate projection
├── list/
│   ├── commands/                                   ← Only if using async pattern
│   │   ├── Add{Feature}ToList.java
│   │   ├── Update{Feature}InList.java
│   │   └── Remove{Feature}FromList.java
│   ├── queries/
│   │   └── List{Features}.java                    ← List query
│   └── repositories/
│       ├── {Feature}ListItem.java                 ← Cassandra entity
│       └── {Feature}ListRepository.java           ← Spring Data Cassandra
├── models/
│   └── {Feature}.java                             ← Domain model
├── queries/
│   └── Get{Feature}History.java                   ← Event history query
└── exceptions/
    └── {Feature}NotFoundException.java

{project}-server/src/main/java/com/{org}/{project}/server/     (or -application if no -server)
├── controllers/
│   ├── {Features}.java                             ← REST controller
│   ├── {Features}Spec.java                        ← OpenAPI spec (optional)
│   ├── converters/
│   │   └── {Feature}To{Feature}ResponseConverter.java
│   └── models/
│       └── {feature}s/
│           ├── Create{Feature}Request.java
│           ├── Update{Feature}Request.java
│           ├── {Feature}Response.java
│           ├── {Feature}MutationResponse.java
│           └── {Feature}ResponseMetadata.java

{project}-consumer/src/main/java/com/{org}/{project}/consumer/ (or -server/-application if no -consumer)
└── messaging/                                       ← Only if using async pattern
    └── {Feature}ListFunctions.java                 ← Event handlers for list updates
```

---

## Critical Imports and Annotations

### Event Sourcing Framework Imports

**Always use these correct imports from the Evento framework:**

```java
// Annotations (NOT the classes!)
import com.source2sea.evento.annotations.Aggregate;       // For aggregate classes
import com.source2sea.evento.annotations.Event;           // For event classes
import com.source2sea.evento.annotations.Projection;      // For projection classes
import com.source2sea.evento.annotations.EventHandler;    // For event handler methods

// Classes (for return types and method parameters)
import com.source2sea.evento.aggregate.Event;             // For Event<A, E> return types
import com.source2sea.evento.aggregate.ReactiveAggregateRepository;
import com.source2sea.evento.aggregate.ReactiveAggregate;
```

### Common Import Mistakes

❌ **WRONG** - Do not import these:
```java
import com.source2sea.evento.aggregate.Aggregate;    // Class, not annotation!
import com.source2sea.evento.aggregate.Event;        // When used as @Event annotation
import com.source2sea.evento.aggregate.Projection;   // Class, not annotation!
import com.source2sea.evento.aggregate.Event;           // Wrong package!
```

✅ **CORRECT** - Use these instead:
```java
import com.source2sea.evento.annotations.Aggregate;   // Annotation
import com.source2sea.evento.annotations.Event;       // Annotation
import com.source2sea.evento.annotations.Projection;  // Annotation
import com.source2sea.evento.aggregate.Event;         // For Event<A, E> types
```

### Aggregate Class Template

```java
package com.source2sea.iam.features.applications.aggregate;

import com.source2sea.evento.annotations.Aggregate;  // ← Annotation, not class!

@Aggregate("Application")
public class Application {
}
```

### Event Class Template

```java
package com.source2sea.iam.features.applications.aggregate.events;

import com.source2sea.evento.annotations.Event;  // ← Annotation, not class!
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Event("iam:application:ApplicationCreated")
@Data
@NoArgsConstructor
@AllArgsConstructor  // ← Only if event has fields!
public class ApplicationCreated {
  private String name;
  private String baseUrl;
}
```

**Empty events:** If an event has no fields (like `ApplicationDeleted`), omit `@AllArgsConstructor` to avoid duplicate constructor errors:

```java
@Event("iam:application:ApplicationDeleted")
@Data
@NoArgsConstructor  // ← Only @NoArgsConstructor for empty events
public class ApplicationDeleted {
}
```

### Projection Class Template

```java
package com.source2sea.iam.features.applications.aggregate.projections;

import com.source2sea.commons.srn.iam.ApplicationSrn;
import com.source2sea.evento.aggregate.Aggregate;
import com.source2sea.evento.aggregate.Event;
import com.source2sea.evento.annotations.EventHandler;  // ← For methods
import com.source2sea.evento.annotations.Projection;    // ← For class
import lombok.Data;

@Projection(value = Application.class, name = "application-details")
@Data
public class ApplicationDetails {
  private final ApplicationSrn srn;  // ← Make SRN final
  private String name;
  private boolean present;

  /**
   * Initializes the projection for the given aggregate.
   * 
   * @param application the source aggregate
   */
  public ApplicationDetails(Aggregate<Application> application) {
    this.srn = new ApplicationSrn(application.getId());  // ← Initialize SRN from aggregate ID
  }

  @EventHandler  // ← Required on every event handler method
  public void on(Event<?, ApplicationCreated> event) {
    this.name = event.getData().getName();
    this.present = true;
  }

  @EventHandler
  public void on(Event<?, ApplicationUpdated> event) {
    if (event.getData().getName() != null) {
      this.name = event.getData().getName();
    }
  }

  @EventHandler
  public void on(Event<?, ApplicationDeleted> event) {
    this.present = false;
  }
}
```

**Important Projection Rules:**
- **Always add a constructor** that takes `Aggregate<YourAggregate>` and initializes the SRN from `aggregate.getId()`.
- Make the `srn` field `final` to ensure it's always initialized.
- Event handlers receive `Event<?, YourEvent>` wrappers, not raw event data.
- Access event data via `event.getData()`, metadata via `event.getEventTime()` and `event.getPrincipal()`.
- The SRN must be set in the constructor, not in event handlers, because projections need identity before processing events.

### Command Class Template

Commands use `Event<A, E>` from `com.source2sea.evento.aggregate` package for return types:

```java
package com.source2sea.iam.features.applications.aggregate.commands;

import com.source2sea.evento.aggregate.Event;  // ← For Event<A, E> return type
import com.source2sea.evento.aggregate.ReactiveAggregateRepository;
import reactor.core.publisher.Flux;

public class CreateApplication extends ReactiveApplicationCommand<ApplicationCreated> {
  
  @Override
  public Flux<Event<Application, ApplicationCreated>> execute() {
    return commitEvent();
  }
  
  private Flux<Event<Application, ApplicationCreated>> commitEvent() {
    return repository.getInstance(Application.class, application.srn())
                     .flatMapMany(a -> a.event(
                         tx -> tx.insert(/* ... */),
                         new ApplicationCreated(name, baseUrl)));
  }
}
```

**Important:** `commitEvent()` must return `Flux`, not `Mono`. Do not call `.next()` at the end.

### Synchronous List Updates Pattern

When using the **synchronous list update pattern** (Pattern B - EventTransaction), commands directly update list tables in the transaction:

```java
private Flux<Event<Application, ApplicationCreated>> commitEvent() {
  return repository.getInstance(Application.class, application.srn())
                   .flatMapMany(a -> a.event(
                       tx -> tx.insert(ApplicationListItem.builder()
                                                         .account(account.srn())
                                                         .applicationId(application.id())
                                                         .applicationSrn(application.srn())  // ← Full SRN
                                                         .name(name)
                                                         .baseUrl(baseUrl)
                                                         .createdAt(Instant.now())
                                                         .updatedAt(Instant.now())
                                                         .build()),
                       new ApplicationCreated(name, baseUrl)));  // ← Do NOT call .next()
}
```

**Key points:**
- The lambda `tx -> tx.insert(...)` provides access to the event transaction
- Use `tx.insert()`, `tx.update()`, or `tx.delete()` to modify list tables synchronously
- Always include the **full SRN** in list items (e.g., `applicationSrn` column) per CQRS skill requirement
- Return the full `Flux` from `flatMapMany` - do NOT call `.next()` to convert to `Mono`
- This ensures immediate consistency between event store and list tables

---

## Command Patterns

### Design Guidance: When to Use Each Pattern

#### **Standard CRUD Commands** (`Create`, `Update`, `Delete`)

Use for **primary lifecycle operations** on the aggregate:

- **Create**: Initial aggregate creation
- **Update**: General state modifications
- **Delete**: Aggregate removal

**Example Features**: Policies, Accounts, Clients, Roles

#### **Specialized Action Commands** (`Attach`, `Detach`, `Assign`, `Add`, `Remove`)

Use for **relationship management** or **specific state transitions**:

- **AttachPolicy/DetachPolicy**: Managing policy associations on accounts/clients/roles
- **AssignRole/UnassignRole**: User-to-role relationships
- **AddOidcIdentityProvider/RemoveOidcIdentityProvider**: Identity provider configuration
- **AssignIdentity/RemoveIdentity**: User identity management

**When to Create Specialized Commands:**

Create specialized commands when:
- The operation has specific business rules beyond simple updates
- The operation manages relationships between aggregates
- You want separate authorization controls for the action
- The operation has its own event type for audit/tracking

Do not create specialized commands when the operation is just a field update. Use the Update command instead.

---

### 1. Base Command Class

Every feature needs a **base command class** that provides:
- Repository reference
- Aggregate identifier (SRN)
- `commitEvent()` helper for event persistence
- Base `assertions()` for validation
- Existence check helpers (see section 2)

**Example (ReactivePolicyCommand):**

```java
@Accessors(fluent = true)
@Setter
abstract public class ReactivePolicyCommand<E> implements ReactiveCommand<Event<Policy, E>> {
  protected final ReactiveAggregateRepository repository;
  protected PolicySrn policy;
  
  public ReactivePolicyCommand(ReactiveAggregateRepository repository) {
    this.repository = repository;
  }
  
  protected Flux<Event<Policy, E>> commitEvent(E event) {
    return repository.getInstance(Policy.class, policy.srn())
                     .flatMapMany(a -> a.event(event));
  }
  
  protected void assertions() {
    Assert.notNull(repository, "repository must not be null");
    Assert.notNull(policy, "policy must not be null");
  }
}
```

---

### 2. Existence Checks in Base Command

Base command classes provide helper methods for existence checks that Update, Delete, and specialized commands use.

**Example (ReactiveAccountCommand):**

```java
protected Mono<Boolean> exists() {
  return materialize()
      .map(AccountDetails::isPresent);
}

protected Mono<AccountDetails> basicAccountData() {
  return materialize()
      .filter(AccountDetails::isPresent)
      .switchIfEmpty(Mono.error(new AccountDoesNotExist("account not found")));
}

protected Mono<AccountDetails> materialize() {
  return repository.getInstance(Account.class, account.srn())
      .flatMap(aggregate -> aggregate.materialize(AccountDetails.class));
}
```

**Usage Example (UpdateAccount):**

```java
@Override
public Flux<Event<Account, AccountChanged>> execute() {
  assertions();
  return basicAccountData()
      .filter(this::isChanged)
      .flatMapMany(ignored -> commitEvent(new AccountChanged(name, tags)));
}

protected boolean isChanged(AccountDetails details) {
  if (!Objects.equals(details.getName(), name)) return true;
  return !Objects.equals(details.getTags(), tags);
}
```

---

### 3. Create Command Pattern

Create commands generate new aggregate identifier (SRN), validate required fields, and emit Created event.

**Example (CreatePolicy):**

```java
@Accessors(fluent = true)
@Setter
public class CreatePolicy extends ReactivePolicyCommand<PolicyCreated> {
  private String name;
  private AccountSrn account;
  private String description;
  private List<PrincipalStatement> statements;
  
  public CreatePolicy(ReactiveAggregateRepository repository) {
    super(repository);
  }
  
  @Override
  public Flux<Event<Policy, PolicyCreated>> execute() {
    policy(new PolicySrn(account, name));
    assertions();
    return commitEvent(new PolicyCreated(description, statements));
  }
  
  @Override
  protected void assertions() {
    super.assertions();
    Assert.notNull(name, "name must not be null");
    Assert.notNull(account, "account must not be null");
    Assert.notNull(statements, "statements must not be null");
    Assert.isTrue(!statements.isEmpty(), "At least one policy statement must be provided");
  }
}
```

---

### 4. Update Command Pattern

Update commands require aggregate identifier, update mutable fields, and check existence before committing.

**Example (UpdatePolicy):**

```java
@Accessors(fluent = true)
@Setter
public class UpdatePolicy extends ReactivePolicyCommand<PolicyUpdated> {
  private String description;
  private List<PrincipalStatement> statements;
  
  public UpdatePolicy(ReactiveAggregateRepository repository) {
    super(repository);
  }
  
  @Override
  public UpdatePolicy policy(PolicySrn policy) {
    super.policy(policy);
    return this;
  }
  
  @Override
  public Flux<Event<Policy, PolicyUpdated>> execute() {
    assertions();
    return commitEvent(new PolicyUpdated(description, statements));
  }
  
  @Override
  protected void assertions() {
    super.assertions();
    Assert.notNull(statements, "statements must not be null");
    Assert.isTrue(!statements.isEmpty(), "At least one policy statement must be provided");
  }
}
```

---

### 5. Delete Command Pattern

Delete commands require only aggregate identifier. Event typically has no payload.

**Example (DeletePolicy):**

```java
@Accessors(fluent = true)
@Setter
public class DeletePolicy extends ReactivePolicyCommand<PolicyDeleted> {
  
  public DeletePolicy(ReactiveAggregateRepository repository) {
    super(repository);
  }
  
  @Override
  public DeletePolicy policy(PolicySrn policy) {
    super.policy(policy);
    return this;
  }
  
  @Override
  public Flux<Event<Policy, PolicyDeleted>> execute() {
    assertions();
    return commitEvent(new PolicyDeleted());
  }
}
```

---

### 6. Specialized Action Commands

Specialized commands manage relationships or specific state transitions beyond simple CRUD.

**Example (AttachPolicy on Account):**

```java
@Accessors(fluent = true)
@Setter
public class AttachPolicy extends ReactiveAccountCommand<PolicyAttached> {
  private PolicySrn policy;
  
  public AttachPolicy(ReactiveAggregateRepository repository) {
    super(repository);
  }
  
  @Override
  public AttachPolicy account(AccountSrn account) {
    super.account(account);
    return this;
  }
  
  @Override
  public Flux<Event<Account, PolicyAttached>> execute() {
    assertions();
    return commitEvent(new PolicyAttached(policy));
  }
  
  @Override
  protected void assertions() {
    super.assertions();
    Assert.notNull(policy, "policy must not be null");
  }
}
```

**Another Example (AssignRole to User):**

```java
@Accessors(fluent = true)
@Setter
public class AssignRole extends ReactiveUserCommand<RoleAssigned> {
  private RoleSrn role;
  
  public AssignRole(ReactiveAggregateRepository repository) {
    super(repository);
  }
  
  @Override
  public AssignRole user(UserSrn user) {
    super.user(user);
    return this;
  }
  
  @Override
  public Flux<Event<User, RoleAssigned>> execute() {
    assertions();
    return commitEvent(new RoleAssigned(role));
  }
  
  @Override
  protected void assertions() {
    super.assertions();
    Assert.notNull(role, "role must not be null");
  }
}
```

---

### 7. Multi-Event Commands

Some commands emit multiple events atomically. Evento's `event()` method accepts varargs `E... event` and applies all events in one transaction.

**Evento Signatures:**
```java
public <E> Flux<Event<A, E>> event(E... event)
public <E> Flux<Event<A, E>> event(Consumer<EventTransaction> txn, E... event)
```

**Example - Direct varargs (CreateUser):**

```java
return repository.getInstance(User.class, user.srn())
                 .flatMapMany(a -> a.event(
                     new UserCreated(email, name, picture),
                     new RoleAssigned(role1),
                     new RoleAssigned(role2),
                     new IdentityAssigned(identityProvider, subjectId, email)
                 ));
```

**Example - Array approach:**

```java
List<UserEvent> events = new ArrayList<>();
events.add(new UserCreated(email, name, picture));
roles.forEach(r -> events.add(new RoleAssigned(r)));
identities.forEach(i -> events.add(new IdentityAssigned(i)));

return repository.getInstance(User.class, user.srn())
                 .flatMapMany(a -> a.event(events.toArray(new UserEvent[0])));
```

---

### 8. Event Payload Design: Complete Data in Events

Events should contain all data relevant to the event, not just identifiers. This enables event handlers and projections to update read models without querying aggregates.

**Example (PolicyCreated):**

```java
@Event("iam:policy:PolicyCreated")
public class PolicyCreated {
  private String description;
  private List<PrincipalStatement> statements;  // Complete object graph
}

// PrincipalStatement contains nested structure
public class PrincipalStatement extends Statement {
  private String statementId;
  private Effect effect;
  private List<String> actions;
  private List<String> principals;
  private List<String> resources;
}
```

**Another Example (AccountCreated):**

```java
@Event("iam:account:AccountCreated")
public class AccountCreated {
  private AccountSrn parent;              // Full SRN object
  private String name;
  private Map<String, String> tags;       // Complete tag map
}
```

Events must contain complete data for the change to enable:
- Event handlers to update list tables without querying aggregates
- Projections to be built from event stream alone
- Audit trail to contain complete context of what changed
- Event replay to reconstruct aggregate state

---

### 9. Transactional List Updates

There are **two patterns** for updating Cassandra list tables:

#### **Pattern A: Asynchronous Event Handlers (Preferred)**

Most features use **event handlers** to update list tables asynchronously after events are committed.

Use this pattern when list updates are not time-critical. Asynchronous updates provide:
- Simpler implementation and better decoupling
- Better fault tolerance and retry capabilities
- No impact on command execution performance
- Eventual consistency is acceptable

Async is the preferred default pattern. Only use synchronous transactions when immediate consistency is required.

**Flow:**
1. Aggregate command commits event
2. Event published to messaging infrastructure
3. Event handler picks up event
4. Event handler invokes list command
5. List command writes to Cassandra list table

**Event Handler Example:**

```java
// In {project}-consumer/src/.../messaging/{Feature}ListFunctions.java (or -server/-application if no -consumer)
@Bean
public MessageMapper {feature}ListEventHandler() {
  return ReactiveMessageMapperFactory.forRoute("{feature}")
                                     .relaxed()
                                     .withErrorHandler(this::errorHandler)
                                     .handle({Feature}Created.class, {feature}CreatedEventHandler())
                                     .handle({Feature}Updated.class, {feature}UpdatedEventHandler())
                                     .handle({Feature}Deleted.class, {feature}DeletedEventHandler())
                                     .build();
}

private ReactiveEventHandler<{Feature}Created> {feature}CreatedEventHandler() {
  return event -> Flux.just(event)
                      .flatMap(e -> commandFactory.create(Add{Feature}ToList.class)
                                                  .{feature}(new {Feature}Srn(e.aggregate().getId()))
                                                  .description(e.getData().getDescription())
                                                  .relatedData(e.getData().getRelatedData())  // From event
                                                  .createdAt(e.eventTime())
                                                  .createdBy(e.principal())
                                                  .execute())
                      .then();
}
```

**Real Example (PolicyListFunctions):**

```java
@Bean
public MessageMapper policyListEventHandler() {
  return ReactiveMessageMapperFactory.forRoute("policy")
                                     .relaxed()
                                     .withErrorHandler(this::errorHandler)
                                     .handle(PolicyCreated.class, policyCreatedEventHandler())
                                     .handle(PolicyDeleted.class, policyDeletedEventHandler())
                                     .handle(PolicyUpdated.class, policyUpdatedEventHandler())
                                     .build();
}

private ReactiveEventHandler<PolicyCreated> policyCreatedEventHandler() {
  return event -> Flux.just(event)
                      .flatMap(e -> commandFactory.create(AddPolicyToList.class)
                                                  .policy(new PolicySrn(e.aggregate().getId()))
                                                  .statements(e.getData().getStatements())
                                                  .description(e.getData().getDescription())
                                                  .createdAt(e.eventTime())
                                                  .createdBy(e.principal())
                                                  .execute())
                      .then();
}
```

#### **Pattern B: Synchronous Transaction with EventTransaction**

For features requiring **immediate list updates**, use Evento's transactional API with `Consumer<EventTransaction> txn` parameter.

Use this pattern when:
- The entity must appear in list queries immediately after creation
- Users expect to list an entity right after creating it
- Strong consistency is required (e.g., account hierarchy where parent-child relationships must be immediately queryable)
- List updates must succeed or fail atomically with the event commit

Example use case: Creating an account and immediately querying the parent's child account list must show the new account.

**Evento Signature:**

```java
public <E> Flux<Event<A, E>> event(Consumer<EventTransaction> txn, E... event)
```

The `EventTransaction` provides methods to execute Cassandra operations atomically with event commits:
- `tx.insert(entity)` - Insert entity
- `tx.update(entity)` - Update entity  
- `tx.delete(entity)` - Delete entity
- `tx.execute(statement)` - Execute custom CQL statement

**Pattern:**

```java
protected Flux<Event<{Feature}, {Feature}Created>> commitEvent({Feature}Data data) {
  return repository.getInstance({Feature}.class, {feature}.srn())
                   .flatMapMany(a -> a.event(
                       tx -> {
                         // Cassandra operations execute in same transaction as event
                         tx.insert(new {Feature}ListItem(...));
                         // Can perform multiple operations
                         tx.insert(new RelatedItem(...));
                       },
                       new {Feature}Created(data)  // Event(s) committed atomically
                   ));
}
```

**Real Example (CreateAccount):**

```java
private Flux<Event<Account, AccountCreated>> commitEvent(String accountId) {
  AccountSrn srn = new AccountSrn(accountId);
  return repository.getInstance(Account.class, srn)
                   .flatMapMany(a -> a.event(
                       tx -> tx.insert(new AccountListItem(
                           parent.srn(),
                           srn.srn(),
                           name,
                           tags,
                           Instant.now(),
                           Instant.now(),
                           null)),
                       new AccountCreated(parent, name, tags)));
}
```

**Multi-Event with Transaction:**

You can combine multiple events with transactional list updates:

```java
return repository.getInstance({Feature}.class, {feature}.srn())
                 .flatMapMany(a -> a.event(
                     tx -> {
                       tx.insert(new {Feature}ListItem(...));
                       tx.insert(new RelatedItem(...));
                     },
                     new {Feature}Created(...),
                     new RelationEstablished(...),
                     new ItemAdded(...)
                 ));
```

All events and all transaction operations commit atomically or roll back together.

#### **Pattern B: Updating List Tables with Projection Data**

When using Pattern B for commands that **don't create the entity** (update, attach policy, etc.), you need to update the list table while preserving all existing fields. The solution is to use the **projection data you already loaded** for existence/validation checks.

**Problem:** ORM `save()` overwrites all fields, so partial updates would null out unset fields.

**Solution:** Pass the projection through and use its data to create a complete list item.

**Pattern:**

```java
@Override
public Flux<Event<Application, PolicyAttached>> execute() {
  assertions();
  return basicApplicationData()                    // Load projection for existence check
          .flatMap(this::ensurePolicyIsUnique)     // Validate (optional)
          .flatMapMany(details -> commitEvent(details));  // Pass projection to commit
}

private Flux<Event<Application, PolicyAttached>> commitEvent(ApplicationDetails details) {
  return repository.getInstance(Application.class, application.srn())
                   .flatMapMany(a -> a.event(
                       tx -> {
                         ApplicationListItem item = new ApplicationListItem();
                         item.setAccount(application.account().srn());
                         item.setApplicationId(application.id());
                         item.setApplicationSrn(application.srn());
                         item.setName(details.getName());           // From projection
                         item.setBaseUrl(details.getBaseUrl());     // From projection
                         item.setCreatedAt(details.getCreatedAt()); // From projection
                         item.setUpdatedAt(Instant.now());          // Update timestamp
                         item.setUpdatedBy(details.getUpdatedBy()); // From projection
                         tx.update(item);
                       },
                       new PolicyAttached(policy)));
}
```

**Key Points:**

1. **Reuse projection data** - Don't query the list table separately
2. **Complete object** - Set ALL fields to avoid nulls from ORM
3. **Update timestamp** - Only `updatedAt` gets current time
4. **Preserve audit fields** - `createdAt` and `updatedBy` come from projection

**For Update Commands with Optional Fields:**

```java
private Flux<Event<Application, ApplicationUpdated>> commitEvent(ApplicationDetails details) {
  return repository.getInstance(Application.class, application.srn())
                   .flatMapMany(a -> a.event(
                       tx -> {
                         ApplicationListItem item = new ApplicationListItem();
                         item.setAccount(application.account().srn());
                         item.setApplicationId(application.id());
                         item.setApplicationSrn(application.srn());
                         // Use new value if provided, otherwise keep existing from projection
                         item.setName(name != null ? name : details.getName());
                         item.setBaseUrl(baseUrl != null ? baseUrl : details.getBaseUrl());
                         item.setCreatedAt(details.getCreatedAt());
                         item.setUpdatedAt(Instant.now());
                         item.setUpdatedBy(details.getUpdatedBy());
                         tx.update(item);
                       },
                       new ApplicationUpdated(name, baseUrl)));
}
```

**Why This Works:**

- ✅ No additional database queries needed
- ✅ Projection has all the data we need
- ✅ ORM gets complete object (no fields left null)
- ✅ Only `updatedAt` changes
- ✅ Atomic with event commit

---

### 10. List Management Commands (Async Pattern Only)

List management commands are only needed when using **Pattern A: Asynchronous Event Handlers**. These commands handle Cassandra list table updates and serialize complex objects to BLOBs.

When using **Pattern B: Synchronous Transaction**, list updates happen directly in the transaction via `tx.insert()`, `tx.update()`, or `tx.delete()` without separate commands.

**Base Pattern:**

```java
public class Add{Feature}ToList extends List{Feature}Command<{Feature}ListItem> {
  private String description;
  private List<ComplexModel> complexData;
  private Instant createdAt;
  private String createdBy;
  
  public Add{Feature}ToList({Feature}ListRepository repository) {
    super(repository);
  }
  
  @Override
  public Mono<{Feature}ListItem> execute() {
    assertions();
    return Mono.just(complexData)
               .map(this::writeValueAsBytes)      // Serialize to JSON
               .map(ByteBuffer::wrap)             // Wrap as ByteBuffer
               .flatMap(buffer -> repository.save({Feature}ListItem.builder()
                                                                   .account({feature}.account().srn())
                                                                   .name({feature}.name())
                                                                   .description(description)
                                                                   .complexData(buffer)  // Store as BLOB
                                                                   .createdAt(createdAt)
                                                                   .updatedAt(createdAt)
                                                                   .updatedBy(createdBy)
                                                                   .build()));
  }
  
  // Jackson serialization helper
  @SneakyThrows
  private byte[] writeValueAsBytes(List<ComplexModel> data) {
    return objectMapper.writeValueAsBytes(data);
  }
}
```

**Real Example (AddPolicyToList):**

```java
public class AddPolicyToList extends ListPolicyCommand<PolicyListItem> {
  private String description;
  private List<PrincipalStatement> statements;
  private Instant createdAt;
  private String createdBy;
  
  public AddPolicyToList(PolicyListRepository policyListRepository) {
    super(policyListRepository);
  }
  
  @Override
  public Mono<PolicyListItem> execute() {
    assertions();
    return Mono.just(statements)
               .map(this::writeValueAsBytes)
               .map(ByteBuffer::wrap)
               .flatMap(b -> policyListRepository.save(PolicyListItem.builder()
                                                                     .account(policy.account().srn())
                                                                     .name(policy.name())
                                                                     .description(description)
                                                                     .statements(b)
                                                                     .createdAt(createdAt)
                                                                     .updatedBy(createdBy)
                                                                     .updatedAt(createdAt)
                                                                     .build()));
  }
  
  @SneakyThrows
  private byte[] writeValueAsBytes(List<PrincipalStatement> statements) {
    return objectMapper.writeValueAsBytes(statements);
  }
}
```

**Update Pattern:**

```java
public class Update{Feature}InList extends List{Feature}Command<{Feature}ListItem> {
  private String description;
  private List<ComplexModel> complexData;
  private Instant updatedAt;
  private String updatedBy;
  
  @Override
  public Mono<{Feature}ListItem> execute() {
    assertions();
    return repository.findByAccountAndName({feature}.account().srn(), {feature}.name())
                     .flatMap(item -> Mono.just(complexData)
                                          .map(this::writeValueAsBytes)
                                          .map(ByteBuffer::wrap)
                                          .map(buffer -> {
                                            item.setDescription(description);
                                            item.setComplexData(buffer);
                                            item.setUpdatedAt(updatedAt);
                                            item.setUpdatedBy(updatedBy);
                                            return item;
                                          })
                                          .flatMap(repository::save));
  }
}
```

**Remove Pattern:**

```java
public class Remove{Feature}FromList extends List{Feature}Command<Void> {
  
  @Override
  public Mono<Void> execute() {
    assertions();
    return repository.deleteByAccountAndName({feature}.account().srn(), {feature}.name());
  }
}
```

**List Repository (Spring Data Cassandra):**

```java
public interface {Feature}ListRepository extends ReactiveCassandraRepository<{Feature}ListItem, String> {
  Flux<{Feature}ListItem> findAllByAccount(String account);
  Mono<{Feature}ListItem> findByAccountAndName(String account, String name);
  Mono<Void> deleteByAccountAndName(String account, String name);
}
```

**List Entity (Cassandra Table):**

List entities must always include the full SRN as a regular column (not clustered). Use the ID or name as the clustering key.

```java
@Table("{feature}_list")
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class {Feature}ListItem {
  @PrimaryKeyColumn(name = "account", ordinal = 0, type = PrimaryKeyType.PARTITIONED)
  private String account;
  
  @PrimaryKeyColumn(name = "name", ordinal = 1, type = PrimaryKeyType.CLUSTERED)
  private String name;  // ID or name as clustering key
  
  @Column("{feature}_srn")
  private String {feature}Srn;  // Full SRN - always required as regular column
  
  @Column("description")
  private String description;
  
  @Column("complex_data")
  private ByteBuffer complexData;  // BLOB column for complex objects
  
  @Column("created_at")
  private Instant createdAt;
  
  @Column("updated_at")
  private Instant updatedAt;
  
  @Column("updated_by")
  private String updatedBy;
}
```

**Real Example (AccountListItem):**

```java
@Table("accounts")
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class AccountListItem {
  @PrimaryKeyColumn(type = PrimaryKeyType.PARTITIONED)
  private String parentSrn;
  
  @PrimaryKeyColumn(type = PrimaryKeyType.CLUSTERED, ordering = Ordering.ASCENDING)
  private String name;  // Name as clustering key
  
  @Column("account_srn")
  private String accountSrn;  // Full SRN as regular column
  
  private Map<String, String> tags;
  private Instant createdAt;
  private Instant updatedAt;
  private String updatedBy;
}
```

---

## Query Patterns

### 1. Single Item Query Pattern

Returns materialized projection of a single aggregate.

**Characteristics:**
- Requires aggregate identifier
- Returns `Mono<Projection>`
- Uses `aggregate.materialize(ProjectionClass.class)`

**Template:**

```java
@Accessors(fluent = true)
@Setter
@RequiredArgsConstructor
public class Get{Feature} implements ReactiveQuery<{Feature}Details> {
  private final ReactiveAggregateRepository repository;
  private {Feature}Srn {feature};
  
  @Override
  public Mono<{Feature}Details> execute() {
    return repository.getInstance({Feature}.class, {feature}.srn())
                     .flatMap(a -> a.materialize({Feature}Details.class));
  }
}
```

**Real Example (GetAccount):**

```java
@Accessors(fluent = true)
@Setter
@RequiredArgsConstructor
public class GetAccount implements ReactiveQuery<AccountDetails> {
  private final ReactiveAggregateRepository repository;
  private AccountSrn account;
  
  @Override
  public Mono<AccountDetails> execute() {
    return repository.getInstance(Account.class, account.srn())
                     .flatMap(a -> a.materialize(AccountDetails.class));
  }
}
```

---

### 2. List Query Pattern

Returns collection of items from read model (Cassandra list repository).

**Characteristics:**
- Uses list repository (Spring Data Cassandra)
- Filters by account/parent
- Returns `Flux<Model>`
- May combine multiple sources (list + managed)

**Template:**

```java
@Accessors(fluent = true)
@Setter
public class List{Features} implements ReactiveQuery<{Feature}> {
  private final {Feature}ListRepository {feature}ListRepository;
  private AccountSrn account;
  
  public List{Features}({Feature}ListRepository {feature}ListRepository) {
    Assert.notNull({feature}ListRepository, "{feature}ListRepository must not be null");
    this.{feature}ListRepository = {feature}ListRepository;
  }
  
  @Override
  public Flux<{Feature}> execute() {
    assertions();
    return {feature}ListRepository.findAllByAccount(account.srn())
                                  .map(item -> {Feature}.builder()
                                                        .srn(item.getSrn())
                                                        .name(item.getName())
                                                        .build());
  }
  
  protected void assertions() {
    Assert.notNull({feature}ListRepository, "{feature}ListRepository must not be null");
    Assert.notNull(account, "account must not be null");
  }
}
```

**Real Example (ListAccounts):**

```java
@Accessors(fluent = true)
@Setter
public class ListAccounts implements ReactiveQuery<AccountListItem> {
  private final AccountListRepository accountListRepository;
  private AccountSrn parent;
  
  public ListAccounts(AccountListRepository accountListRepository) {
    this.accountListRepository = accountListRepository;
  }
  
  @Override
  public Flux<AccountListItem> execute() {
    assertions();
    return accountListRepository.findAllByParentSrn(parent.srn());
  }
  
  protected void assertions() {
    Assert.notNull(accountListRepository, "accountListRepository must not be null");
    Assert.notNull(parent, "parent must not be null");
  }
}
```

**Real Example with Multiple Sources (ListPolicies):**

```java
@Accessors(fluent = true)
@Setter
public class ListPolicies implements ReactiveQuery<Policy> {
  private final ManagedPolicyRepository managedPolicyRepository;
  private final PolicyListRepository policyListRepository;
  private static final ObjectMapper objectMapper = createObjectMapper();
  
  private AccountSrn account;
  
  public ListPolicies(ManagedPolicyRepository managedPolicyRepository,
                      PolicyListRepository policyListRepository) {
    Assert.notNull(policyListRepository, "policyListRepository must not be null");
    Assert.notNull(managedPolicyRepository, "managedPolicyRepository must not be null");
    
    this.managedPolicyRepository = managedPolicyRepository;
    this.policyListRepository = policyListRepository;
  }
  
  @Override
  public Flux<Policy> execute() {
    return Mono.justOrEmpty(this.account)
               .flatMapMany(a -> policyListRepository.findAllByAccount(a.srn())
                                                     .map(p -> Policy.builder()
                                                                     .srn(new PolicySrn(new AccountSrn(p.getAccount()),
                                                                                        p.getName()).srn())
                                                                     .name(p.getName())
                                                                     .description(p.getDescription())
                                                                     .statements(readStatements(p))
                                                                     .build()))
               .switchIfEmpty(Flux.empty())
               .concatWith(managedPolicyRepository.findAll());
  }
  
  @SneakyThrows
  private List<PrincipalStatement> readStatements(PolicyListItem p) {
    return objectMapper.readValue(p.getStatements().array(), new TypeReference<>() {});
  }
}
```

---

### 3. History Query Pattern

Returns event stream for audit/debugging.

**Characteristics:**
- Returns raw events: `Flux<Event>`
- Retrieves complete aggregate history
- No projection/transformation

**Template:**

```java
@Accessors(fluent = true)
@Setter
@RequiredArgsConstructor
public class Get{Feature}History implements ReactiveQuery<Event> {
  private final ReactiveAggregateRepository repository;
  private {Feature}Srn {feature};
  
  @Override
  public Flux<Event> execute() {
    return repository.getInstance({Feature}.class, {feature}.srn())
                     .flatMapMany(Aggregate::events);
  }
}
```

**Real Example (GetPolicyHistory):**

```java
@Accessors(fluent = true)
@Setter
@RequiredArgsConstructor
public class GetPolicyHistory implements ReactiveQuery<Event> {
  private final ReactiveAggregateRepository repository;
  private PolicySrn policy;
  
  @Override
  public Flux<Event> execute() {
    return repository.getInstance(Policy.class, policy.srn())
                     .flatMapMany(ReactiveAggregate::events)
                     .switchIfEmpty(Mono.error(new PolicyNotFoundException("Policy not found")));
  }
}
```

**Key Points:**
- Use `ReactiveAggregate::events` method reference (not `Aggregate::events`)
- Add `.switchIfEmpty(Mono.error(...))` to return 404 when aggregate doesn't exist
- Use the appropriate `{Feature}NotFoundException` exception for your domain
- The events stream is empty when the aggregate is not found, so `switchIfEmpty` triggers the error

---

## Controller Integration

### CQRS in Controllers

Controllers orchestrate commands and queries. **See the spring-rest-api skill** for:
- Controller structure and spec/implementation separation
- Dual endpoint pattern (events + model responses)
- Helper method organization (command builders, projection loaders)
- OpenAPI annotations and authorization
- Request/Response DTOs with validation
- Converter patterns
- Complete endpoint examples

### CQRS-Specific Controller Patterns

#### **Command Builder Helper**

```java
private Create{Feature} create{Feature}(AccountSrn account, Create{Feature}Request request) {
  return commandFactory.create(Create{Feature}.class)
                       .account(account)
                       .name(request.getName())
                       .description(request.getDescription());
}
```

#### **Projection Loader Helper**

```java
private Mono<{Feature}MutationResponse> get{Feature}(
    {Feature}Srn srn,
    Collection<? extends Event<{Feature}, ?>> events) {
  return queryFactory.create(Get{Feature}.class)
                     .{feature}(srn)
                     .execute()
                     .mapNotNull(converter::convert)
                     .map(response -> new {Feature}MutationResponse(
                         response,
                         {Feature}ResponseMetadata.builder().events(events).build()));
}
```

#### **Event Endpoint (Direct Command Execution)**

```java
@PostMapping(produces = APPLICATION_VND_SOURCE2SEA_EVENT_JSON_VALUE)
@ResponseStatus(HttpStatus.CREATED)
public Flux<Event<{Feature}, {Feature}Created>> create{Feature}AsEvents(
    @ModelAttribute AccountSrn account,
    @RequestBody @Valid Create{Feature}Request request) {
  return create{Feature}(account, request).execute();
}
```

#### **Model Endpoint (Query After Command)**

```java
@PostMapping(produces = APPLICATION_VND_SOURCE2SEA_JSON_VALUE)
@ResponseStatus(HttpStatus.CREATED)
public Mono<{Feature}MutationResponse> create{Feature}AsModel(
    @ModelAttribute AccountSrn account,
    @RequestBody @Valid Create{Feature}Request request) {
  return create{Feature}(account, request)
      .execute()
      .collectList()
      .flatMap(events -> get{Feature}(extract{Feature}Srn(events), events));
}
```

Never construct post-mutation model from request DTO. Always query after command to get authoritative projection.

---

## Validation & Error Handling

### 1. Command Validation (Spring Assert)

Commands use **Spring's `Assert` utility** in `assertions()` methods:

```java
@Override
protected void assertions() {
  super.assertions();
  
  // Null checks
  Assert.notNull(name, "name must not be null");
  Assert.notNull(account, "account must not be null");
  
  // Collection validation
  Assert.notNull(statements, "statements must not be null");
  Assert.isTrue(!statements.isEmpty(), "At least one statement must be provided");
  
  // Conditional validation
  Assert.isTrue(priority >= 1 && priority <= 5, "priority must be between 1 and 5");
  
  // String validation
  Assert.hasText(email, "email must not be blank");
}
```

**Common Assert Methods:**
- `Assert.notNull(value, message)` - Value must not be null
- `Assert.isTrue(condition, message)` - Condition must be true
- `Assert.hasText(string, message)` - String must have content
- `Assert.notEmpty(collection, message)` - Collection must not be empty
- `Assert.state(condition, message)` - State validation

---

### 2. Request Validation (Jakarta Bean Validation)

Controllers automatically validate request DTOs marked with `@Valid`:

```java
@PostMapping
public Mono<{Feature}Response> create(
    @RequestBody @Valid Create{Feature}Request request) {
  // If validation fails, Spring returns 400 Bad Request automatically
}
```

**Validation triggers `IllegalArgumentException` if failed**, which is caught by `ControllerAdvice`.

---

### 3. Global Exception Handling

Add exception handlers to `ControllerAdvice.java`:

**Template:**

```java
@Slf4j
@RestControllerAdvice
public class ControllerAdvice implements IamSecurityControllerAdvice, ProblemHandling {
  
  @ExceptionHandler({Feature}NotFoundException.class)
  public ErrorResponse errorResponse{Feature}NotFound({Feature}NotFoundException e) {
    return ErrorResponse.create(e, HttpStatus.NOT_FOUND, e.getMessage());
  }
  
  @ExceptionHandler({Feature}AlreadyExistsException.class)
  public ErrorResponse errorResponse{Feature}AlreadyExists({Feature}AlreadyExistsException e) {
    return ErrorResponse.create(e, HttpStatus.CONFLICT, e.getMessage());
  }
  
  @ExceptionHandler(IllegalArgumentException.class)
  public ErrorResponse errorResponseIllegalArgumentException(IllegalArgumentException e) {
    return ErrorResponse.create(e, HttpStatus.BAD_REQUEST, e.getMessage());
  }
}
```

**Real Examples:**

```java
@ExceptionHandler(AccountAttachmentAlreadyExistsException.class)
public ErrorResponse errorResponseAccountAttachmentAlreadyExistsException(
    AccountAttachmentAlreadyExistsException e) {
  return ErrorResponse.create(e, HttpStatus.CONFLICT, e.getMessage());
}

@ExceptionHandler(UserDoesNotExist.class)
public ErrorResponse errorResponseUserNotFound(UserDoesNotExist e) {
  return ErrorResponse.create(e, HttpStatus.NOT_FOUND, e.getMessage());
}

@ExceptionHandler(PolicyNotFoundException.class)
public ErrorResponse errorResponsePolicyNotFound(PolicyNotFoundException e) {
  return ErrorResponse.create(e, HttpStatus.NOT_FOUND, e.getMessage());
}
```

---

### 4. Custom Exception Classes

Create domain-specific exceptions:

**Template:**

```java
public class {Feature}NotFoundException extends RuntimeException {
  public {Feature}NotFoundException(String message) {
    super(message);
  }
  
  public {Feature}NotFoundException({Feature}Srn srn) {
    super(String.format("{Feature} not found: %s", srn.srn()));
  }
}
```

**Real Example (UserDoesNotExist):**

```java
public class UserDoesNotExist extends RuntimeException {
  public UserDoesNotExist(String message) {
    super(message);
  }
}
```

---

### 5. @ModelAttribute for SRN Binding

`ControllerAdvice` can define `@ModelAttribute` methods to automatically construct SRNs from path variables and authentication context:

```java
@ModelAttribute
public AccountSrn accountSrn(@AuthenticationPrincipal final IamUser iamUser) {
  if (iamUser != null) {
    return iamUser.account();
  }
  return null;
}

@ModelAttribute
public {Feature}Srn {feature}Srn(
    @AuthenticationPrincipal final IamUser iamUser,
    @PathVariable(required = false) String {feature}Id) {
  if ({feature}Id == null) {
    return null;
  }
  return new {Feature}Srn(iamUser.account(), {feature}Id);
}
```

**Real Example:**

```java
@ModelAttribute
public PolicySrn policySrn(
    @AuthenticationPrincipal final IamUser iamUser,
    @PathVariable(required = false) String policyId) {
  if (policyId == null) {
    return null;
  }
  return new PolicySrn(iamUser.account(), policyId);
}
```

---

## Naming Conventions

### Commands
- **Format:** `{Verb}{Noun}`
- **Examples:** `CreatePolicy`, `UpdateAccount`, `AttachPolicy`, `AssignRole`, `DeleteClient`

### Queries
- **Format:** `{Verb}{Feature}` or `List{Features}`
- **Examples:** `GetAccount`, `ListPolicies`, `GetUserDetails`, `GetPolicyHistory`

### Events
- **Format:** `{Feature}{PastTenseVerb}`
- **Examples:** `PolicyCreated`, `AccountUpdated`, `UserDeleted`, `RoleAttached`, `PolicyDetached`

### Controllers
- **Format:** Plural feature name
- **Examples:** `Accounts`, `Policies`, `Clients`, `Roles`, `Users`, `Tags`

### Request DTOs
- **Format:** `{Verb}{Feature}Request`
- **Examples:** `CreatePolicyRequest`, `UpdateAccountRequest`, `AttachPolicyRequest`

### Response DTOs
- **Simple:** `{Feature}Response`
- **Mutation:** `{Feature}MutationResponse`
- **Extended:** `Extended{Feature}Response`
- **Metadata:** `{Feature}ResponseMetadata`
- **Examples:** `PolicyResponse`, `AccountMutationResponse`, `ExtendedAccountResponse`, `UserResponseMetadata`

### Repositories
- **List:** `{Feature}ListRepository`
- **Managed:** `Managed{Feature}Repository`
- **Examples:** `PolicyListRepository`, `AccountListRepository`, `ManagedPolicyRepository`

### Converters
- **Format:** `{Source}To{Target}Converter`
- **Examples:** `PolicyToPolicyResponseConverter`, `AccountToAccountResponseConverter`

### Projections
- **Format:** `{Feature}Details`
- **Examples:** `AccountDetails`, `UserDetails`, `PolicyDetails`

### Exceptions
- **Format:** `{Feature}{ErrorCondition}Exception`
- **Examples:** `PolicyNotFoundException`, `AccountAlreadyExistsException`, `UserDoesNotExist`

---

## Complete Feature Implementation Example

Let's implement a complete "Tags" feature following all patterns:

### Step 1: Domain Model (-internals)

#### **1.1: Base Command**

```java
package com.source2sea.iam.features.tags.aggregate.commands;

import com.source2sea.commons.cqrs.ReactiveCommand;
import com.source2sea.evento.aggregate.ReactiveAggregateRepository;
import com.source2sea.evento.aggregate.Event;
import com.source2sea.iam.features.tags.aggregate.Tag;
import com.source2sea.iam.features.tags.models.TagSrn;
import lombok.Setter;
import lombok.experimental.Accessors;
import org.springframework.util.Assert;
import reactor.core.publisher.Flux;

@Accessors(fluent = true)
@Setter
abstract public class ReactiveTagCommand<E> implements ReactiveCommand<Event<Tag, E>> {
  protected final ReactiveAggregateRepository repository;
  protected TagSrn tag;
  
  public ReactiveTagCommand(ReactiveAggregateRepository repository) {
    this.repository = repository;
  }
  
  protected Flux<Event<Tag, E>> commitEvent(E event) {
    return repository.getInstance(Tag.class, tag.srn())
                     .flatMapMany(a -> a.event(event));
  }
  
  protected void assertions() {
    Assert.notNull(repository, "repository must not be null");
    Assert.notNull(tag, "tag must not be null");
  }
}
```

#### **1.2: Create Command**

```java
package com.source2sea.iam.features.tags.aggregate.commands;

import com.source2sea.evento.aggregate.ReactiveAggregateRepository;
import com.source2sea.evento.aggregate.Event;
import com.source2sea.iam.features.tags.aggregate.Tag;
import com.source2sea.iam.features.tags.aggregate.events.TagCreated;
import com.source2sea.iam.features.tags.models.TagSrn;
import com.source2sea.iam.models.AccountSrn;
import lombok.Setter;
import lombok.experimental.Accessors;
import org.springframework.util.Assert;
import reactor.core.publisher.Flux;

@Accessors(fluent = true)
@Setter
public class CreateTag extends ReactiveTagCommand<TagCreated> {
  private String name;
  private AccountSrn account;
  private String description;
  private String color;
  
  public CreateTag(ReactiveAggregateRepository repository) {
    super(repository);
  }
  
  @Override
  public Flux<Event<Tag, TagCreated>> execute() {
    tag(new TagSrn(account, name));
    assertions();
    return commitEvent(new TagCreated(description, color));
  }
  
  @Override
  protected void assertions() {
    super.assertions();
    Assert.notNull(name, "name must not be null");
    Assert.notNull(account, "account must not be null");
    Assert.hasText(name, "name must not be blank");
  }
}
```

#### **1.3: Update Command**

```java
package com.source2sea.iam.features.tags.aggregate.commands;

import com.source2sea.evento.aggregate.ReactiveAggregateRepository;
import com.source2sea.evento.aggregate.Event;
import com.source2sea.iam.features.tags.aggregate.Tag;
import com.source2sea.iam.features.tags.aggregate.events.TagUpdated;
import com.source2sea.iam.features.tags.models.TagSrn;
import lombok.Setter;
import lombok.experimental.Accessors;
import reactor.core.publisher.Flux;

@Accessors(fluent = true)
@Setter
public class UpdateTag extends ReactiveTagCommand<TagUpdated> {
  private String description;
  private String color;
  
  public UpdateTag(ReactiveAggregateRepository repository) {
    super(repository);
  }
  
  @Override
  public UpdateTag tag(TagSrn tag) {
    super.tag(tag);
    return this;
  }
  
  @Override
  public Flux<Event<Tag, TagUpdated>> execute() {
    assertions();
    return commitEvent(new TagUpdated(description, color));
  }
}
```

#### **1.4: Delete Command**

```java
package com.source2sea.iam.features.tags.aggregate.commands;

import com.source2sea.evento.aggregate.ReactiveAggregateRepository;
import com.source2sea.evento.aggregate.Event;
import com.source2sea.iam.features.tags.aggregate.Tag;
import com.source2sea.iam.features.tags.aggregate.events.TagDeleted;
import com.source2sea.iam.features.tags.models.TagSrn;
import lombok.Setter;
import lombok.experimental.Accessors;
import reactor.core.publisher.Flux;

@Accessors(fluent = true)
@Setter
public class DeleteTag extends ReactiveTagCommand<TagDeleted> {
  
  public DeleteTag(ReactiveAggregateRepository repository) {
    super(repository);
  }
  
  @Override
  public DeleteTag tag(TagSrn tag) {
    super.tag(tag);
    return this;
  }
  
  @Override
  public Flux<Event<Tag, TagDeleted>> execute() {
    assertions();
    return commitEvent(new TagDeleted());
  }
}
```

#### **1.5: Queries**

```java
// GetTag.java
package com.source2sea.iam.features.tags.aggregate.queries;

import com.source2sea.commons.cqrs.ReactiveQuery;
import com.source2sea.evento.aggregate.ReactiveAggregateRepository;
import com.source2sea.iam.features.tags.aggregate.Tag;
import com.source2sea.iam.features.tags.aggregate.projections.TagDetails;
import com.source2sea.iam.features.tags.models.TagSrn;
import lombok.RequiredArgsConstructor;
import lombok.Setter;
import lombok.experimental.Accessors;
import reactor.core.publisher.Mono;

@Accessors(fluent = true)
@Setter
@RequiredArgsConstructor
public class GetTag implements ReactiveQuery<TagDetails> {
  private final ReactiveAggregateRepository repository;
  private TagSrn tag;
  
  @Override
  public Mono<TagDetails> execute() {
    return repository.getInstance(Tag.class, tag.srn())
                     .flatMap(a -> a.materialize(TagDetails.class));
  }
}

// ListTags.java
package com.source2sea.iam.features.tags.list.queries;

import com.source2sea.commons.cqrs.ReactiveQuery;
import com.source2sea.iam.features.tags.list.repositories.TagListRepository;
import com.source2sea.iam.features.tags.models.TagModel;
import com.source2sea.iam.models.AccountSrn;
import lombok.Setter;
import lombok.experimental.Accessors;
import org.springframework.util.Assert;
import reactor.core.publisher.Flux;

@Accessors(fluent = true)
@Setter
public class ListTags implements ReactiveQuery<TagModel> {
  private final TagListRepository tagListRepository;
  private AccountSrn account;
  
  public ListTags(TagListRepository tagListRepository) {
    Assert.notNull(tagListRepository, "tagListRepository must not be null");
    this.tagListRepository = tagListRepository;
  }
  
  @Override
  public Flux<TagModel> execute() {
    assertions();
    return tagListRepository.findAllByAccount(account.srn())
                            .map(item -> TagModel.builder()
                                                 .srn(item.getSrn())
                                                 .name(item.getName())
                                                 .description(item.getDescription())
                                                 .color(item.getColor())
                                                 .build());
  }
  
  protected void assertions() {
    Assert.notNull(tagListRepository, "tagListRepository must not be null");
    Assert.notNull(account, "account must not be null");
  }
}

// GetTagHistory.java
package com.source2sea.iam.features.tags.queries;

import com.source2sea.commons.cqrs.ReactiveQuery;
import com.source2sea.evento.aggregate.Aggregate;
import com.source2sea.evento.aggregate.ReactiveAggregateRepository;
import com.source2sea.evento.aggregate.Event;
import com.source2sea.iam.features.tags.aggregate.Tag;
import com.source2sea.iam.features.tags.models.TagSrn;
import lombok.RequiredArgsConstructor;
import lombok.Setter;
import lombok.experimental.Accessors;
import reactor.core.publisher.Flux;

@Accessors(fluent = true)
@Setter
@RequiredArgsConstructor
public class GetTagHistory implements ReactiveQuery<Event> {
  private final ReactiveAggregateRepository repository;
  private TagSrn tag;
  
  @Override
  public Flux<Event> execute() {
    return repository.getInstance(Tag.class, tag.srn())
                     .flatMapMany(Aggregate::events);
  }
}
```

---

### Step 2: API Layer (-server or -application)

#### **2.1: Request DTOs**

```java
// CreateTagRequest.java
package com.source2sea.iam.application.controllers.models.tags;

import io.swagger.v3.oas.annotations.media.Schema;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Pattern;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class CreateTagRequest {
  @NotBlank
  @Pattern(regexp = "^[:/0-9a-zA-Z_-]{3,}$")
  @Schema(description = "Tag name", example = "production")
  private String name;
  
  @Schema(description = "Tag description", example = "Production environment tag")
  private String description;
  
  @Pattern(regexp = "^#[0-9A-Fa-f]{6}$")
  @Schema(description = "Tag color in hex format", example = "#FF5733")
  private String color;
}

// UpdateTagRequest.java
package com.source2sea.iam.application.controllers.models.tags;

import io.swagger.v3.oas.annotations.media.Schema;
import jakarta.validation.constraints.Pattern;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class UpdateTagRequest {
  @Schema(description = "Updated description")
  private String description;
  
  @Pattern(regexp = "^#[0-9A-Fa-f]{6}$")
  @Schema(description = "Tag color in hex format", example = "#3498DB")
  private String color;
}
```

#### **2.2: Response DTOs**

```java
// TagResponse.java
package com.source2sea.iam.application.controllers.models.tags;

import io.swagger.v3.oas.annotations.media.Schema;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class TagResponse {
  @Schema(description = "Source2Sea Resource Name")
  private String srn;
  
  @Schema(description = "Tag name")
  private String name;
  
  @Schema(description = "Tag description")
  private String description;
  
  @Schema(description = "Tag color")
  private String color;
}

// TagMutationResponse.java
package com.source2sea.iam.application.controllers.models.tags;

import lombok.AllArgsConstructor;
import lombok.Data;

@Data
@AllArgsConstructor
public class TagMutationResponse {
  private TagResponse tag;
  private TagResponseMetadata metadata;
}

// TagResponseMetadata.java
package com.source2sea.iam.application.controllers.models.tags;

import com.source2sea.evento.aggregate.Event;
import com.source2sea.iam.features.tags.aggregate.Tag;
import lombok.Builder;
import lombok.Data;

import java.util.Collection;

@Data
@Builder
public class TagResponseMetadata {
  private Collection<? extends Event<Tag, ?>> events;
}
```

#### **2.3: Converter**

```java
package com.source2sea.iam.application.controllers.converters;

import com.source2sea.iam.application.controllers.models.tags.TagResponse;
import com.source2sea.iam.features.tags.models.TagModel;

public class TagToTagResponseConverter {
  
  public TagResponse convert(TagModel tag) {
    if (tag == null) {
      return null;
    }
    
    return TagResponse.builder()
                      .srn(tag.getSrn())
                      .name(tag.getName())
                      .description(tag.getDescription())
                      .color(tag.getColor())
                      .build();
  }
}
```

#### **2.4: Controller**

```java
package com.source2sea.iam.application.controllers;

import com.source2sea.commons.cqrs.CommandFactory;
import com.source2sea.commons.cqrs.QueryFactory;
import com.source2sea.evento.aggregate.Event;
import com.source2sea.iam.application.controllers.converters.TagToTagResponseConverter;
import com.source2sea.iam.application.controllers.models.tags.*;
import com.source2sea.iam.client.annotations.IamRequireAction;
import com.source2sea.iam.features.tags.aggregate.Tag;
import com.source2sea.iam.features.tags.aggregate.commands.CreateTag;
import com.source2sea.iam.features.tags.aggregate.commands.DeleteTag;
import com.source2sea.iam.features.tags.aggregate.commands.UpdateTag;
import com.source2sea.iam.features.tags.aggregate.events.TagCreated;
import com.source2sea.iam.features.tags.aggregate.events.TagDeleted;
import com.source2sea.iam.features.tags.aggregate.events.TagUpdated;
import com.source2sea.iam.features.tags.aggregate.projections.TagDetails;
import com.source2sea.iam.features.tags.aggregate.queries.GetTag;
import com.source2sea.iam.features.tags.list.queries.ListTags;
import com.source2sea.iam.features.tags.models.TagSrn;
import com.source2sea.iam.features.tags.queries.GetTagHistory;
import com.source2sea.iam.models.AccountSrn;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.media.Content;
import io.swagger.v3.oas.annotations.media.Schema;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.responses.ApiResponses;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.*;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

import java.util.Collection;

import static com.source2sea.iam.application.controllers.MediaTypes.APPLICATION_VND_SOURCE2SEA_EVENT_JSON_VALUE;
import static com.source2sea.iam.application.controllers.MediaTypes.APPLICATION_VND_SOURCE2SEA_JSON_VALUE;

@RestController
@RequiredArgsConstructor
@RequestMapping("/iam/tags")
public class Tags {
  private final CommandFactory commandFactory;
  private final QueryFactory queryFactory;
  private final TagToTagResponseConverter tagConverter = new TagToTagResponseConverter();
  
  // ========== HELPER METHODS ==========
  
  private CreateTag createTag(AccountSrn account, CreateTagRequest request) {
    return commandFactory.create(CreateTag.class)
                         .account(account)
                         .name(request.getName())
                         .description(request.getDescription())
                         .color(request.getColor());
  }
  
  private UpdateTag updateTag(TagSrn tagSrn, UpdateTagRequest request) {
    return commandFactory.create(UpdateTag.class)
                         .tag(tagSrn)
                         .description(request.getDescription())
                         .color(request.getColor());
  }
  
  private DeleteTag deleteTag(TagSrn tagSrn) {
    return commandFactory.create(DeleteTag.class)
                         .tag(tagSrn);
  }
  
  private Mono<TagMutationResponse> getTag(
      TagSrn tagSrn,
      Collection<? extends Event<Tag, ?>> events) {
    return getTagDetails(tagSrn)
        .mapNotNull(tagConverter::convert)
        .map(response -> new TagMutationResponse(
            response,
            TagResponseMetadata.builder()
                .events(events)
                .build()));
  }
  
  private Mono<TagDetails> getTagDetails(TagSrn tagSrn) {
    return queryFactory.create(GetTag.class)
                       .tag(tagSrn)
                       .execute();
  }
  
  // ========== ENDPOINTS ==========
  
  @GetMapping(produces = APPLICATION_VND_SOURCE2SEA_JSON_VALUE)
  @Operation(summary = "List tags", description = "Returns all tags for the authenticated account")
  @IamRequireAction(group = "iam", actions = "ListTags")
  public Flux<TagResponse> list(@ModelAttribute AccountSrn account) {
    return queryFactory.create(ListTags.class)
                       .account(account)
                       .execute()
                       .mapNotNull(tagConverter::convert);
  }
  
  @PostMapping(produces = APPLICATION_VND_SOURCE2SEA_EVENT_JSON_VALUE)
  @ResponseStatus(HttpStatus.CREATED)
  @Operation(summary = "Create tag (events)", description = "Create a new tag and return event stream")
  @ApiResponses({
      @ApiResponse(responseCode = "201", description = "Tag created successfully")
  })
  @IamRequireAction(group = "iam", actions = "CreateTag")
  public Flux<Event<Tag, TagCreated>> createTagAsEvents(
      @ModelAttribute AccountSrn account,
      @RequestBody @Valid CreateTagRequest request) {
    return createTag(account, request).execute();
  }
  
  @PostMapping(produces = APPLICATION_VND_SOURCE2SEA_JSON_VALUE)
  @ResponseStatus(HttpStatus.CREATED)
  @Operation(summary = "Create tag", description = "Create a new tag and return the latest projection")
  @ApiResponses({
      @ApiResponse(responseCode = "201", description = "Tag created successfully",
          content = @Content(schema = @Schema(implementation = TagMutationResponse.class)))
  })
  @IamRequireAction(group = "iam", actions = "CreateTag")
  public Mono<TagMutationResponse> createTagAsModel(
      @ModelAttribute AccountSrn account,
      @RequestBody @Valid CreateTagRequest request) {
    return createTag(account, request)
        .execute()
        .collectList()
        .flatMap(events -> getTag(
            new TagSrn(events.stream().findAny().orElseThrow()
                      .getAggregate().getId()),
            events));
  }
  
  @GetMapping(value = "/{tagId}", produces = APPLICATION_VND_SOURCE2SEA_JSON_VALUE)
  @Operation(summary = "Get tag", description = "Returns the latest projection of the tag")
  @IamRequireAction(group = "iam", actions = "GetTag")
  public Mono<TagResponse> getTagAsModel(
      @PathVariable String tagId,
      @ModelAttribute TagSrn tagSrn) {
    return getTagDetails(tagSrn)
        .mapNotNull(tagConverter::convert);
  }
  
  @GetMapping(value = "/{tagId}", produces = APPLICATION_VND_SOURCE2SEA_EVENT_JSON_VALUE)
  @Operation(summary = "Get tag history", description = "Returns the complete event stream")
  @IamRequireAction(group = "iam", actions = "GetTag")
  public Flux<Event> getTagAsEvents(
      @PathVariable String tagId,
      @ModelAttribute TagSrn tagSrn) {
    return queryFactory.create(GetTagHistory.class)
                       .tag(tagSrn)
                       .execute();
  }
  
  @PostMapping(value = "/{tagId}", produces = APPLICATION_VND_SOURCE2SEA_EVENT_JSON_VALUE)
  @Operation(summary = "Update tag (events)")
  @IamRequireAction(group = "iam", actions = "UpdateTag")
  public Flux<Event<Tag, TagUpdated>> updateTagAsEvents(
      @PathVariable String tagId,
      @ModelAttribute TagSrn tagSrn,
      @RequestBody @Valid UpdateTagRequest request) {
    return updateTag(tagSrn, request).execute();
  }
  
  @PostMapping(value = "/{tagId}", produces = APPLICATION_VND_SOURCE2SEA_JSON_VALUE)
  @Operation(summary = "Update tag")
  @IamRequireAction(group = "iam", actions = "UpdateTag")
  public Mono<TagMutationResponse> updateTagAsModel(
      @PathVariable String tagId,
      @ModelAttribute TagSrn tagSrn,
      @RequestBody @Valid UpdateTagRequest request) {
    return updateTag(tagSrn, request)
        .execute()
        .collectList()
        .flatMap(events -> getTag(tagSrn, events));
  }
  
  @DeleteMapping(value = "/{tagId}", produces = APPLICATION_VND_SOURCE2SEA_EVENT_JSON_VALUE)
  @Operation(summary = "Delete tag")
  @IamRequireAction(group = "iam", actions = "DeleteTag")
  public Flux<Event<Tag, TagDeleted>> deleteTagAsEvents(
      @PathVariable String tagId,
      @ModelAttribute TagSrn tagSrn) {
    return deleteTag(tagSrn).execute();
  }
}
```

#### **2.5: Exception Handling**

Add to `ControllerAdvice.java`:

```java
@ExceptionHandler(TagNotFoundException.class)
public ErrorResponse errorResponseTagNotFound(TagNotFoundException e) {
  return ErrorResponse.create(e, HttpStatus.NOT_FOUND, e.getMessage());
}

@ModelAttribute
public TagSrn tagSrn(
    @AuthenticationPrincipal final IamUser iamUser,
    @PathVariable(required = false) String tagId) {
  if (tagId == null) {
    return null;
  }
  return new TagSrn(iamUser.account(), tagId);
}
```

---

## Testing Patterns

### Command Tests

```java
@Test
void testCreate{Feature}() {
  Create{Feature} command = new Create{Feature}(repository)
      .account(new AccountSrn("srn:iam:test:account/test"))
      .name("test-{feature}")
      .description("Test description");
  
  StepVerifier.create(command.execute())
              .assertNext(event -> {
                assertThat(event.getType()).isEqualTo({Feature}Created.class);
                assertThat(event.getData().getName()).isEqualTo("test-{feature}");
              })
              .verifyComplete();
}
```

### Query Tests

```java
@Test
void testList{Features}() {
  AccountSrn account = new AccountSrn("srn:iam:test:account/test");
  
  List{Features} query = new List{Features}({feature}ListRepository)
      .account(account);
  
  StepVerifier.create(query.execute())
              .assertNext({feature} -> {
                assertThat({feature}.getName()).isNotNull();
              })
              .verifyComplete();
}
```

### Controller Tests

```java
@Test
void testCreate{Feature}Endpoint() {
  Create{Feature}Request request = new Create{Feature}Request(
      "test-{feature}",
      "Test description"
  );
  
  webTestClient.post()
               .uri("/iam/{features}")
               .contentType(MediaType.APPLICATION_JSON)
               .bodyValue(request)
               .exchange()
               .expectStatus().isCreated()
               .expectBody({Feature}MutationResponse.class)
               .value(response -> {
                 assertThat(response.get{Feature}().getName()).isEqualTo("test-{feature}");
               });
}
```

---

## Summary Checklist

When implementing a new CQRS feature, follow this checklist:

### Domain Layer (-internals)
- [ ] Create package structure: `features/{feature}/`
- [ ] Define SRN class (`{Feature}Srn`)
- [ ] Create base command (`Reactive{Feature}Command`)
- [ ] Implement Create/Update/Delete commands
- [ ] Define domain events (Created/Updated/Deleted)
- [ ] Create aggregate projection (`{Feature}Details`)
- [ ] Implement queries (Get, List, History)
- [ ] Create list repository and entity
- [ ] Add list management commands
- [ ] Define custom exceptions

### API Layer (-server or -application)
- [ ] Create request DTOs with validation
- [ ] Create response DTOs with metadata
- [ ] Implement converter class
- [ ] Create controller with dual endpoints
- [ ] Add OpenAPI annotations
- [ ] Configure authorization (`@IamRequireAction`)
- [ ] Add exception handlers to `ControllerAdvice`
- [ ] Add `@ModelAttribute` for SRN binding

### Infrastructure
- [ ] Create Cassandra migration scripts
- [ ] Configure event handlers (if needed)
- [ ] Add managed data (if applicable)
- [ ] Write unit tests
- [ ] Write integration tests
- [ ] Update API documentation

---

## Capturing Event Metadata in Projections

Projections can capture metadata from events (timestamps, user/principal) by wrapping event handlers with the `Event<?, EventType>` parameter instead of just the raw event type.

### Event Wrapper Methods

The `Event` wrapper (from `com.source2sea.evento.aggregate.Event`) provides:

- **`getData()`** - Returns the event payload/data
- **`getEventTime()`** - Returns `Instant` when the event occurred
- **`getPrincipal()`** - Returns the user/principal who caused the event

### Pattern: Capture Metadata with Different Method Names

Due to Java type erasure, you cannot overload methods with `Event<?, EventType1>` and `Event<?, EventType2>` - they both erase to `Event`. Use **different method names** for each event handler:

```java
@Projection(value = Application.class, name = "application-details")
@Data
public class ApplicationDetails {
  private ApplicationSrn srn;
  private String name;
  private String baseUrl;
  private Instant createdAt;
  private Instant updatedAt;
  private String updatedBy;
  private boolean present;

  @EventHandler
  public void onCreated(Event<?, ApplicationCreated> event) {
    ApplicationCreated data = event.getData();
    this.name = data.getName();
    this.baseUrl = data.getBaseUrl();
    this.present = true;
    this.createdAt = event.getEventTime();
    this.updatedAt = this.createdAt;
    this.updatedBy = event.getPrincipal();
  }

  @EventHandler
  public void onUpdated(Event<?, ApplicationUpdated> event) {
    ApplicationUpdated data = event.getData();
    if (data.getName() != null) {
      this.name = data.getName();
    }
    if (data.getBaseUrl() != null) {
      this.baseUrl = data.getBaseUrl();
    }
    this.updatedAt = event.getEventTime();
    this.updatedBy = event.getPrincipal();
  }

  @EventHandler
  public void onDeleted(Event<?, ApplicationDeleted> event) {
    this.present = false;
    this.updatedAt = event.getEventTime();
    this.updatedBy = event.getPrincipal();
  }
}
```

### Key Points

1. **Use unique method names** - `onCreated()`, `onUpdated()`, `onDeleted()` instead of overloading `on()`
2. **Wrap ALL events that need metadata** - Every event handler that updates timestamps/user must use `Event<?, EventType>`
3. **Access payload with `event.getData()`** - The event data is inside the wrapper
4. **Store metadata in projection fields** - Makes timestamps/user available to queries and API responses

### When to Use

- When you need `createdAt`, `updatedAt`, or `updatedBy` fields in API responses
- When audit information is required from the projection
- When timestamps need to be available without querying the event store

### Alternative: Raw Event Handlers

If you don't need metadata, use raw event types (simpler, no wrapper):

```java
@EventHandler
public void on(ApplicationUpdated event) {
  if (event.getName() != null) {
    this.name = event.getName();
  }
}
```

---

## Advanced Patterns (Optional)

### QueryFactory in Commands

Some projects (e.g., warehouse) inject `QueryFactory` into commands to enable cross-aggregate queries before committing events.

**Pattern:**

```java
@RequiredArgsConstructor
abstract public class Reactive{Feature}Command<E> implements ReactiveCommand<Event<{Feature}, E>> {
  
  @Autowired
  protected ReactiveAggregateRepository repository;
  
  @Autowired
  protected QueryFactory queryFactory;  // Cross-aggregate queries
  
  protected Flux<Event<{Feature}, E>> commitEvent(E event) {
    return repository.getInstance({Feature}.class, id.srn())
                     .flatMapMany(a -> a.event(event));
  }
}
```

**Usage:**

```java
public Flux<Event<Product, ProductCreated>> execute() {
  assertions();
  return queryFactory
      .create(CatalogDetails.class)  // Query another aggregate first
      .catalog(id().catalog())
      .execute()
      .thenMany(materialize()
          .flatMapMany(details -> commitEvent(new ProductCreated(...))));
}
```

Use this pattern when commands need to validate data from other aggregates before committing events.

---

### Change Detection Optimization

Some projects implement change detection to prevent redundant events when data has not changed.

**Pattern:**

```java
@Value("${cqrs.change.detection:true}")
protected boolean changeDetectionEnabled = true;

@Override
public Flux<Event<{Feature}, {Feature}Updated>> execute() {
  assertions();
  return materialize()
      .flatMapMany(current -> {
        if (current.isPresent() && !isChanged(current)) {
          return Mono.empty();  // No change, no event
        }
        return commitEvent(new {Feature}Updated(...));
      });
}

protected boolean isChanged({Feature}Details current) {
  if (!Objects.equals(current.getName(), name)) return true;
  if (!Objects.equals(current.getDescription(), description)) return true;
  return false;
}
```

Use this pattern to optimize event volume when updates may be called with unchanged data.

---

### ECST (Event Context State Transfer)

ECST must be implemented for every feature to enable projection state transfer. ECST synchronizes projection state across services and instances without querying aggregates.

#### When to Use ECST

- Always implement ECST when creating features
- Enables zero-downtime deployments with projection state transfer
- Allows new service instances to receive current projection state
- Supports cross-service projection synchronization

#### Implementation Structure

ECST requires three components in the `-internals` module:

**1. Enable Annotation**

```java
package com.{org}.{project}.features.{feature}.projection.annotations;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface Enable{Feature}ProjectionProcessing {
}
```

**2. AutoConfiguration with Publisher and Consumer**

```java
package com.{org}.{project}.features.{feature}.projection;

import com.source2sea.commons.cqrs.factories.CommandFactory;
import com.source2sea.commons.cqrs.factories.QueryFactory;
import com.source2sea.evento.aggregate.ReactiveAggregateRepository;
import com.source2sea.evento.messaging.mappers.MessageMapper;
import com.source2sea.evento.messaging.mappers.ecst.AggregateRepositoryEcstProjectionLoader;
import com.source2sea.evento.messaging.mappers.ecst.EcstMessageConsumer;
import com.source2sea.evento.messaging.mappers.ecst.EcstMessageMapperFactory;
import com.source2sea.evento.messaging.mappers.ecst.EcstPublisher;
import com.source2sea.evento.messaging.mappers.reactive.ReactiveEcstConsumerMessageMapperFactory;
import org.springframework.boot.autoconfigure.condition.ConditionalOnBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
@ConditionalOnBean(annotation = Enable{Feature}ProjectionProcessing.class)
public class {Feature}ProjectionAutoConfiguration {

    @Bean
    public {Feature}Service {feature}Service(CommandFactory commandFactory, 
                                             QueryFactory queryFactory) {
        return new {Feature}Service(commandFactory, queryFactory);
    }

    /**
     * Publisher bean for ECST state transfer.
     * Loads projection from aggregate repository and publishes to other instances.
     */
    @Bean
    public MessageMapper {feature}StateTransfer(ReactiveAggregateRepository aggregateRepository, 
                                               EcstPublisher publisher) {
        return EcstMessageMapperFactory.forProjection({Feature}Projection.class)
                                       .withLoader(new AggregateRepositoryEcstProjectionLoader<>(aggregateRepository))
                                       .withPublisher(publisher)
                                       .handleAll()
                                       .relaxed()
                                       .build();
    }

    /**
     * Consumer bean for ECST state transfer.
     * Receives projection state from other instances and updates local state.
     */
    @Bean
    public EcstMessageConsumer {feature}StateTransferConsumer({Feature}Service {feature}Service) {
        return ReactiveEcstConsumerMessageMapperFactory.forProjection({Feature}Projection.class)
                                                       .withoutConsumerGroup()
                                                       .withHandler(projection -> 
                                                           {feature}Service.updateProjection(
                                                               projection.getData(), 
                                                               projection.getEventId()))
                                                       .build();
    }
}
```

**3. Service with Projection Update Handler**

```java
package com.{org}.{project}.features.{feature}.projection;

import com.source2sea.commons.cqrs.factories.CommandFactory;
import com.source2sea.commons.cqrs.factories.QueryFactory;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import reactor.core.publisher.Mono;

import java.util.UUID;

@Slf4j
@RequiredArgsConstructor
public class {Feature}Service {
    
    private final CommandFactory commandFactory;
    private final QueryFactory queryFactory;
    
    /**
     * Updates the projection with state received via ECST.
     * <p>
     * This method is called by the ECST consumer when projection state
     * is transferred from another service instance.
     *
     * @param projection the projection data
     * @param eventId the event ID associated with this projection state
     * @return completion signal
     */
    public Mono<Void> updateProjection({Feature}Projection projection, UUID eventId) {
        return commandFactory.create(Save{Feature}Projection.class)
                             .projection(projection)
                             .eventId(eventId)
                             .execute()
                             .then();
    }
}
```

#### Enable ECST in Consumer/Server Module

In the `-consumer` or `-server` module main class, add the enable annotation:

```java
package com.{org}.{project};

import com.{org}.{project}.features.{feature}.projection.annotations.Enable{Feature}ProjectionProcessing;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@Enable{Feature}ProjectionProcessing  // Enable ECST for this feature
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

#### ECST Configuration

ECST requires Evento messaging configuration. The publisher and consumer beans are automatically wired by Evento when:

1. `EcstPublisher` bean is available (provided by Evento)
2. `ReactiveAggregateRepository` is configured
3. Enable annotation is present on application class

#### Real Example (Warehouse Categories)

```java
@Bean
public MessageMapper categoryTreeStateTransfer(ReactiveAggregateRepository aggregateRepository, 
                                              EcstPublisher publisher) {
    return EcstMessageMapperFactory.forProjection(Tree.class)
                                   .withLoader(new AggregateRepositoryEcstProjectionLoader<>(aggregateRepository))
                                   .withPublisher(publisher)
                                   .handleAll()
                                   .relaxed()
                                   .build();
}

@Bean
public EcstMessageConsumer categoryStateTransferConsumer(CategoryService categoryService) {
    return ReactiveEcstConsumerMessageMapperFactory.forProjection(Tree.class)
                                                   .withoutConsumerGroup()
                                                   .withHandler(tree -> 
                                                       categoryService.updateTree(
                                                           tree.getData(), 
                                                           tree.getEventId()))
                                                   .build();
}
```

#### Package Structure for ECST

```
{project}-internals/src/main/java/com/{org}/{project}/features/{feature}/
├── aggregates/
│   └── projections/
│       └── {Feature}Projection.java        ← Projection class for ECST
└── projection/
    ├── annotations/
    │   └── Enable{Feature}ProjectionProcessing.java
    ├── commands/
    │   └── Save{Feature}Projection.java    ← Save received projection
    ├── queries/
    │   └── Get{Feature}Projection.java     ← Query saved projection
    ├── {Feature}ProjectionAutoConfiguration.java
    └── {Feature}Service.java
```

---

## Javadoc Guidelines

### Purpose and Scope

Add Javadoc to classes, fields, and methods in the domain layer (-internals module). Javadoc must provide context about design decisions for future maintainers.

Follow the antora-docs skill tone: precise, neutral, helpful. Keep documentation short, usually one to three sentences per element.

### When to Document

**Always document:**
- Command classes - explain business purpose and constraints
- Event classes - explain what happened and when emitted
- Query classes - explain what data is retrieved and why
- Projection classes - explain materialized state shape
- Base command classes - explain shared behavior and helpers

**Document design decisions:**
- Why certain validations exist
- Why specific event sequences matter
- Why transactional vs async patterns were chosen
- Why certain queries are needed
- Why projections include specific fields

### Documentation Style

Use calm expert tone from antora-docs skill:
- Precise and neutral, not marketing language
- Short paragraphs, one to three sentences
- Use "must", "should", "can" with exact meanings
- Do not say "easy", "simple", "just", or "obviously"

**Modal meanings:**
- `must`: required for correctness, safety, or compatibility
- `should`: recommended default with valid exceptions
- `can`: optional capability

### Class Documentation Template

```java
/**
 * Command to create a new {Feature} aggregate.
 * <p>
 * Validates required fields and emits a {@link {Feature}Created} event.
 * The {feature} name must be unique within the account scope.
 * Inherits existence checks from {@link Reactive{Feature}Command}.
 */
@Accessors(fluent = true)
@Setter
public class Create{Feature} extends Reactive{Feature}Command<{Feature}Created> {
```

**Real Example:**

```java
/**
 * Command to update metadata of an Account aggregate.
 * <p>
 * Validates the provided account name and propagates tag changes by emitting
 * an {@link AccountChanged} event. Inherits common account existence and state
 * checks from {@link ReactiveAccountCommand}.
 */
@Accessors(fluent = true)
@Setter
public class UpdateAccount extends ReactiveAccountCommand<UpdateAccount,AccountChanged> {
```

### Field Documentation Template

```java
/**
 * The new human-readable name to assign to the {feature}.
 */
private String name;

/**
 * A map of tags (key-value pairs) to associate with the {feature}.
 * Tags provide metadata for organization and filtering.
 */
private Map<String, String> tags;
```

### Method Documentation Template

Document the purpose, parameters, return value, and any exceptions. Explain design decisions.

```java
/**
 * Executes the command by validating inputs and emitting a {Feature}Created event.
 * <p>
 * The {feature} SRN is generated before validation to ensure consistency.
 * All required fields are validated via {@link #assertions()}.
 *
 * @return a {@link Flux} emitting the committed {@link Event} for the {feature} creation
 */
@Override
public Flux<Event<{Feature}, {Feature}Created>> execute() {
  {feature}(new {Feature}Srn(account, name));
  assertions();
  return commitEvent(new {Feature}Created(description, ...));
}
```

**Document validation reasoning:**

```java
/**
 * Validates required preconditions before execution.
 * <ul>
 *   <li>{Feature} existence and state from the superclass.</li>
 *   <li>Non-null name.</li>
 *   <li>At least one statement must be provided for policy enforcement.</li>
 * </ul>
 * <p>
 * The statements list must not be empty because policies without statements
 * would grant no permissions.
 *
 * @throws IllegalArgumentException if name is null or statements is empty
 */
@Override
protected void assertions() {
  super.assertions();
  Assert.notNull(name, "name must not be null");
  Assert.notNull(statements, "statements must not be null");
  Assert.isTrue(!statements.isEmpty(), "At least one policy statement must be provided");
}
```

### Event Documentation

Explain what happened and when the event is emitted.

```java
/**
 * Event emitted when a {feature} is successfully created.
 * <p>
 * Contains the complete {feature} data to enable event handlers
 * to update list tables without querying the aggregate.
 * Emitted by {@link Create{Feature}} command.
 */
@Event("domain:{feature}:{Feature}Created")
public class {Feature}Created {
  /**
   * Description of the {feature}.
   */
  private String description;
  
  /**
   * Complete list of related objects. Full objects are included
   * rather than IDs to enable handlers to process this event independently.
   */
  private List<ComplexModel> relatedObjects;
}
```

### Base Command Documentation

Document shared helpers and their purpose.

```java
/**
 * Base class for all {feature} aggregate commands.
 * <p>
 * Provides common functionality:
 * <ul>
 *   <li>{@link #exists()} - Check if aggregate exists</li>
 *   <li>{@link #basic{Feature}Data()} - Get aggregate data or throw if not found</li>
 *   <li>{@link #materialize()} - Load aggregate projection</li>
 *   <li>{@link #commitEvent(Object)} - Persist events to event store</li>
 * </ul>
 * Update, Delete, and specialized commands use {@link #basic{Feature}Data()}
 * to verify existence before modifying the aggregate.
 *
 * @param <E> the event type this command emits
 */
@Accessors(fluent = true)
@Setter
abstract public class Reactive{Feature}Command<E> implements ReactiveCommand<Event<{Feature}, E>> {
  
  /**
   * Check if the {feature} aggregate exists.
   *
   * @return {@code true} if aggregate exists, {@code false} otherwise
   */
  protected Mono<Boolean> exists() {
    return materialize().map({Feature}Details::isPresent);
  }
  
  /**
   * Get {feature} aggregate data or throw exception if not found.
   * <p>
   * Use this method in Update, Delete, and specialized commands to verify
   * the aggregate exists before attempting modifications.
   *
   * @return the {feature} projection data
   * @throws {Feature}DoesNotExist if aggregate is not found
   */
  protected Mono<{Feature}Details> basic{Feature}Data() {
    return materialize()
        .filter({Feature}Details::isPresent)
        .switchIfEmpty(Mono.error(new {Feature}DoesNotExist("{feature} not found")));
  }
}
```

### Design Decision Documentation

When making design choices, document the reasoning:

```java
/**
 * Commits the event using synchronous transaction pattern.
 * <p>
 * The account list item is inserted in the same transaction as the event
 * because account hierarchy queries must show new accounts immediately.
 * This ensures strong consistency for parent-child account relationships.
 * <p>
 * Most features use async event handlers for list updates, but accounts
 * require immediate consistency.
 *
 * @param accountId the generated account identifier
 * @return a flux emitting the committed AccountCreated event
 */
private Flux<Event<Account, AccountCreated>> commitEvent(String accountId) {
  AccountSrn srn = new AccountSrn(accountId);
  return repository.getInstance(Account.class, srn)
                   .flatMapMany(a -> a.event(
                       tx -> tx.insert(new AccountListItem(...)),
                       new AccountCreated(parent, name, tags)));
}
```

---

## Kafka Topic Configuration

When creating a new feature, Kafka topics must be configured in FluxCD for event routing and ECST state transfer.

### Manifest Verification Checklist

Before considering a feature complete:

1. Compare application routing with `spec.topicName` (or `metadata.name` when `spec.topicName` is omitted) for every aggregate, ECST state, and ECST replay topic. All routed topics must have matching manifests.
2. Verify topic manifest files are included in the relevant Kustomize `resources`, directly or through a referenced base. A file in the FluxCD folder alone is not deployed.
3. Inspect the service KafkaUser manifests for topic Read/Write permissions and consumer-group permissions matching the actual group IDs. Reuse existing prefix ACLs when they cover the new topics and groups; do not add redundant ACLs or broaden permissions unnecessarily.
4. Match repository conventions for partitions, replication, retention, cleanup policy, labels, prune annotations, and supported Strimzi API versions. The values below are examples, not universal defaults.
5. Preserve existing topic version suffixes, such as `.v2`, wherever required by routing and manifests. Do not rename existing topics or infer a version suffix from a manifest filename.

### Topic Types and Naming Conventions

#### **1. Aggregate Event Topics**

Store domain events for event sourcing.

**Naming Pattern:** `com.source2sea.{service}.aggregates.{aggregate}`

**Configuration:**
- **Partitions:** 150
- **Replication:** 3
- **Retention:** Infinite (`retention.ms: -1`, `retention.bytes: -1`)

**Example FluxCD Manifest:**

```yaml
apiVersion: kafka.strimzi.io/v1beta1
kind: KafkaTopic
metadata:
  name: {service}-aggregates-{feature}  # Hyphenated repository convention
  labels:
    app.kubernetes.io/managed-by: fluxcd
    app.kubernetes.io/name: {service}
    strimzi.io/cluster: ${kafka_strimzi_cluster:=kafka-cluster}
  annotations:
    kustomize.toolkit.fluxcd.io/prune: "disabled"
spec:
  partitions: 150
  replicas: 3
  topicName: com.source2sea.{service}.aggregates.{feature}  # Actual Kafka topic name
  config:
    retention.ms: -1
    retention.bytes: -1
```

**Key Points:**
- `metadata.name`: Kubernetes resource identifier; hyphenated names follow the repository convention. Kubernetes DNS-subdomain resource names can contain dots.
- `topicName`: Actual Kafka topic name (use dots for namespacing)
- Labels should include app.kubernetes.io metadata for proper tracking

**Application Properties Routing:**

```properties
evento.messaging.routes.{feature}.sink=com.source2sea.{service}.aggregates.{feature}
```

---

#### **2. ECST State Topics**

Store projection state snapshots for cross-service synchronization.

**Naming Pattern:** `com.source2sea.{service}.{entity}`

**Configuration:**
- **Partitions:** 150
- **Replication:** 3
- **Cleanup Policy:** `compact` only (required for ECST)
- **Retention:** Do not set retention settings - compaction manages retention
- **Purpose:** State snapshots - keeps only latest state per key
- **Difference from Event Topics:** ECST topics must use compaction only, event topics use infinite retention

**Example FluxCD Manifest:**

```yaml
apiVersion: kafka.strimzi.io/v1beta1
kind: KafkaTopic
metadata:
  name: {service}-{entity}-projection
  labels:
    strimzi.io/cluster: ${kafka_strimzi_cluster:=kafka-cluster}
  annotations:
    kustomize.toolkit.fluxcd.io/prune: "disabled"
spec:
  partitions: 150
  replicas: 3
  topicName: com.source2sea.{service}.{entity}
  config:
    cleanup.policy: compact  # ECST topics must use compaction only (no retention settings)
```

**Application Properties Routing:**

```properties
evento.messaging.routes.{feature}.ecst.{projection-name}.sink=com.source2sea.{service}.{entity}
```

---

#### **3. ECST Replay Topics**

Store time-limited events for rebuilding consumers.

**Naming Pattern:** `com.source2sea.{service}.{entity}.replay`

**Configuration:**
- **Partitions:** 150 (match state topic)
- **Replication:** 3
- **Retention:** 7 days (`retention.ms: 604800000`)

**Example FluxCD Manifest:**

```yaml
apiVersion: kafka.strimzi.io/v1beta1
kind: KafkaTopic
metadata:
  name: com.source2sea.{service}.{entity}.replay
  labels:
    strimzi.io/cluster: ${kafka_strimzi_cluster:=kafka-cluster}
  annotations:
    kustomize.toolkit.fluxcd.io/prune: "disabled"
spec:
  partitions: 150
  replicas: 3
  config:
    retention.ms: "604800000"
    retention.bytes: "-1"
```

**Application Properties Routing:**

```properties
evento.messaging.routes.{feature}.ecst.{projection-name}.replaySink=com.source2sea.{service}.{entity}.replay
```

---

### Complete Application Properties Example

For a new "tags" feature in IAM:

```properties
# Aggregate event routing
evento.messaging.routes.tag.sink=com.source2sea.iam.aggregates.tag.v2

# ECST projection routing
evento.messaging.routes.tag.ecst.tag-details.sink=com.source2sea.iam.tag
evento.messaging.routes.tag.ecst.tag-details.replaySink=com.source2sea.iam.tag.replay
```

---

### FluxCD File Structure

Organize topic definitions in shared base directory:

```
{project}/fluxcd/base/shared/
├── kafka-topics-v2.yaml           # Aggregate event topics
├── kafka-topics-ecst.yaml         # ECST state + replay topics
└── kafka-topics-{feature}.yaml    # Feature-specific topics if complex
```

---

### Kafka User Permissions

Service KafkaUser must have permissions for all topics with service prefix.

**Example KafkaUser (Strimzi):**

```yaml
apiVersion: kafka.strimzi.io/v1beta2
kind: KafkaUser
metadata:
  name: {service}-server-kafka-user
  labels:
    strimzi.io/cluster: ${kafka_strimzi_cluster:=kafka-cluster}
spec:
  authentication:
    type: scram-sha-512
  authorization:
    type: simple
    acls:
      - operations: [Read, Write]
        resource:
          name: com.source2sea.{service}
          patternType: prefix
          type: topic
      - operations: [Read, Delete, Describe]
        resource:
          name: {service}
          patternType: prefix
          type: group
```

---

### Environment Variables (Deployment)

Configure Kafka connection in deployment manifests:

```yaml
- name: EVENTO_MESSAGING_KAFKA_BROKERS
  value: ${kafka_bootstrap_servers:=localhost:9092}
- name: EVENTO_MESSAGING_KAFKA_CONFIGURATION_SASL_MECHANISM
  value: SCRAM-SHA-512
- name: EVENTO_MESSAGING_KAFKA_CONFIGURATION_SECURITY_PROTOCOL
  value: SASL_PLAINTEXT
- name: EVENTO_MESSAGING_KAFKA_CONFIGURATION_SASL_JAAS_CONFIG
  valueFrom:
    secretKeyRef:
      key: sasl.jaas.config
      name: {service}-server-kafka-user
```

---

### Checklist for New Feature Topics

When adding a new feature:

1. Create aggregate event topic with v2 suffix, 150 partitions, infinite retention (no compaction)
2. Create ECST state topic with 150 partitions and `cleanup.policy: compact` (required)
3. Create ECST replay topic with 150 partitions, 7-day retention (no compaction)
4. Add routing configuration to application.properties
5. Verify KafkaUser has prefix permissions for service topics
6. Add topics to FluxCD base/shared directory
7. Protect topics with prune disabled annotation
8. Use variable substitution for cluster name

---

## Creating SRNs for New Features

When implementing a new feature, you must create an SRN (Source2Sea Resource Name) type in the commons-srn library.

**Required Steps:**

1. Create SRN class in `commons-srn/src/main/java/com/source2sea/commons/srn/{service}/`
2. Create SRN parser (if custom validation needed)
3. Create SRN converter in `application/converters/`
4. Register converter in `Srn.java` static map
5. Write unit tests for SRN class and converter

**Example SRN format:** `srn:iam:123e4567-e89b-12d3-a456-426614174000:tag/production`

**For detailed implementation guidance, use the `source2sea-srn` skill:**

This skill covers:
- SRN class structure and patterns
- Parser implementation with validation rules
- Converter implementation and registration
- Jackson deserializer setup
- Unit test patterns for SRN and converter
- Usage patterns in commands and controllers
- Naming conventions and validation rules

---

## Key Principles

1. **Separation of Concerns**: Commands write, queries read
2. **Immutable Events**: All state changes captured as events
3. **Dual Responses**: Offer both event streams and materialized models
4. **Factory Pattern**: Use factories for dependency injection
5. **Fluent APIs**: Commands and queries use fluent setters
6. **Reactive All The Way**: Non-blocking I/O throughout
7. **Validation Layers**: Request validation (Jakarta) + command validation (Spring Assert)
8. **OpenAPI First**: Comprehensive API documentation
9. **Authorization**: Fine-grained action-based permissions
10. **Testability**: Separate business logic for easy testing
11. **Document Decisions**: Javadoc must explain design choices for future maintainers
