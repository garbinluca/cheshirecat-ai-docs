# Logging

`CCAT_LOG_LEVEL` env var che definisce il minimo valore di log visualizzato.

I valori disponibili per i livelli di log sono:
- `DEBUG`
- `INFO`
- `WARNING`
- `ERROR`
- `CRITICAL`

Se ad esempio è impostato `CCAT_LOG_LEVEL=CRITICAL`, tutti gli errori di importanza inferiore vengono ignorati.

## Utilizzo
```python
from cat.log import log
log.error("A simple text here")
log.info(f"Value of user message is {user_message_json["text"]}")
log.critical(variable_value)

```