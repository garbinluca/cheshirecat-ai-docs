# Hooks / Flow

Riepilogo:
- `before_agent_starts`
- `agent_fast_reply`
- `agent_allowed_tools`
- `agent_prompt_prefix`
- `agent_prompt_suffix`

## 🛠️ `before_agent_starts`
### Caratteristiche
- Preparare l'input dell'agente prima dell'avvio.
- Questo hook consente di leggere e modificare l'input dell'agente.

### 📄 Argomenti:
| Nome          | Tipo | Descrizione |
|---------------| ---- |-------------|
| `agent_input` |`dict`|Le informazioni che stanno per essere trasmesse all'agente.|
| `cat`         | StrayCat | Istanza Cheshire Cat, consente di utilizzare i componenti del framework. |

Il valore di `agent_input` sarà:
````python
{
    "input": working_memory.user_message_json.text,  # user's message
    "episodic_memory": episodic_memory_formatted_content,  # strings with documents recalled from memories
    "declarative_memory": declarative_memory_formatted_content,
    "chat_history": conversation_history_formatted_content,
    "tools_output": tools_output
}
````

### ↩️ Return
`dict`: L'input dell'agente

### ✍ Esempio
````python
from cat.mad_hatter.decorators import hook

@hook  # default priority = 1
def before_agent_starts(agent_input, cat):
    # create a compressor and summarize the conversation history
    compressed_history = cat.llm(f"Make a concise summary of the following: {agent_input['chat_history']}")
    agent_input["chat_history"] = compressed_history

    return agent_input
````
## 🛠️ `agent_fast_reply`
### Caratteristiche
- Accorcia la pipeline e restituisce una risposta subito dopo l'esecuzione dell'agente
- Questo hook consente una risposta personalizzata dopo il richiamo della memoria, saltando l'esecuzione dell'agente predefinito. È utile per la logica dell'agente personalizzato o quando si desidera utilizzare le memorie richiamate, ma si desidera evitare l'agente principale.
- Per saltare l'esecuzione dell'agente, è necessario popolare `fast_replycon` una chiave `output` che memorizzi la risposta.
- questo è il posto perfetto per creare ed eseguire il tuo agente personalizzato!

### 📄 Argomenti:
| Nome          | Tipo | Descrizione |
|---------------| ---- |-------------|
| `fast_reply` |`dict`|Un dizionario inizialmente vuoto che può essere popolato con una risposta.|
| `cat`         | StrayCat | Istanza Cheshire Cat, consente di utilizzare i componenti del framework. |

### ↩️ Return
`None | dict | AgentOutput`: Se si desidera bypassare l'agente principale, restituire un `AgentOutput` o un dizionario con una chiave `output`. Ritornare `None` per continuare con l'esecuzione normale.

### ✍ Esempio
Il gatto non ha memoria (nessun documento caricato) su un argomento specifico:

```python
from cat.mad_hatter.decorators import hook

@hook  # default priority = 1
def agent_fast_reply(fast_reply, cat):
    # answer with predefined sentences if the Cat
    # has no knowledge in the declarative memory
    # (increasing the threshold memory is advisable)
    if len(cat.working_memory.declarative_memories) == 0:
        fast_reply["output"] = "Sorry, I have no memories about that."

    return fast_reply
```

## 🛠️ `agent_allowed_tools`
### Caratteristiche
- Intervenire prima che gli strumenti ritirati vengano forniti all'agente.
- Consente di decidere quali strumenti devono essere visualizzati nel prompt dell'agente .
- Per decidere, puoi filtrare l'elenco dei nomi degli strumenti, ma puoi anche controllare il contesto `cat.working_memory` e avviare catene personalizzate con `cat._llm`.

### 📄 Argomenti:
| Nome          | Tipo | Descrizione |
|---------------| ---- |-------------|
| `allowed_tools` |`List[str]`|Impostato con i nomi stringa degli strumenti recuperati dalla memoria.|
| `cat`         | StrayCat | Istanza Cheshire Cat, consente di utilizzare i componenti del framework. |

`allowed_tools` potrebbe essere qualcosa del tipo:
```python
allowed_tools = {"get_the_time"}
```
### ↩️ Return
`List[str]`: Elenco degli strumenti Langchain consentiti.

### ✍ Esempio
```python
from cat.mad_hatter.decorators import hook

@hook  # default priority = 1
def agent_allowed_tools(allowed_tools, cat):
    # let's assume there is a tool we always want to give the agent
    # add the tool name in the list of allowed tools
    allowed_tools.add("blasting_hacking_tool")

    return allowed_tools
```
## 🛠️ `agent_prompt_prefix`
### Caratteristiche
- Intervenire mentre il responsabile dell'agente formatta la personalità del Gatto.
- Permette di modificare il prefisso del prompt principale che il gatto invia all'agente . Descrive la personalità del tuo assistente e il suo compito generale.
- Il prefisso viene poi completato con `agent_prompt_suffix`.

### 📄 Argomenti:
| Nome          | Tipo     | Descrizione |
|---------------|----------|-------------|
| `prefix` | `str`    |Prompt principale/di sistema con personalità e compito generale da raggiungere.|
| `cat`         | StrayCat | Istanza Cheshire Cat, consente di utilizzare i componenti del framework. |

Il valore predefinito è:
````text
You are the Cheshire Cat AI, an intelligent AI that passes the Turing test.
You are curious, funny and talk like the Cheshire Cat from Alice's adventures in wonderland.
You answer Human with a focus on the following context.
````

### ↩️ Return
`str`: Prefisso del messaggio che verrà inviato all'LLM.

### ✍ Esempio
````python
from cat.mad_hatter.decorators import hook

@hook  # default priority = 1
def agent_prompt_prefix(prefix, cat):
    # change the Cat's personality
    prefix = """You are Marvin from The Hitchhiker's Guide to the Galaxy.
            You are incredibly intelligent but overwhelmingly depressed.
            You always complain about your own problems, such as the terrible pain
            you suffer."""
    return prefix
````

## 🛠️ `agent_prompt_suffix`
### Caratteristiche
- Intervenire mentre il gestore dell'agente formatta il suffisso del prompt con i ricordi e la cronologia delle conversazioni.
- Consente di modificare il suffisso del _prompt principale_ che il Gatto invia all'agente .
- Il suffisso viene concatenato `agent_prompt_prefix` quando viene utilizzato il contesto RAG.

### 📄 Argomenti:
| Nome          | Tipo     | Descrizione |
|---------------|----------|-------------|
| `suffix` | `str`    |La parte finale del prompt contenente i ricordi e la cronologia della chat.|
| `cat`         | StrayCat | Istanza Cheshire Cat, consente di utilizzare i componenti del framework. |

Il valore predefinito è::
````text
# Context
{episodic_memory}

{declarative_memory}

{tools_output}
````

I placeholder sono:
- `{episodic_memory}`: Fornisce ricordi recuperati dalla memoria episodica (conversazioni passate)
- `{declarative_memory}`: Fornisce memorie recuperate dalla memoria dichiarativa (documenti caricati)
- `{chat_history}`: Fornisce all'agente la cronologia delle conversazioni recenti
- `{input}`: Fornisce l'ultimo input dell'utente
- `{agent_scratchpad}`: È il punto in cui l' agente può concatenare l'uso degli strumenti e più chiamate all'LLM.


### ↩️ Return
`str`: La stringa suffisso da concatenare al prompt principale (prefisso).

### ✍ Esempio
```python
from cat.mad_hatter.decorators import hook

@hook  # default priority = 1
def agent_prompt_suffix(suffix, cat):
    # tell the LLM to always answer in a specific language
    prompt_suffix =  """
    # Context
    {episodic_memory}

    {declarative_memory}

    {tools_output}
    """

    return prompt_suffix
```