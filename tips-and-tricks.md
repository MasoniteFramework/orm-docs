# Tips & Tricks

## Dynamic Scope using Enums

### The Problem
During database design it is common to have multiple tables that have the same column
name that are used for the same purpose but contain different values based on the useage of the table.

An example of this might be a `status` column which can have different values depending on the table it is used in.

One way to address this is to have a global `StatusMixin` which is applied to every table and contains every value and it's 
accessor methodsthat could be used in any table.
This can quickly become very complicated especially if some values can have multiple contexts.
An example is the value `PENDING` 

This also means that any value can also be used in any table which could create data integirty issues.

### Solution - Dynamic Scope using Enum for context
#### Overview
The global scope uses a provided `Enum` class to provide scope methiods named `is_<enum value>` for the class it is 
attached to.

This means there is no accidental usage of methids which are not used in the table context. 
It also throw `KeyError` for invalid string values used via the `has_status()` method  

#### Setup
```python
# statusMixin.py
from .statusScope import StatusScope

class StatusMixin:
    """Mixin to add methods for status filtering"""

    def boot_StatusMixin(self, builder):
        if not hasattr(self, "__statuses__"):
            raise AttributeError("__statuses__ is not defined")
        builder.set_global_scope(StatusScope(getattr(self, "__statuses__")))
```
```python
# statusScope.py
from enum import Enum
from functools import partial
from masoniteorm.scopes.BaseScope import BaseScope

class StatusScope(BaseScope):
    """Global scope to add methods for status column filtering"""

    def __init__(self, statuses):
        self.__statuses: Enum = statuses

    def on_boot(self, builder):
        """Setup global scopes"""
        
        # create an 'is_xxx for each of the status items
        for status in self.__statuses:
            value = status.value
            method_name = f"is_{value.lower()}"
            builder.macro(
                method_name, partial(self._has_status, status=status)
            )

        builder.macro("has_status", self._has_status)

    def _has_status(self, model, builder, status: str | Enum):
        """
        Filter the model by status
        """
        if isinstance(status, str):
            status = self.__statuses[status.upper()]

        return builder.where("status", status.value)
```
```python
# company.py
from masoniteorm import Model
from .statusMixin import StatusMixin


class CompanyStatus(Enum):
    INITIATED = "INITIATED"
    PENDING = "PENDING"
    CONFIRMED = "CONFIRMED"


class Company(Model, StatusMixin):
    __statuses__ = CompanyStatus
    ...
```
```python
# invoice.py
from masoniteorm import Model
from .statusMixin import StatusMixin


class InvoiceStatus(Enum):
    RAISED = "RAISED"
    PAYMENT_PENDING = "PAYMENT_PENDING"
    PAYMENT_PROCESSED = "PAYMENT_PROCESSED"
    CARD_ISSUE = "CARD_ISSUE"
    CANCELED = "CANCELED"

    
class Invoice(Model, StatusMixin):
    __statuses__ = InvoiceStatus
    ...
```
#### Usage
Example usage  
```python
from .company import Company
from .invoice import Invoice

# using the created method
confirmed_companys = Company.is_confirmed().get()

# using a string
pending_invoices = Invoice.has_status("payment_pending").get()

# using an Enum is better than a string 
processed_invoices = Invoice.has_status(InvoiceStatus.PAYMENT_PROCESSED).get()

```
