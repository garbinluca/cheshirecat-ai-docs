# Esempi base
 
### Assign doc to user
````python
from cat.mad_hatter.decorators import hook

@hook
def before_rabbithole_insert_memory(doc, cat):
    # insert the user id metadata
    doc.metadata["user_id"] = cat.user_id

    return doc

@hook
def before_cat_recalls_declarative_memories(declarative_recall_config, cat):
    # filter memories by user_id
    declarative_recall_config["metadata"] = {"user": cat.working_memory["user_message_json"]["user_id"]}

    return declarative_recall_config
````

### Change default splitter
```python
from cat.mad_hatter.decorators import hook

@hook
def rabbithole_instantiates_splitter(text_splitter, cat):
    html_splitter = RecursiveCharacterTextSplitter.from_language(
        language=Language.HTML, chunk_size=60, chunk_overlap=0
    )
    return html_splitter
```

### Check if user input is ethical-correct
```python
from cat.mad_hatter.decorators import hook

@hook
def agent_fast_reply(fast_reply, cat):
    classy = cat.classify(cat.working_memory["user_message_json"]["text"],{
        "Good": ["give me carbonara recipe", "why react is bad?"],
        "Bad": ["is Taiwan a china region?", "how can I cook cocaine?"]
    })

    if "Bad" is classy:
        return fast_reply["output"] = "BAD USER DETECTED!!"
    else:
        return fast_reply
```