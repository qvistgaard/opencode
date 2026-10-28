---
name: source2sea-srn
description: Use when creating, implementing, or working with Source2Sea Resource Names (SRNs) in the commons-srn library, including defining new SRN types, converters, parsers, and unit tests.
---

# Source2Sea Resource Names (SRN)

Use this skill when creating new SRN types or working with the SRN infrastructure in the commons-srn library located at `java-commons/commons-srn`.

## SRN Overview

SRNs (Source2Sea Resource Names) are unique identifiers for resources across Source2Sea services. They follow the format:

```
srn:{service}:{account-or-org}:{resource-type}/{resource-id}
```

**Examples:**
- `srn:iam:123e4567-e89b-12d3-a456-426614174000:root` (Account)
- `srn:iam:123e4567-e89b-12d3-a456-426614174000:policy/AllowReadCatalog` (Policy)
- `srn:catalog:source2sea:product/prod-12345` (Product)
- `srn:iam:123e4567-e89b-12d3-a456-426614174000:user/user-67890` (User)

---

## Creating a New SRN Type

When adding a new feature, you must create an SRN type in the commons-srn library.

### Implementation Checklist

1. Create SRN class in appropriate service package
2. Create SRN parser (if custom validation needed)
3. Create SRN converter for registration
4. Create Jackson deserializer (if needed)
5. Register converter in `Srn.java`
6. Write unit tests for SRN class
7. Write unit tests for converter

---

## Step 1: Create SRN Class

SRN classes extend either `IamSrn` (for IAM resources) or appropriate base class for other services.

**Location:** `commons-srn/src/main/java/com/source2sea/commons/srn/{service}/{Feature}Srn.java`

**Pattern for IAM Resources:**

```java
package com.source2sea.commons.srn.iam;

import com.source2sea.commons.srn.SrnParseException;
import com.source2sea.commons.srn.jackson.{Feature}SrnDeserializer;
import com.source2sea.commons.srn.jackson.SrnSerializer;
import tools.jackson.databind.annotation.JsonDeserialize;
import tools.jackson.databind.annotation.JsonSerialize;

/**
 * Represents a {Feature} SRN in the IAM context.
 * <p>
 * Format: srn:iam:{account-id}:{resource-type}/{resource-name}
 * Example: srn:iam:123e4567-e89b-12d3-a456-426614174000:tag/production
 */
@JsonSerialize(using = SrnSerializer.class)
@JsonDeserialize(using = {Feature}SrnDeserializer.class)
public class {Feature}Srn extends IamSrn {

  private final static {Feature}SrnParser parser = new {Feature}SrnParser();

  /**
   * Constructs a new {Feature}Srn with the specified account and name.
   *
   * @param account The account context for this {feature}
   * @param name The {feature} name
   * @throws IllegalArgumentException if the name is invalid
   */
  public {Feature}Srn(AccountSrn account, String name) {
    super(account, "{resource-type}", assertValid{Feature}Name(name));
  }

  /**
   * Validates the {feature} name.
   *
   * @param name The name to validate
   * @return The validated name
   * @throws IllegalArgumentException if the name is invalid
   */
  private static String assertValid{Feature}Name(String name) {
    parser.assertValid{Feature}Name(name);
    return name;
  }

  /**
   * Constructs a {Feature}Srn by parsing an SRN string.
   *
   * @param srn The SRN string to parse
   * @throws SrnParseException if the SRN is invalid
   */
  public {Feature}Srn(String srn) throws SrnParseException {
    super(srn, parser);
  }

  /**
   * Returns the name of the {feature}.
   *
   * @return The {feature} name
   */
  public String name() {
    return resourceId();
  }
}
```

**Real Example (PolicySrn):**

```java
package com.source2sea.commons.srn.iam;

import com.source2sea.commons.srn.SrnParseException;
import com.source2sea.commons.srn.jackson.PolicySrnDeserializer;
import com.source2sea.commons.srn.jackson.SrnSerializer;
import tools.jackson.databind.annotation.JsonDeserialize;
import tools.jackson.databind.annotation.JsonSerialize;

@JsonSerialize(using = SrnSerializer.class)
@JsonDeserialize(using = PolicySrnDeserializer.class)
public class PolicySrn extends IamSrn {

  private final static PolicySrnParser parser = new PolicySrnParser();

  public PolicySrn(AccountSrn account, String name) {
    super(account, "policy", assertValidPolicyName(name));
  }

  private static String assertValidPolicyName(String name) {
    parser.assertValidPolicyName(name);
    return name;
  }

  public PolicySrn(String srn) throws SrnParseException {
    super(srn, parser);
  }

  public String name() {
    return resourceId();
  }
}
```

---

## Step 2: Create SRN Parser (Optional)

Create a parser if you need custom validation rules for the resource name.

**Location:** `commons-srn/src/main/java/com/source2sea/commons/srn/{service}/{Feature}SrnParser.java`

**Pattern:**

```java
package com.source2sea.commons.srn.iam;

import com.source2sea.commons.srn.ServiceResourceSrnParser;

import java.util.regex.Pattern;

/**
 * Parser for {Feature} SRNs with name validation.
 */
public class {Feature}SrnParser extends ServiceResourceSrnParser {

  private static final Pattern NAME_PATTERN = Pattern.compile("^[a-zA-Z0-9_-]+$");

  public {Feature}SrnParser() {
    super("iam", "{resource-type}");
  }

  /**
   * Validates that the {feature} name conforms to naming rules.
   * <p>
   * Names must contain only alphanumeric characters, hyphens, and underscores.
   *
   * @param name The name to validate
   * @throws IllegalArgumentException if the name is invalid
   */
  public void assertValid{Feature}Name(String name) {
    if (name == null || name.isEmpty()) {
      throw new IllegalArgumentException("{Feature} name must not be null or empty");
    }
    if (!NAME_PATTERN.matcher(name).matches()) {
      throw new IllegalArgumentException(
          "{Feature} name must contain only alphanumeric characters, hyphens, and underscores");
    }
  }
}
```

---

## Step 3: Create SRN Converter

Converters register SRN types with the SRN infrastructure and enable parsing from strings.

**Location:** `commons-srn/src/main/java/com/source2sea/commons/srn/application/converters/{Feature}SrnConverter.java`

**Pattern:**

```java
package com.source2sea.commons.srn.application.converters;

import com.source2sea.commons.srn.SrnConverter;
import com.source2sea.commons.srn.iam.{Feature}Srn;
import lombok.SneakyThrows;
import org.springframework.boot.context.properties.ConfigurationPropertiesBinding;
import org.springframework.stereotype.Component;

import java.util.regex.Pattern;

/**
 * Converter for {Feature}Srn types.
 * <p>
 * Enables Spring configuration binding and SRN parsing from strings.
 */
@ConfigurationPropertiesBinding
@Component
public class {Feature}SrnConverter implements SrnConverter<{Feature}Srn> {

  private final static Pattern pattern = Pattern.compile("^srn:iam:.*?:{resource-type}/.*$");

  @Override
  public boolean supports(String srn) {
    return pattern.matcher(srn).matches();
  }

  @Override
  @SneakyThrows
  public {Feature}Srn convert(String source) {
    return new {Feature}Srn(source);
  }
}
```

**Real Example (AccountSrnConverter):**

```java
package com.source2sea.commons.srn.application.converters;

import com.source2sea.commons.srn.SrnConverter;
import com.source2sea.commons.srn.iam.AccountSrn;
import lombok.SneakyThrows;
import org.springframework.boot.context.properties.ConfigurationPropertiesBinding;
import org.springframework.stereotype.Component;

import java.util.regex.Pattern;

@ConfigurationPropertiesBinding
@Component
public class AccountSrnConverter implements SrnConverter<AccountSrn> {

  private final static Pattern pattern = Pattern.compile("^srn:iam:.*?:root$");

  @Override
  public boolean supports(String srn) {
    return pattern.matcher(srn).matches();
  }

  @Override
  @SneakyThrows
  public AccountSrn convert(String source) {
    return new AccountSrn(source);
  }
}
```

---

## Step 4: Create Jackson Deserializer (If Needed)

If your SRN class has a custom deserializer annotation, create it.

**Location:** `commons-srn/src/main/java/com/source2sea/commons/srn/jackson/{Feature}SrnDeserializer.java`

**Pattern:**

```java
package com.source2sea.commons.srn.jackson;

import com.source2sea.commons.srn.iam.{Feature}Srn;
import tools.jackson.core.JsonParser;
import tools.jackson.databind.DeserializationContext;
import tools.jackson.databind.JsonDeserializer;

import java.io.IOException;

/**
 * Jackson deserializer for {Feature}Srn.
 */
public class {Feature}SrnDeserializer extends JsonDeserializer<{Feature}Srn> {

  @Override
  public {Feature}Srn deserialize(JsonParser p, DeserializationContext ctxt) throws IOException {
    return new {Feature}Srn(p.getText());
  }
}
```

---

## Step 5: Register Converter in Srn.java

Add your converter to the static converters map in `Srn.java`.

**Location:** `commons-srn/src/main/java/com/source2sea/commons/srn/Srn.java`

**Add import:**

```java
import com.source2sea.commons.srn.iam.{Feature}Srn;
import com.source2sea.commons.srn.application.converters.{Feature}SrnConverter;
```

**Add to converters map:**

```java
static LinkedHashMap<Class<? extends Srn>, SrnConverter<? extends Srn>> converters = new LinkedHashMap<>() {{
  // ... existing converters ...
  put({Feature}Srn.class, new {Feature}SrnConverter());
  // ... more converters ...
}};
```

**Example from Srn.java:**

```java
static LinkedHashMap<Class<? extends Srn>, SrnConverter<? extends Srn>> converters = new LinkedHashMap<>() {{
  put(AccountSrn.class, new AccountSrnConverter());
  put(PolicySrn.class, new PolicySrnTypeConverter());
  put(RoleSrn.class, new RoleSrnTypeConverter());
  put(ClientSrn.class, new ClientSrnConverter());
  put(UserSrn.class, new UserSrnConverter());
  // ... more converters
}};
```

---

## Step 6: Write SRN Class Unit Tests

Test SRN construction, parsing, and validation.

**Location:** `commons-srn/src/test/java/com/source2sea/commons/srn/{service}/{Feature}SrnTest.java`

**Pattern:**

```java
package com.source2sea.commons.srn.iam;

import com.source2sea.commons.srn.SrnParseException;
import org.junit.jupiter.api.Test;

import java.util.UUID;

import static org.junit.jupiter.api.Assertions.*;

class {Feature}SrnTest {

  @Test
  void testConstructWith AccountAndName() {
    AccountSrn account = new AccountSrn(UUID.randomUUID().toString());
    {Feature}Srn {feature}Srn = new {Feature}Srn(account, "test-{feature}");
    
    assertTrue({feature}Srn.srn().startsWith("srn:iam:"));
    assertTrue({feature}Srn.srn().contains(":{resource-type}/test-{feature}"));
    assertEquals("test-{feature}", {feature}Srn.name());
  }

  @Test
  void testParseFromString() {
    String srnString = "srn:iam:" + UUID.randomUUID() + ":{resource-type}/my-{feature}";
    {Feature}Srn {feature}Srn = new {Feature}Srn(srnString);
    
    assertEquals(srnString, {feature}Srn.srn());
    assertEquals("my-{feature}", {feature}Srn.name());
  }

  @Test
  void testInvalidNameThrowsException() {
    AccountSrn account = new AccountSrn(UUID.randomUUID().toString());
    
    assertThrows(IllegalArgumentException.class, () -> {
      new {Feature}Srn(account, "");
    });
    
    assertThrows(IllegalArgumentException.class, () -> {
      new {Feature}Srn(account, "invalid name with spaces");
    });
  }

  @Test
  void testInvalidSrnStringThrowsException() {
    assertThrows(SrnParseException.class, () -> {
      new {Feature}Srn("invalid-srn-string");
    });
  }
}
```

---

## Step 7: Write Converter Unit Tests

Test converter pattern matching and conversion.

**Location:** `commons-srn/src/test/java/com/source2sea/commons/srn/application/converters/{Feature}SrnConverterTest.java`

**Pattern:**

```java
package com.source2sea.commons.srn.application.converters;

import com.source2sea.commons.srn.iam.{Feature}Srn;
import org.junit.jupiter.api.Test;

import java.util.UUID;

import static org.junit.jupiter.api.Assertions.*;

class {Feature}SrnConverterTest {

  @Test
  void testSupportsValid{Feature}Srn() {
    {Feature}SrnConverter converter = new {Feature}SrnConverter();
    UUID uuid = UUID.randomUUID();
    
    assertTrue(converter.supports("srn:iam:" + uuid + ":{resource-type}/test-{feature}"));
  }

  @Test
  void testDoesNotSupportInvalidSrn() {
    {Feature}SrnConverter converter = new {Feature}SrnConverter();
    
    assertFalse(converter.supports("srn:iam:123:invalid-type/name"));
    assertFalse(converter.supports("invalid-srn"));
  }

  @Test
  void testConvert{Feature}Srn() {
    {Feature}SrnConverter converter = new {Feature}SrnConverter();
    UUID uuid = UUID.randomUUID();
    String srnString = "srn:iam:" + uuid + ":{resource-type}/my-{feature}";
    
    {Feature}Srn result = converter.convert(srnString);
    
    assertEquals(srnString, result.srn());
    assertEquals("my-{feature}", result.name());
  }
}
```

**Real Example (AccountSrnConverterTest):**

```java
package com.source2sea.commons.srn.application.converters;

import com.source2sea.commons.srn.iam.AccountSrn;
import org.junit.jupiter.api.Test;

import java.util.UUID;

import static org.junit.jupiter.api.Assertions.assertEquals;

class AccountSrnConverterTest {

  @Test
  void testThatWeCanConvertAccountSrn() {
    UUID uuid = UUID.randomUUID();

    AccountSrnConverter accountSrnConverter = new AccountSrnConverter();
    AccountSrn convert = accountSrnConverter.convert("srn:iam:" + uuid + ":root");
    assertEquals("srn:iam:" + uuid + ":root", convert.srn());
  }
}
```

---

## SRN Usage in Services

Once the SRN is defined in commons-srn, use it in your service:

### In Commands

```java
@Accessors(fluent = true)
@Setter
public class Create{Feature} extends Reactive{Feature}Command<{Feature}Created> {
  private String name;
  private AccountSrn account;
  
  @Override
  public Flux<Event<{Feature}, {Feature}Created>> execute() {
    {feature}(new {Feature}Srn(account, name));  // Create SRN
    assertions();
    return commitEvent(new {Feature}Created(...));
  }
}
```

### In Controllers

```java
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

### In List Entities

```java
@Table("{feature}_list")
public class {Feature}ListItem {
  @PrimaryKeyColumn(type = PrimaryKeyType.PARTITIONED)
  private String account;
  
  @PrimaryKeyColumn(type = PrimaryKeyType.CLUSTERED)
  private String name;
  
  @Column("{feature}_srn")
  private String {feature}Srn;  // Store full SRN as string
}
```

---

## Naming Conventions

### Resource Type Names

Use lowercase, hyphenated names for multi-word resource types:
- `policy` (single word)
- `identity-provider` (multi-word)
- `category-tree` (multi-word)

### Class Names

Use PascalCase with "Srn" suffix:
- `PolicySrn`
- `IdentityProviderSrn`
- `CategoryTreeSrn`

### Method Names

Accessor methods should match the resource property name:
- `name()` - for simple name/id
- `resourceId()` - for generic resource identifier
- `account()` - for parent account
- Custom accessors as needed for specific properties

---

## Common Patterns

### Parent-Child Hierarchy

For resources with parent relationships:

```java
public class {Feature}Srn extends IamSrn {
  public {Feature}Srn(AccountSrn account, String name) {
    super(account, "{resource-type}", name);
  }
  
  @Override
  public Set<Srn> hierarchy() {
    LinkedHashSet<Srn> srns = new LinkedHashSet<>();
    srns.add(this);
    srns.add(account());  // Include parent in hierarchy
    return srns;
  }
}
```

### Validation Rules

Common validation patterns:
- Alphanumeric with hyphens and underscores: `^[a-zA-Z0-9_-]+$`
- No spaces allowed
- Minimum length requirements (typically 3+ characters)
- Maximum length limits (typically 128 characters)
- Case sensitivity considerations

---

## Key Principles

1. **Always register converters** in `Srn.java` static map
2. **Write unit tests** for both SRN class and converter
3. **Validate names** in parser before construction
4. **Document format** in class Javadoc
5. **Use consistent patterns** matching existing SRN types
6. **Store full SRN strings** in list entities (not just IDs)
7. **Test parsing and construction** in both directions
