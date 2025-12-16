# Hooks / Flow

Riepilogo:
- `before_cat_reads_message`
- `cat_recall_query`
- `before_cat_recalls_memories`
- `before_cat_recalls_episodic_memories`
- `before_cat_recalls_declarative_memories`
- `before_cat_recalls_procedural_memories`
- `after_cat_recalls_memories`
- `before_cat_stores_episodic_memory`
- `before_cat_sends_message`

## 🛠️ `before_cat_reads_message`
### Caratteristiche
- Intervenire non appena viene ricevuto un messaggio WebSocket.
- Consente di modificare e arricchire il messaggio in arrivo ricevuto dalla connessione WebSocket.
- può essere utilizzato per tradurre il messaggio dell'utente prima di inviarlo al Gatto
- Aggiunge chiavi personalizzate al dizionario JSON:
  ```json
  {
      "text": "<message content>"
  }
  ```

### 📄 Argomenti:

| Nome                | Tipo     | Descrizione                                                              |
|---------------------|----------|--------------------------------------------------------------------------|
| `user_message_json` | `dict`   | Dizionario JSON con il messaggio ricevuto dalla chat.                    |
| `cat`               | StrayCat | Istanza Cheshire Cat, consente di utilizzare i componenti del framework. |

### ↩️ Return
`dict`: Dizionario JSON modificato che verrà inserito nel Cat.

### ✍ Esempio

````python
from cat.mad_hatter.decorators import hook

@hook  # default priority = 1 
def before_cat_reads_message(user_message_json, cat):
    # Aggiunge dati alla memoria di lavoro
    user_message_json["text"] = "The original message has been replaced"
    cat.working_memory.hacked = True
    
    # oppure:
    user_message_json["custom_key"] = True

    return user_message_json
````

## 🛠️ `cat_recall_query`

### Caratteristiche
- Intervenire prima che la query di richiamo venga incorporata, agganciandosi alla query di ricerca semantica.
- Consente di modificare il messaggio dell'utente, utilizzato come una query per il recupero del contesto dalle memorie. Di conseguenza, il contesto recuperato può essere condizionato modificando il messaggio dell'utente.
- E' adatto per eseguire l'Hypothetical Document Embedding (HyDE). La strategia HyDE 1 sfrutta il messaggio dell'utente per generare una risposta ipotetica. Questa viene poi utilizzata per richiamare il contesto rilevante dalla memoria.

### 📄 Argomenti:

| Nome                | Tipo     | Descrizione                                                              |
|---------------------|----------|--------------------------------------------------------------------------|
| `user_message` | `str`    | Messaggio dell'utente che verrà utilizzato per interrogare le memorie vettoriali |
| `cat`               | StrayCat | Istanza Cheshire Cat, consente di utilizzare i componenti del framework. |

### ↩️ Return
`str`: Stringa modificata da utilizzare per il recupero del contesto in memoria. La stringa restituita viene ulteriormente memorizzata nella memoria di lavoro in `cat.working_memory.recall_query`.

### ✍ Esempio

```python
from cat.mad_hatter.decorators import hook

@hook  # default priority = 1
def cat_recall_query(user_message, cat):
    # Ask the LLM to generate an answer for the question
    new_query = cat.llm(f"If the input is a question, generate a plausible answer. Input --> {user_message}")

    # Replace the original message and use the answer as a query
    return new_query
```

## 🛠️ `before_cat_recalls_memories`

### Caratteristiche
- Intervenire prima che il Gatto cerchi nei ricordi specifici.
- Intercetta quando il Gatto interroga le memorie utilizzando l'input incorporato dell'utente.
- Viene eseguito appena prima che il Gatto cerchi il contesto significativo in entrambe le memorie e lo memorizzi nella memoria di lavoro.

### 📄 Argomenti:

| Nome                | Tipo     | Descrizione                                                              |
|---------------------|----------|--------------------------------------------------------------------------|
| `cat`               | StrayCat | Istanza Cheshire Cat, consente di utilizzare i componenti del framework. |

### ↩️ Return
Restituisce? Oh no, caro sviluppatore, questa funzione è un viaggio di sola andata nella tana del Bianconiglio 😺

### ✍ Esempio

```python
from cat.mad_hatter.decorators import hook

@hook  # default priority = 1
def before_cat_recalls_memories(cat):
    # do whatever here

```

## 🛠️ `before_cat_recalls_episodic_memories`

### Caratteristiche
- Intervenire prima che il Gatto cerchi nei messaggi degli utenti precedenti.
- Intercetta quando il Cat interroga le memorie utilizzando l'input incorporato dell'utente.
- Viene eseguito appena prima che il Cat cerchi il contesto significativo in entrambe le memorie e lo memorizzi nella memoria di lavoro.
- Restituisce i valori per il numero massimo (k) di elementi da recuperare dalla memoria e la soglia di punteggio applicata alla query nella memoria vettoriale. Restituisce inoltre la query incorporata (embedding) e le condizioni di richiamo (metadati).

### 📄 Argomenti:

| Nome                | Tipo     | Descrizione                                                              |
|---------------------|----------|--------------------------------------------------------------------------|
| `episodic_recall_config` | `dict`   | Dizionario con i dati necessari per richiamare i ricordi episodici |
| `cat`               | StrayCat | Istanza Cheshire Cat, consente di utilizzare i componenti del framework. |

Il valore predefinito `episodic_recall_config` è:
```json
{
    "embedding": recall_query_embedding,  # embedding of the recall query
    "k": 3,  # number of memories to retrieve
    "threshold": 0.7,  # similarity threshold to retrieve memories
    "metadata": {"source": self.user_id},  # dictionary of metadata to filter memories, by default it filters for user id
}
```

### ↩️ Return
`dict`: Dizionario modificato che verrà inserito come query nel database Vector.

### ✍ Esempio

```python
from cat.mad_hatter.decorators import hook

@hook  # default priority = 1
def before_cat_recalls_episodic_memories(episodic_recall_config, cat):
    # increase the number of recalled memories
    episodic_recall_config["k"] = 6

    return episodic_recall_config
```

## 🛠️ `before_cat_recalls_declarative_memories`

### Caratteristiche
- Intervenite prima che il Gatto cerchi nei documenti.
- Intercetta quando il Cat interroga le memorie utilizzando l'input incorporato dell'utente.
- L'hook viene eseguito appena prima che il Cat cerchi il contesto significativo in entrambe le memorie e lo memorizzi nella memoria di lavoro.
- Restituisce i valori per il numero massimo (k) di elementi da recuperare dalla memoria e la soglia di punteggio applicata alla query nella memoria vettoriale. Restituisce inoltre la query incorporata (embedding) e le condizioni di richiamo (metadati).

### 📄 Argomenti:

| Nome                | Tipo     | Descrizione                                                             |
|---------------------|----------|-------------------------------------------------------------------------|
| `declarative_recall_config` | `dict`   | Dizionario con i dati necessari per richiamare memorie dichiarative |
| `cat`               | StrayCat | Istanza Cheshire Cat, consente di utilizzare i componenti del framework. |

Il valore predefinito `declarative_recall_configè` è:
```json
{
    "embedding": recall_query_embedding,  # embedding of the recall query
    "k": 3,  # number of memories to retrieve
    "threshold": 0.7,  # similarity threshold to retrieve memories
    "metadata": None,  # dictionary of metadata to filter memories
}
```

### ↩️ Return
`dict`: Dizionario modificato che verrà inserito come query nel database Vector.

### ✍ Esempio

```python
from cat.mad_hatter.decorators import hook

@hook  # default priority = 1
def before_cat_recalls_declarative_memories(declarative_recall_config, cat):
    # filter memories using custom metadata. 
    # N.B. you must add the metadata when uploading the document! 
    declarative_recall_config["metadata"] = {"topic": "cats"}

    return declarative_recall_config
```

## 🛠️ `before_cat_recalls_procedural_memories`

### Caratteristiche
- Intervieni prima che il Gatto cerchi tra le azioni che conosce.
- Intercetta quando il Cat interroga le memorie utilizzando l'input incorporato dell'utente.
- L'hook viene eseguito appena prima che il Cat cerchi il contesto significativo in entrambe le memorie e lo memorizzi nella memoria di lavoro.
- Restituisce i valori per il numero massimo (k) di elementi da recuperare dalla memoria e la soglia di punteggio applicata alla query nella memoria vettoriale. Restituisce inoltre la query incorporata (embedding) e le condizioni di richiamo (metadati).

### 📄 Argomenti:

| Nome                | Tipo     | Descrizione                                                             |
|---------------------|----------|-------------------------------------------------------------------------|
| `procedural_recall_config` | `dict`   | Dizionario con i dati necessari per richiamare gli strumenti dalla memoria procedurale |
| `cat`               | StrayCat | Istanza Cheshire Cat, consente di utilizzare i componenti del framework. |

Il valore predefinito `procedural_recall_config` è:
```json
{
    "embedding": recall_query_embedding,  # embedding of the recall query
    "k": 3,  # number of memories to retrieve
    "threshold": 0.7,  # similarity threshold to retrieve memories
    "metadata": None,  # dictionary of metadata to filter memories
}
```

### ↩️ Return
`dict`: Dizionario modificato che verrà inserito come query nel database Vector.

### ✍ Esempio

```python
from cat.mad_hatter.decorators import hook

@hook  # default priority = 1
def before_cat_recalls_procedural_memories(procedural_recall_config, cat):
    # decrease the threshold to recall more tools
    declarative_recall_config["threshold"] = 0.5

    return procedural_recall_config
```

## 🛠️ `after_cat_recalls_memories`

### Caratteristiche
- Intervenire dopo che il Gatto ha richiamato il contenuto dei ricordi.
- L'hook viene eseguito subito dopo che il Cat ha cercato il contesto significativo nei ricordi e lo ha memorizzato nella memoria di lavoro.

### 📄 Argomenti:

| Nome                | Tipo     | Descrizione                                                             |
|---------------------|----------|-------------------------------------------------------------------------|
| `cat`               | StrayCat | Istanza Cheshire Cat, consente di utilizzare i componenti del framework. |

### ↩️ Return
Restituisce? Oh no, caro sviluppatore, questa funzione è un viaggio di sola andata nella tana del Bianconiglio 😺

### ✍ Esempio

```python
from cat.mad_hatter.decorators import hook

@hook  # default priority = 1
def after_cat_recalls_memories(cat):
    # do whatever here
```


## 🛠️ `before_cat_stores_episodic_memory`

### Caratteristiche
- Intervenire prima che il Gatto immagazzini ricordi episodici.
- Permette di intercettare il messaggio utente `Document` prima che venga inserito nella memoria vettoriale
- I `Document` possono quindi essere modificati e migliorati prima che il Gatto li memorizzi nella memoria vettoriale episodica.

### 📄 Argomenti:

| Nome  | Tipo     | Descrizione                                                             |
|-------|----------|-------------------------------------------------------------------------|
| `doc` | Document | Langchain Documentda inserire nella memoria. |
| `cat` | StrayCat | Istanza Cheshire Cat, consente di utilizzare i componenti del framework. |

`Document` ha due proprietà:
- `page_content`: la stringa con il testo da salvare in memoria;
- `metadata`: un dizionario con almeno due chiavi:
  - `source`: da dove proviene il testo;
  - `when`: timestamp per tenere traccia di quando è stato caricato.

### ↩️ Return
`Document`: Langchain `Document` che viene aggiunta nella memoria vettoriale episodica.

### ✍ Esempio

```python
from cat.mad_hatter.decorators import hook

@hook  # default priority = 1
def before_cat_stores_episodic_memory(doc, cat):
    if doc.metadata["source"] == "dolphin":
        doc.metadata["final_answer"] = 42 # Grande citazione!
    return doc
```


## 🛠️ `before_cat_sends_message`

### Caratteristiche
- Intervenire prima che il gatto invii la sua risposta tramite WebSocket.
- Consente di modificare il dizionario JSON che verrà inviato al client tramite connessione WebSocket
- Questo hook può essere utilizzato per modificare il messaggio inviato all'utente o per aggiungere chiavi al dizionario

### 📄 Argomenti:

| Nome  | Tipo   | Descrizione                                                             |
|-------|--------|-------------------------------------------------------------------------|
| `message` | `dict` | Dizionario JSON da inviare al client WebSocket. |
| `cat` | StrayCat | Istanza Cheshire Cat, consente di utilizzare i componenti del framework. |

### ↩️ Return
`dict`: Dizionario JSON modificato con la risposta del gatto.

### ✍ Esempio

```python
from cat.mad_hatter.decorators import hook

@hook  # default priority = 1
def before_cat_sends_message(message, cat):
    # use the LLM to rephrase the Cat's answer
    new_answer = cat.llm(f"Reformat this sentence like if you were a dog")  # Baauuuuu
    message["content"] = new_answer

    return message
```

