# Content Summary: Chapters 02–12

## Chapter 02 — Basics

### 02.02 - Creating a Pydantic Model
Covers inheriting from `BaseModel` to define models with type-hinted fields. Demonstrates field access via dot notation, the `model_fields` property, and how `ValidationError` reports all issues at once rather than failing on the first.

### 02.03 - Deserialization
Three deserialization methods: constructor kwargs, `model_validate()` for dicts, and `model_validate_json()` for JSON strings. Recommends `model_validate`/`model_validate_json` over `**` unpacking for complex models.

### 02.04 - Serialization
Converting models to dicts (`model_dump()`) and JSON (`model_dump_json()`). Covers `exclude` and `include` parameters for controlling which fields appear in output. By default serialization uses field names, not aliases.

### 02.05 - Type Coercion
Pydantic's default lax coercion (e.g., int → float). Highlights differences between Python-object and JSON coercion rules. Not all types coerce automatically — consult Pydantic's conversion table.

### 02.06 - Required vs Optional Fields
Optional fields require defaults. Pydantic deep-copies mutable defaults per instance (safer than plain Python). Defaults are not validated unless explicitly configured.

### 02.07 - Nullable Fields
Three equivalent syntaxes: `int | None`, `Union[int, None]`, `Optional[int]`. Nullable (accepts `None`) and optional (has a default) are independent concepts.

### 02.08 - Combining Nullable and Optional
Walks through all four combinations: required/non-nullable, required/nullable, optional/non-nullable, optional/nullable. The most common pattern is `int | None = None`.

### 02.09 - Inspecting Fields
`model_fields` for schema-level info; `model_fields_set` for per-instance tracking of which fields the caller actually provided. Useful for REST APIs that should echo back only user-supplied data via `model_dump(include=...)`.

### 02.10 - JSON Schema Generation
`model_json_schema()` produces a JSON Schema dict, useful for API documentation. FastAPI generates this automatically.

### 02.11–02.12 - Project & Solution
Practical exercise building an `Automobile` model that applies all Chapter 02 concepts (various field types, nullable strings, dates, booleans).

---

## Chapter 03 — Model Configuration

### 03.02 - Handling Extra Fields
Three modes via `ConfigDict(extra=...)`: `"ignore"` (default), `"forbid"` (raise error), `"allow"` (store in `model_extra`). Use `"forbid"` for strict APIs; `"allow"` for flexible ingestion.

### 03.03 - Strict and Lax Type Coercion
`ConfigDict(strict=True)` tightens coercion rules. Strict mode still permits some conversions. Python-object and JSON deserialization follow different strictness tables.

### 03.04 - Validating Default Values
Defaults are trusted by default. `ConfigDict(validate_default=True)` enables validation — useful when validators apply transformations that defaults should also undergo.

### 03.05 - Validating Assignments
Post-instantiation attribute changes are not validated by default. `ConfigDict(validate_assignment=True)` enables this, catching invalid mutations.

### 03.06 - Mutability
Models are mutable by default. `ConfigDict(frozen=True)` makes them immutable and hashable, allowing use as dict keys or in sets.

### 03.07 - Coercing Numbers to Strings
`ConfigDict(coerce_numbers_to_str=True)` enables automatic number → string conversion, handy when external APIs send numeric values for string fields.

### 03.08 - Standardizing Strings
`str_strip_whitespace`, `str_to_lower`, `str_to_upper` config options normalize string data without custom validators.

### 03.09 - Handling Python Enums
Using `Enum` types for constrained fields. `ConfigDict(use_enum_values=True)` stores the raw value instead of the Enum member. Covers `AliasChoices` for accepting multiple field names.

### 03.10–03.11 - Project & Solution
Extends the `Automobile` model with an `AutomobileType` enum, extra-field forbidding, whitespace stripping, and default/assignment validation.

---

## Chapter 04 — Field Aliasing, Serialization and Deserialization

### 04.02 - Field Aliases and Default Values
`Field(alias="...")` maps external names to internal field names during deserialization. Serialization uses field names unless `by_alias=True` is passed.

### 04.03 - Alias Generator Functions
`ConfigDict(alias_generator=to_camel)` auto-generates aliases. Pydantic ships `to_camel`, `to_snake`, `to_pascal`. Individual fields can override the generated alias.

### 04.04 - Deserializing by Field Name or Alias
`ConfigDict(populate_by_name=True)` lets callers supply either the field name or alias during deserialization.

### 04.05 - Serialization Aliases
`Field(serialization_alias="...")` sets a separate alias used only during serialization, useful when input and output schemas differ.

### 04.06 - Validation Aliases
`Field(validation_alias="...")` overrides only the deserialization alias. `AliasChoices` accepts multiple candidate names, helpful for data sources with inconsistent naming.

### 04.07 - Custom Serializers
`@field_serializer` decorator controls per-field output. The `when_used` parameter (`"always"`, `"json"`, `"json-unless-none"`, `"unless-none"`) and `FieldSerializationInfo.mode` allow mode-dependent formatting (e.g., different datetime formats for dict vs JSON).

### 04.08–04.09 - Project & Solution
Applies camelCase alias generation, special-case aliases, `AliasChoices` for alternative names, and a custom date serializer to the `Automobile` model.

---

## Chapter 05 — Specialized Pydantic Types

### 05.01 - Pydantic Types Documentation Links
Reference links to Pydantic's types and network-types documentation.

### 05.02 - PositiveInt
`PositiveInt` constrains a field to integers > 0. Related types include `NegativeInt`, `NonNegativeInt`, and float equivalents. Internally implemented via `Annotated` with `Gt`/`Lt` metadata.

### 05.03 - Constrained Lists
`conlist(int, min_length=2, max_length=3)` validates list element type and length. Tuples are coerced to lists. `unique_items` was removed in V2 — use `Set` for uniqueness.

### 05.04 - UUID
`UUID4` validates UUID v4 values. Use `Field(default_factory=uuid4)` (not `default=uuid4()`) to generate a unique ID per instance.

### 05.05 - Date Related Types
`PastDate`, `PastDatetime`, `NaiveDatetime`, `AwareDatetime` enforce temporal and timezone constraints. Naive datetimes are treated as local time; custom validators are recommended for UTC normalization.

### 05.06 - Network Types
`EmailStr` / `NameEmail` (requires `email-validator`), `AnyUrl` / `HttpUrl` for URL parsing, and `IPvAnyAddress` for IP validation. These types both validate and parse, exposing components like scheme, host, and path.

### 05.07–05.08 - Project & Solution
Adds a `UUID4` field (with alias `"id"` and default `None`) to the `Automobile` model, demonstrating specialized-type integration.

---

## Chapter 06 — Additional Field Features

### 06.02 - Numerical Constraints
`Field()` parameters for numeric validation: `gt`, `ge`, `lt`, `le`, and `multiple_of`. `PositiveInt` is equivalent to `Field(gt=0)`. Multiple constraints can be combined on a single field.

### 06.03 - String Constraints
Length constraints (`min_length`, `max_length`) apply to both strings and sequences. `pattern` enables regex validation. Works with variadic tuples (`tuple[int, ...]`) for length checking.

### 06.04 - Default Factories
`default_factory` parameter ensures fresh default values per instance. Use `lambda` functions for dynamic defaults (e.g., `datetime.now()`). Pydantic automatically handles mutable defaults safely, but `default_factory` is explicit.

### 06.05 - Additional Field Configurations
Per-field overrides for model-level config: `strict` (lax/strict coercion), `validate_default`, `frozen` (immutable field), and `exclude` (omit from serialization). Field-level config takes precedence over model-level config.

### 06.06–06.07 - Project & Solution
Practical exercise applying field constraints and configuration overrides.

---

## Chapter 07 — Annotated Types

### 07.02 - Pydantic and Annotated Types
Using `Annotated` to attach `Field` metadata to types for reuse. Eliminates duplication when the same constraints appear across multiple fields or models.

### 07.03 - Annotated Types and Type Variables
`TypeVar` enables generic annotated types. Create reusable constrained collections (e.g., `BoundedList[int]`, `BoundedList[str]`) with a single type definition.

### 07.04 - String Constraints with StringConstraints
`StringConstraints` provides advanced string validation: `to_lower`, `to_upper`, `strip_whitespace`, `min_length`, `max_length`, and `pattern`. More powerful than `Field` for string-specific transformations combined with validation.

### 07.05–07.06 - Project & Solution
Practical exercise applying annotated types and string constraints.

---

## Chapter 08 — Custom Validators

### 08.02 - After Validators
Default validator mode — runs after Pydantic's built-in validation. Receives already-coerced, validated values. Raise `ValueError` for validation failures (Pydantic converts to `ValidationError`). Can transform values. Apply to multiple fields or all fields with `"*"`.

### 08.03 - Before Validators
Runs before Pydantic validation (`mode="before"`). Receives raw input data of any type. Used for custom deserialization or preprocessing. Multiple before validators execute bottom-to-top.

### 08.04 - Combining Before and After Validators
Execution order: before (bottom → top) → Pydantic built-in → after (top → bottom). Powerful for preprocessing raw data then post-processing validated values.

### 08.05 - Custom Validators Using Annotations
Define validators as standalone functions attached via `BeforeValidator()` and `AfterValidator()` in `Annotated`. Promotes reusability across models without class methods. Multiple validators in a single annotation.

### 08.06 - Dependent Field Validations
Access previously validated fields via `ValidationInfo.data`. Only fields defined before the current field are available. Always check if the dependent field exists in `data` before using it.

### 08.07–08.08 - Project & Solution
Practical exercise combining before/after validators with dependent field validation.

---

## Chapter 09 — Properties and Computed Fields

### 09.02 - Properties
Standard Python `@property` works on Pydantic models. Properties are not model fields — they are not serialized, not in `model_dump()`, and not in repr. `@cached_property` available for expensive computations (freeze relevant fields to avoid stale caches).

### 09.03 - Computed Fields
`@computed_field` makes properties behave like model fields — serialized in `model_dump()` and `model_dump_json()`. Return type annotation is required. Supports aliases and `repr=False`. Can combine with `@cached_property` for efficiency.

### 09.04–09.05 - Project & Solution
Practical exercise implementing properties and computed fields.

---

## Chapter 10 — Custom Serializers using Annotated Types

### 10.02 - Custom Serializers
`PlainSerializer` replaces Pydantic's default serialization for a type. Attach to `Annotated` types alongside validators for a complete reusable type (validation + serialization in one definition). `when_used` parameter controls when custom serialization applies: `"always"`, `"json"`, `"json-unless-none"`, `"unless-none"`.

### 10.03–10.04 - Project & Solution
Practical exercise building a complete annotated type with `BeforeValidator`, `AfterValidator`, and `PlainSerializer`.

---

## Chapter 11 — Complex Models

### 11.02 - Model Composition
Nest Pydantic models as fields in other models. Each sub-model independently configured (e.g., `extra="ignore"` to filter unwanted nested data). Automatic nested serialization and deserialization.

### 11.03 - Model Inheritance
Create a custom base model with shared `ConfigDict` and fields. All child models inherit configuration. Useful for standardizing behavior across an application (e.g., alias generators, extra-field handling). Avoid multiple inheritance in Pydantic v2. Enables response wrapper patterns with common metadata (request ID, timestamps).

### 11.04–11.06 - Project & Solution
Practical exercise building complex nested models with shared base configuration.

---

## Chapter 12 — Applications

### 12.02 - Consuming a REST API
Create Pydantic models matching API response schemas. Use `model_validate()` with `response.json()`. Set `extra="ignore"` to handle unexpected fields. Custom validators clean up inconsistent values (e.g., converting `"Unknown"` to `None`).

### 12.03 - Ingesting a CSV File
CSV data is all strings — use `BeforeValidator` for custom type parsing (e.g., stripping commas from numbers). `csv.DictReader` maps columns to field names. Generator functions (`yield`) for memory-efficient row-by-row processing.

### 12.04 - Validating Function Arguments
`@validate_call` decorator applies Pydantic validation to function arguments. Works with `Annotated` types for argument constraints. Raises `ValidationError` before the function executes. Type coercion applies to arguments.

### 12.05 - Model Code Generators
`datamodel-code-generator` CLI tool auto-generates Pydantic models from JSON Schema, OpenAPI, JSON data, and CSV files. Works well for simple/moderate schemas but cannot capture complex conditional logic (e.g., if-else schema rules). Generated models often need manual refinement — use as a starting point. CSV generation produces all-string fields; manual validators still needed for type conversion.
