# Tool

Esempio di come funziona un tool:

```python
@tool
def socks_prices(color, cat):
    """How much do socks cost? Input is the sock color."""
    prices = {
        "black": 5,
        "white": 10,
        "pink": 50,
    }
    if color not in prices.keys():
        return f"No {color} socks"
    else:
        return f"{prices[color]} €"
```

Quando l'utente chiede "Quanto costano i calzini rosa?", il Gatto:
- prende tutti i tool disponibili con le relative descrizioni e li passa all'LLM
- LLM sceglie, in base al contesto della conversazione, quale tool eseguire. L'output dell'LLM sarà quindi:

```json
{
    "action": "socks_prices",
    "action_input": "pink"
}
```

L'output verrà inviato all'agente o direttamente visualizzato in chat, in base all'impostazione `@tool(return_direct=True)`.
I tool sono utili per:
- comunicare con un webservice
- cercare informazioni in un database esterno
- eseguire operazioni matematiche
- eseguire cose nel terminale (danger zone)
- tenere traccia di informazioni specifiche e fare cose fantasiose con esse
- Interagisci con altri componenti del Gatto come llm, embedder, working memory, vector memory, white rabbit, rabbit hole ecc.

```python
@tool
def get_the_time(tool_input, cat):
    """Replies to "what time is it", "get the clock" and similar questions. Input is always None.."""
    return str(datetime.now())
```

## Documentazione

I docstring devono essere composti nel seguente modo: `"""When to use the tool. Tool input description"""`

```python
@tool(
    return_direct=False,
    examples=["what time is it", "get the time"]
)
def get_the_time(tool_input, cat):
    """Useful to get the current time when asked. Input is always None."""
    return f"The current time is {str(datetime.now())}"
```

- Il parametro `tool_input` è una stringa, quindi se viene richiesto, nella descrizione (docstring), di produrre un intero o un dict, assicurarsi di effettuare un cast o parsare la stringa
- Il parametro `cat` permette di acccedere ai componenti del framework. [Documentazione](https://cheshire-cat-ai.github.io/docs/framework/cat-components/cheshire_cat/stray_cat/)

## Esempio complesso

```python
from currency_converter import CurrencyConverter
from cat.mad_hatter.decorators import tool

@tool(return_direct=True)
def convert_currency(tool_input, cat):
    """Useful to convert currencies. This tool converts euros (EUR) to another currency.
    The inputs are two values separated with a minus: the first one is a number;
    the second one is the name of a currency. Example input: '15-GBP'.
    Use when the user says something like: 'convert 15 EUR to GBP'"""

    # Currency Converter
    converter = CurrencyConverter(decimal=True)

    # Parse the input
    parsed_input = tool_input.split("-")

    # Check input is correct
    if len(parsed_input) == 2:  # 
        eur, currency = parsed_input[0].strip("'"), parsed_input[1].strip("'")
    else:
        return "Something went wrong using the tool"

    # Use the LLM to convert the currency name into its symbol
    symbol = cat.llm(f"You will be given a currency code, translate the input in the corresponding currency symbol. \
                    Examples: \
                        euro -> € \
                        {currency} -> [answer here]")  # 
    # Remove new line if any
    symbol = symbol.strip("\n")

    # Check the currencies are in the list of available ones
    if currency not in converter.currencies:
        return f"{currency} is not available"

    # Convert EUR to currency
    result = converter.convert(float(eur), "EUR", currency)

    return f"{eur}€ = {float(result):.2f}{symbol}"
```