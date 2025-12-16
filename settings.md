# Settings

Definizione delle impostazioni del plugin, con la possibilità di poterle modificare dal pannello di amministrazione.

Per definire le impostazioni del plugin è possibile procedere in due modi:
- definendo `settings_schema` e decorandolo con `@plugin`. La funzione deve ritornare un [JSON schema](https://json-schema.org/). Utile se si definisce la struttura in un file json caricato da disco.
- definendo `settings_model` e decorandolo con `@plugin`.

Esempio:
```python
from pydantic import BaseModel
from enum import Enum
from datetime import date, time
from cat.mad_hatter.decorators import plugin

# select box
#   (will be used in class DemoSettings below to give a multiple choice setting)
class NameSelect(Enum):
    a: str = 'Nicola'
    b: str = 'Emanuele'
    c: str = 'Daniele'

# settings
class DemoSettings(BaseModel):

    # Integer
    #   required setting
    required_int: int
    #   optional setting, with default value
    optional_int: int = 42

    # Float
    required_float: float
    optional_float: float = 12.95

    # String
    required_str: str
    optional_str: str = "stocats"

    # Boolean
    required_bool: bool
    optional_bool_true: bool = True

    # Date
    required_date: date
    optional_date: date = date(2020, 11, 2)

    # Time
    required_time: time
    optional_time: time = time(4, 12, 54)

    # Select
    required_enum: NameSelect
    optional_enum: NameSelect = NameSelect.b

# Give your settings model to the Cat.
@plugin
def settings_model():
    return DemoSettings
```

Accesso alle impostazioni del plugin
```python
# Lettura
settings = cat.mad_hatter.get_plugin().load_settings()

# Scrittura
settings = cat.mad_hatter.get_plugin().save_settings(settings)
```
