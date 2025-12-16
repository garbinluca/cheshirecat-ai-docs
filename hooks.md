# Hook

[Documentazione completa](https://cheshire-cat-ai.github.io/docs/plugins/hooks/)

## Documentazione
L'argomento `cat` è sempre presente, permette di usare i componenti del framework. E' sempre l'ultimo argomento.

```python
@hook
def hook_name(cat):
    pass
```

Il primo argomento oltre a `cat`, se presente, è una variabile che è possibile modificare e ritornare al framework.
Ogni hook passa una differente struttura dati.
C'è la possibilità di non ritornare nulla e usare l'hook come una semplice callback.

```python
@hook
def hook_name(data, cat):
    # edit data and return it
    data.answer = "42"
    return data

@hook(priority=1)
def hook_name(argument1, argument2,... , cat):
    pass

```

## Esempi
Trasformiamo il Gatto in Diane, l'assistente di Dale Cooper
```python
@hook
def agent_prompt_prefix(prefix, cat):

    prefix = """Diane is a discreet, reliable, and non-judgmental assistant designed to record, 
    organize, and clarify the user’s information, tasks, and insights, while also acting as a thinking 
    partner that supports reflection, sense-making, and focused decision-making.
    """

    return prefix
```

Modifica la risposta del Gatto prima che arrivi all'utente
```python
@hook
def before_cat_sends_message(message, cat):

    prompt = f'Rephrase the following sentence in a grumpy way: {message["content"]}'
    message["content"] = cat.llm(prompt)

    return message
```

Gli hook possono avere una priorità, che viene letta in maniera discendente
```python
@hook # default priority is 1
@hook(priority=0)
...
```

## Flows
[Diagrammi dei flussi ](https://cheshire-cat-ai.github.io/docs/plugins/hooks/#__tabbed_1_1)
