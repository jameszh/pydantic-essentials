# Pydantic Essentials Cheat Sheet (Sections 02–05)

## Model Basics

```python
from pydantic import BaseModel

class User(BaseModel):
    name: str
    age: int
    email: str | None = None  # optional + nullable
```

### Deserialization
```python
u = User(name="Alice", age=30)              # constructor
u = User.model_validate({"name": "Alice", "age": 30})  # from dict
u = User.model_validate_json('{"name":"Alice","age":30}')  # from JSON
```

### Serialization
```python
u.model_dump()                        # → dict
u.model_dump_json()                   # → JSON string
u.model_dump(include={"name"})        # only specific fields
u.model_dump(exclude={"email"})       # exclude fields
u.model_dump(by_alias=True)           # use aliases as keys
```

### Field Kinds
```python
name: str                  # required, non-nullable
name: str | None           # required, nullable
name: str = "default"      # optional, non-nullable
name: str | None = None    # optional, nullable
```

### Inspection
```python
User.model_fields              # schema-level field info
u.model_fields_set             # fields caller actually provided
User.model_json_schema()       # JSON Schema dict
```

---

## Model Configuration

```python
from pydantic import ConfigDict

class MyModel(BaseModel):
    model_config = ConfigDict(
        extra="forbid",              # "ignore" | "forbid" | "allow"
        strict=True,                 # disable lax type coercion
        frozen=True,                 # immutable + hashable
        validate_default=True,       # validate default values
        validate_assignment=True,    # validate on attribute set
        coerce_numbers_to_str=True,  # 123 → "123"
        str_strip_whitespace=True,   # strip leading/trailing spaces
        str_to_lower=True,           # normalize to lowercase
        use_enum_values=True,        # store raw value, not Enum member
        populate_by_name=True,       # accept field name OR alias
    )
```

---

## Field Aliases

```python
from pydantic import Field
from pydantic.alias_generators import to_camel

class Order(BaseModel):
    model_config = ConfigDict(alias_generator=to_camel)

    order_id: int                                          # alias → "orderId"
    status: str = Field(alias="order_status")              # manual alias
    code: str = Field(validation_alias="source_code")      # deserialization only
    label: str = Field(serialization_alias="display_label")# serialization only
```

### AliasChoices (accept multiple names)
```python
from pydantic import AliasChoices

name: str = Field(validation_alias=AliasChoices("name", "full_name", "userName"))
```

### Custom Serializer
```python
from pydantic import field_serializer
from datetime import datetime

class Event(BaseModel):
    ts: datetime

    @field_serializer("ts")
    @classmethod
    def fmt_ts(cls, v: datetime, _info) -> str:
        return v.strftime("%Y-%m-%d")
```

`when_used` options: `"always"` | `"json"` | `"unless-none"` | `"json-unless-none"`

---

## Specialized Types

```python
from pydantic import PositiveInt, NegativeInt, NonNegativeInt
from pydantic import conlist
from pydantic import UUID4, Field
from pydantic import PastDate, PastDatetime, NaiveDatetime, AwareDatetime
from pydantic import AnyUrl, HttpUrl, IPvAnyAddress
from pydantic import EmailStr, NameEmail  # requires email-validator
from uuid import uuid4

class Item(BaseModel):
    qty: PositiveInt                                    # > 0
    scores: conlist(int, min_length=1, max_length=5)    # constrained list
    id: UUID4 = Field(default_factory=uuid4)            # auto-generate UUID
    born: PastDate                                      # must be in the past
    created: AwareDatetime                              # must have timezone
    site: HttpUrl
    email: EmailStr
    ip: IPvAnyAddress
```

| Type | Constraint |
|------|-----------|
| `PositiveInt` | `> 0` |
| `NegativeInt` | `< 0` |
| `NonNegativeInt` | `>= 0` |
| `conlist(T, min_length, max_length)` | list length bounds |
| `UUID4` | valid UUID v4 |
| `PastDate` / `PastDatetime` | must be in the past |
| `NaiveDatetime` | no timezone |
| `AwareDatetime` | requires timezone |
| `HttpUrl` | valid HTTP(S) URL |
| `EmailStr` | valid email |
| `IPvAnyAddress` | IPv4 or IPv6 |
