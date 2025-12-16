# StrayCat
`cat` è un'istanza di `StrayCat`, un componente chiave del framework Cheshire Cat, che funge da punto di accesso principale a tutte le sue funzionalità. Questo componente gestisce le sessioni utente, gestisce la memoria di lavoro, facilita le conversazioni e fornisce metodi essenziali per l'interazione LLM, la messaggistica WebSocket e altro ancora.

`cat` è progettato per essere l'interfaccia principale per sfruttare appieno le potenzialità del framework. Utilizzando questo oggetto, è possibile accedere a varie funzionalità senza doverle importare. Fornisce inoltre comode scorciatoie per accedere a strutture annidate o correlate all'utente all'interno di Cat, semplificando il processo di sviluppo.

## Shortcut
Ad esempio, è possibile utilizzare facilmente LLM:
```python
@hook
def agent_fast_reply(reply, cat):
    prompt = "Say a joke in german"
    reply["output"] = cat.llm(prompt)
    return reply
```
Un esempio che utilizza lo scheduler White Rabbit :
````python
@tool(return_direct=True)
def schedule_say_hello(minutes, cat):
    """Say hello in a few minutes. Input is the number of minutes."""
    delay = int(minutes)

    # We can access the White Rabbit to schedule jobs
    job_id = cat.white_rabbit.schedule_chat_message("Hello", cat, minutes=delay)

    return f"Scheduled job {job_id} to say hello in {delay} minutes."
````

Allo stesso modo, puoi accedere ad altri componenti del Gatto, come il `Mad Hatter` o `Rabbit Hole`.
Puoi anche accedere alla Memoria di Lavoro . Facilmente, in questo modo:
````python
@tool(return_direct=True)
def clear_working_memory(arg, cat):
    """Use this tool to clear / reset / delete the conversation history."""

    # We can access the Working Memory to clear the chat history
    cat.working_memory.history = []

    return "Chat history cleared"
````


## Metodi
Lo `StrayCat` espone alcuni metodi, alcuni sono particolarmente utili per lo sviluppo di plugin:
- `cat.send_chat_message(...)`: Invia un messaggio all'utente utilizzando la connessione WebSocket attiva.
- `cat.send_notification(...)`: Invia una notifica all'utente utilizzando la connessione WebSocket attiva.
- `cat.send_error(...)`: Invia un messaggio di errore all'utente utilizzando la connessione WebSocket attiva.
- `cat.llm(...)`: Metodo di scelta rapida per generare una risposta utilizzando LLM e passando un prompt personalizzato.
- `cat.embedder.embed_query(...)`: Metodo di scelta rapida per incorporare una stringa nello spazio vettoriale.
- `cat.classify(...)`: Metodo di utilità per classificare una frase data utilizzando l'LLM. È possibile passare un elenco di stringhe come possibili etichette oppure un dizionario di etichette come chiavi e un elenco di stringhe come esempi.
- `cat.stringify_chat_history(...)`: Metodo di utilità per trasformare in stringa la cronologia della chat.

Documentazione
- [Metodi utili e scorciatoie](https://cheshire-cat-ai.github.io/docs/framework/cat-components/cheshire_cat/stray_cat/#useful-methods-and-shortcuts)
- [API StrayCat](https://cheshire-cat-ai.github.io/docs/API_Documentation/looking_glass/stray_cat/)

For more details, see the StrayCat API reference