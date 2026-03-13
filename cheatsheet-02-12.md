# Pydantic Essentials Cheat Sheet (Chapters 02–12)

## Chapter 02 — Basics

### 02.02 Creating a Model
```python
from pydantic import BaseModel

class User(BaseModel):
    name: str
    age: int
    email: str | None = None  # optional + nullable
```

### 02.03 Deserialization
```python
u = User(name="Alice", age=30)              # constructor
u = User.model_validate({"name": "Alice", "age": 30})  # from dict
u = User.model_validate_json('{"name":"Alice","age":30}')  # from JSON
```

### 02.04 Serialization
```python
u.model_dump()                        # → dict
u.model_dump_json()                   # → JSON string
u.model_dump(include={"name"})        # only specific fields
u.model_dump(exclude={"email"})       # exclude fields
u.model_dump(by_alias=True)           # use aliases as keys
```

### 02.06–02.08 Field Kinds
```python
name: str                  # required, non-nullable
name: str | None           # required, nullable
name: str = "default"      # optional, non-nullable
name: str | None = None    # optional, nullable
```

### 02.09 Inspection
```python
User.model_fields              # schema-level field info
u.model_fields_set             # fields caller actually provided
User.model_json_schema()       # JSON Schema dict (02.10)
```

---

## Chapter 03 — Model Configuration

### 03.02–03.09 ConfigDict Options
```python
from pydantic import ConfigDict

class MyModel(BaseModel):
    model_config = ConfigDict(
        extra="forbid",              # 03.02: "ignore" | "forbid" | "allow"
        strict=True,                 # 03.03: disable lax type coercion
        validate_default=True,       # 03.04: validate default values
        validate_assignment=True,    # 03.05: validate on attribute set
        frozen=True,                 # 03.06: immutable + hashable
        coerce_numbers_to_str=True,  # 03.07: 123 → "123"
        str_strip_whitespace=True,   # 03.08: strip leading/trailing spaces
        str_to_lower=True,           # 03.08: normalize to lowercase
        use_enum_values=True,        # 03.09: store raw value, not Enum member
        populate_by_name=True,       # accept field name OR alias
    )
```

---

## Chapter 04 — Field Aliasing, Serialization and Deserialization

### 04.02–04.03 Field Aliases
```python
from pydantic import Field
from pydantic.alias_generators import to_camel

class Order(BaseModel):
    model_config = ConfigDict(alias_generator=to_camel)  # 04.03

    order_id: int                                          # alias → "orderId"
    status: str = Field(alias="order_status")              # 04.02: manual alias
    code: str = Field(validation_alias="source_code")      # 04.06: deserialization only
    label: str = Field(serialization_alias="display_label")# 04.05: serialization only
```

### 04.04 Deserializing by Field Name or Alias
```python
model_config = ConfigDict(populate_by_name=True)
# Now accepts both field name and alias during deserialization
```

### 04.06 AliasChoices (accept multiple names)
```python
from pydantic import AliasChoices

name: str = Field(validation_alias=AliasChoices("name", "full_name", "userName"))
```

### 04.07 Custom Serializer
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

## Chapter 05 — Specialized Pydantic Types

### 05.02–05.06 Type Reference
```python
from pydantic import PositiveInt, NegativeInt, NonNegativeInt  # 05.02
from pydantic import conlist                                    # 05.03
from pydantic import UUID4, Field                               # 05.04
from pydantic import PastDate, PastDatetime                     # 05.05
from pydantic import NaiveDatetime, AwareDatetime               # 05.05
from pydantic import AnyUrl, HttpUrl, IPvAnyAddress             # 05.06
from pydantic import EmailStr, NameEmail                        # 05.06 (requires email-validator)
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

| Type | Section | Constraint |
|------|---------|-----------|
| `PositiveInt` | 05.02 | `> 0` |
| `NegativeInt` | 05.02 | `< 0` |
| `NonNegativeInt` | 05.02 | `>= 0` |
| `conlist(T, min_length, max_length)` | 05.03 | list length bounds |
| `UUID4` | 05.04 | valid UUID v4 |
| `PastDate` / `PastDatetime` | 05.05 | must be in the past |
| `NaiveDatetime` | 05.05 | no timezone |
| `AwareDatetime` | 05.05 | requires timezone |
| `HttpUrl` | 05.06 | valid HTTP(S) URL |
| `EmailStr` | 05.06 | valid email |
| `IPvAnyAddress` | 05.06 | IPv4 or IPv6 |

---

## Chapter 06 — Additional Field Features

### 06.02 Numerical Constraints
```python
from pydantic import Field

number: float = Field(gt=0, le=100)           # 0 < x <= 100
step: int = Field(ge=0, lt=50, multiple_of=5) # 0 <= x < 50, multiples of 5
```

### 06.03 String & Sequence Constraints
```python
name: str = Field(min_length=1, max_length=50)
zip_code: str = Field(pattern=r"^[0-9]{5}(?:-[0-9]{4})?$")
items: list[float] = Field(min_length=1, max_length=10)
coords: tuple[int, ...] = Field(min_length=2, max_length=3)
```

### 06.04 Default Factories
```python
from datetime import datetime, UTC

dt: datetime = Field(default_factory=lambda: datetime.now(UTC))
tags: list[str] = Field(default_factory=list)
```

### 06.05 Per-Field Config Overrides
```python
class Model(BaseModel):
    model_config = ConfigDict(strict=False)

    strict_flag: bool = Field(strict=True)        # override: strict for this field
    internal: str = Field(exclude=True)            # never serialized
    locked: int = Field(frozen=True)               # immutable field
    checked: int = Field(validate_default=True)    # validate this default
```

---

## Chapter 07 — Annotated Types

### 07.02 Reusable Constrained Types
```python
from typing import Annotated

BoundedFloat = Annotated[float, Field(ge=0, le=1.0)]
ShortStr = Annotated[str, Field(min_length=1, max_length=50)]

class Model(BaseModel):
    score: BoundedFloat
    name: ShortStr
```

### 07.03 Generic Constraints with TypeVar
```python
from typing import TypeVar
T = TypeVar("T")
BoundedList = Annotated[list[T], Field(max_length=10)]

class Model(BaseModel):
    ints: BoundedList[int] = []
    strs: BoundedList[str] = []
```

### 07.04 StringConstraints
```python
from pydantic import StringConstraints

CleanStr = Annotated[str, StringConstraints(
    strip_whitespace=True, to_lower=True, min_length=1, max_length=100
)]
```

---

## Chapter 08 — Custom Validators

### 08.02 After Validator (default — runs after Pydantic validation)
```python
from pydantic import field_validator

class Model(BaseModel):
    price: float = Field(gt=0)

    @field_validator("price")
    @classmethod
    def round_price(cls, value: float) -> float:
        return round(value, 2)

    # Apply to multiple fields:
    @field_validator("cost", "price")
    @classmethod
    def round_money(cls, v: float) -> float:
        return round(v, 2)

    # Apply to ALL fields:
    @field_validator("*")
    @classmethod
    def round_all(cls, v: float) -> float:
        return round(v, 2)
```

### 08.03 Before Validator (runs before Pydantic validation)
```python
class Model(BaseModel):
    dt: datetime

    @field_validator("dt", mode="before")
    @classmethod
    def parse_dt(cls, value):
        if isinstance(value, str):
            return dateutil.parser.parse(value)
        return value
```

### 08.05 Validators via Annotated Types (reusable)
```python
from pydantic import BeforeValidator, AfterValidator

def parse_datetime(value):
    if isinstance(value, str):
        return dateutil.parser.parse(value)
    return value

def make_utc(dt: datetime) -> datetime:
    if dt.tzinfo is None:
        dt = pytz.utc.localize(dt)
    else:
        dt = dt.astimezone(pytz.utc)
    return dt

DateTimeUTC = Annotated[datetime, BeforeValidator(parse_datetime), AfterValidator(make_utc)]
```

Execution order (08.04): **before** (bottom → top) → **Pydantic** → **after** (top → bottom)

### 08.06 Dependent Field Validation
```python
from pydantic import ValidationInfo

class Model(BaseModel):
    start: datetime
    end: datetime

    @field_validator("end")
    @classmethod
    def end_after_start(cls, value, info: ValidationInfo):
        if "start" in info.data and value <= info.data["start"]:
            raise ValueError("end must be after start")
        return value
```

---

## Chapter 09 — Properties and Computed Fields

### 09.02 Properties (not serialized)
```python
class Circle(BaseModel):
    radius: float = Field(gt=0)

    @property
    def area(self) -> float:
        return 3.14159 * self.radius ** 2
```

### 09.03 Computed Fields (serialized like regular fields)
```python
from pydantic import computed_field
from functools import cached_property

class Circle(BaseModel):
    radius: float = Field(gt=0, frozen=True)

    @computed_field(alias="AREA", repr=False)
    @cached_property
    def area(self) -> float:          # return type REQUIRED
        return 3.14159 * self.radius ** 2

c = Circle(radius=5)
c.model_dump()              # {'radius': 5.0, 'area': 78.5...}
c.model_dump(by_alias=True) # {'radius': 5.0, 'AREA': 78.5...}
```

---

## Chapter 10 — Custom Serializers using Annotated Types

### 10.02 PlainSerializer in Annotated Types
```python
from pydantic import PlainSerializer

def fmt_dt(dt: datetime) -> str:
    return dt.strftime("%Y/%m/%d %I:%M %p UTC")

DateTimeUTC = Annotated[
    datetime,
    BeforeValidator(parse_datetime),
    AfterValidator(make_utc),
    PlainSerializer(fmt_dt, when_used="json-unless-none"),
]

# model_dump()      → datetime object (Python)
# model_dump_json() → "2020/01/01 03:00 PM UTC" (custom string)
```

---

## Chapter 11 — Complex Models

### 11.02 Model Composition (nested models)
```python
class Address(BaseModel):
    model_config = ConfigDict(extra="ignore")
    city: str
    country: str

class Person(BaseModel):
    name: str
    address: Address

data = {"name": "Alice", "address": {"city": "Paris", "country": "FR", "zip": "75001"}}
p = Person.model_validate(data)  # zip silently ignored
```

### 11.03 Model Inheritance (shared config base)
```python
from pydantic.alias_generators import to_camel

class AppBaseModel(BaseModel):
    model_config = ConfigDict(
        extra="ignore",
        alias_generator=to_camel,
        populate_by_name=True,
    )

class User(AppBaseModel):
    first_name: str     # alias → "firstName"

class Order(AppBaseModel):
    order_id: int       # alias → "orderId"
```

### 11.03 Response Wrapper Pattern
```python
class RequestInfo(AppBaseModel):
    query_id: UUID4 = Field(default_factory=uuid4)
    execution_dt: DateTimeUTC = Field(default_factory=...)

class ResponseBase(AppBaseModel):
    request_info: RequestInfo

class UsersResponse(ResponseBase):
    users: list[User] = []
```

---

## Chapter 12 — Applications

### 12.02 Consuming a REST API
```python
import requests

class IPGeo(BaseModel):
    model_config = ConfigDict(extra="ignore")
    ip: IPvAnyAddress
    country: str | None = None

    @field_validator("country", mode="after")
    @classmethod
    def unknown_to_none(cls, v):
        return None if v and v.casefold() == "unknown" else v

geo = IPGeo.model_validate(requests.get("https://api.example.com/geo/8.8.8.8").json())
```

### 12.03 Ingesting CSV Data
```python
import csv

FunkyInt = Annotated[int, BeforeValidator(lambda v: int(v.strip().replace(",", "")))]

class Row(BaseModel):
    area: str
    population: FunkyInt

def load_csv(path):
    with open(path) as f:
        reader = csv.DictReader(f, fieldnames=["area", "population"])
        next(reader)  # skip header
        for row in reader:
            yield Row.model_validate(row)
```

### 12.04 Validating Function Arguments
```python
from pydantic import validate_call

NonEmpty = Annotated[str, Field(min_length=1)]

@validate_call
def greet(name: NonEmpty, times: int = 1) -> str:
    return f"Hello, {name}! " * times

greet("Alice")         # works
greet("")              # ValidationError
greet("Alice", "3")    # coerces "3" → 3, works
```
