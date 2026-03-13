# Content Summary: Sections 02–05

## Section 02 — Basics

### 02 - Creating a Pydantic Model
Covers inheriting from `BaseModel` to define models with type-hinted fields. Demonstrates field access via dot notation, the `model_fields` property, and how `ValidationError` reports all issues at once rather than failing on the first.

### 03 - Deserialization
Three deserialization methods: constructor kwargs, `model_validate()` for dicts, and `model_validate_json()` for JSON strings. Recommends `model_validate`/`model_validate_json` over `**` unpacking for complex models.

### 04 - Serialization
Converting models to dicts (`model_dump()`) and JSON (`model_dump_json()`). Covers `exclude` and `include` parameters for controlling which fields appear in output. By default serialization uses field names, not aliases.

### 05 - Type Coercion
Pydantic's default lax coercion (e.g., int → float). Highlights differences between Python-object and JSON coercion rules. Not all types coerce automatically — consult Pydantic's conversion table.

### 06 - Required vs Optional Fields
Optional fields require defaults. Pydantic deep-copies mutable defaults per instance (safer than plain Python). Defaults are not validated unless explicitly configured.

### 07 - Nullable Fields
Three equivalent syntaxes: `int | None`, `Union[int, None]`, `Optional[int]`. Nullable (accepts `None`) and optional (has a default) are independent concepts.

### 08 - Combining Nullable and Optional
Walks through all four combinations: required/non-nullable, required/nullable, optional/non-nullable, optional/nullable. The most common pattern is `int | None = None`.

### 09 - Inspecting Fields
`model_fields` for schema-level info; `model_fields_set` for per-instance tracking of which fields the caller actually provided. Useful for REST APIs that should echo back only user-supplied data via `model_dump(include=...)`.

### 10 - JSON Schema Generation
`model_json_schema()` produces a JSON Schema dict, useful for API documentation. FastAPI generates this automatically.

### 11–12 - Project & Solution
Practical exercise building an `Automobile` model that applies all Section 02 concepts (various field types, nullable strings, dates, booleans).

---

## Section 03 — Model Configuration

### 02 - Handling Extra Fields
Three modes via `ConfigDict(extra=...)`: `"ignore"` (default), `"forbid"` (raise error), `"allow"` (store in `model_extra`). Use `"forbid"` for strict APIs; `"allow"` for flexible ingestion.

### 03 - Strict and Lax Type Coercion
`ConfigDict(strict=True)` tightens coercion rules. Strict mode still permits some conversions. Python-object and JSON deserialization follow different strictness tables.

### 04 - Validating Default Values
Defaults are trusted by default. `ConfigDict(validate_default=True)` enables validation — useful when validators apply transformations that defaults should also undergo.

### 05 - Validating Assignments
Post-instantiation attribute changes are not validated by default. `ConfigDict(validate_assignment=True)` enables this, catching invalid mutations.

### 06 - Mutability
Models are mutable by default. `ConfigDict(frozen=True)` makes them immutable and hashable, allowing use as dict keys or in sets.

### 07 - Coercing Numbers to Strings
`ConfigDict(coerce_numbers_to_str=True)` enables automatic number → string conversion, handy when external APIs send numeric values for string fields.

### 08 - Standardizing Strings
`str_strip_whitespace`, `str_to_lower`, `str_to_upper` config options normalize string data without custom validators.

### 09 - Handling Python Enums
Using `Enum` types for constrained fields. `ConfigDict(use_enum_values=True)` stores the raw value instead of the Enum member. Covers `AliasChoices` for accepting multiple field names.

### 10–11 - Project & Solution
Extends the `Automobile` model with an `AutomobileType` enum, extra-field forbidding, whitespace stripping, and default/assignment validation.

---

## Section 04 — Field Aliasing, Serialization and Deserialization

### 02 - Field Aliases and Default Values
`Field(alias="...")` maps external names to internal field names during deserialization. Serialization uses field names unless `by_alias=True` is passed.

### 03 - Alias Generator Functions
`ConfigDict(alias_generator=to_camel)` auto-generates aliases. Pydantic ships `to_camel`, `to_snake`, `to_pascal`. Individual fields can override the generated alias.

### 04 - Deserializing by Field Name or Alias
`ConfigDict(populate_by_name=True)` lets callers supply either the field name or alias during deserialization.

### 05 - Serialization Aliases
`Field(serialization_alias="...")` sets a separate alias used only during serialization, useful when input and output schemas differ.

### 06 - Validation Aliases
`Field(validation_alias="...")` overrides only the deserialization alias. `AliasChoices` accepts multiple candidate names, helpful for data sources with inconsistent naming.

### 07 - Custom Serializers
`@field_serializer` decorator controls per-field output. The `when_used` parameter (`"always"`, `"json"`, `"json-unless-none"`, `"unless-none"`) and `FieldSerializationInfo.mode` allow mode-dependent formatting (e.g., different datetime formats for dict vs JSON).

### 08–09 - Project & Solution
Applies camelCase alias generation, special-case aliases, `AliasChoices` for alternative names, and a custom date serializer to the `Automobile` model.

---

## Section 05 — Specialized Pydantic Types

### 01 - Pydantic Types Documentation Links
Reference links to Pydantic's types and network-types documentation.

### 02 - PositiveInt
`PositiveInt` constrains a field to integers > 0. Related types include `NegativeInt`, `NonNegativeInt`, and float equivalents. Internally implemented via `Annotated` with `Gt`/`Lt` metadata.

### 03 - Constrained Lists
`conlist(int, min_length=2, max_length=3)` validates list element type and length. Tuples are coerced to lists. `unique_items` was removed in V2 — use `Set` for uniqueness.

### 04 - UUID
`UUID4` validates UUID v4 values. Use `Field(default_factory=uuid4)` (not `default=uuid4()`) to generate a unique ID per instance.

### 05 - Date Related Types
`PastDate`, `PastDatetime`, `NaiveDatetime`, `AwareDatetime` enforce temporal and timezone constraints. Naive datetimes are treated as local time; custom validators are recommended for UTC normalization.

### 06 - Network Types
`EmailStr` / `NameEmail` (requires `email-validator`), `AnyUrl` / `HttpUrl` for URL parsing, and `IPvAnyAddress` for IP validation. These types both validate and parse, exposing components like scheme, host, and path.

### 07–08 - Project & Solution
Adds a `UUID4` field (with alias `"id"` and default `None`) to the `Automobile` model, demonstrating specialized-type integration.
