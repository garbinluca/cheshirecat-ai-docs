# CheshireCat AI Framework

**⚠️⚠️ ️️This is not the official documentation ⚠️️⚠️**

Una versione light e personale della documentazione del framework Cheshire Cat AI.

## Dictionary
- `Episodic memory`: Memoria che contiene tutte le conversazioni precedenti. Quando viene fatta una domanda, il Gatto risponde prendendo le conversazioni all'interno l'account.
- `Declarative memory`: memoria popolata tramite l'upload di documenti. Viene gestita dal Rabbit Hole
- `Rabbit hole`: componente responsabile dell'integrazione dei documenti
- `White Rabbit`: componente che gestisce la schedulazione di attività. Le attività possono essere one-time, interval-based o cron.
- `Mad Hatter`: componente che gestisce ed esegue i plugins
- `Hook`: funzioni python appese al flow del processo principale: il codice verrà invocato durante l'esecuzione del processo e può modificare il comportamento interno del Gatto, senza modificarne il core.
- `Tool`: funzioni python chiamate dall'LLM per eseguire azioni. Sono divise in due parti: la prima contiene le istruzioni da dare all'LLM quando e come chiamare la funzione; la seconda contiene il codice da eseguire.
- `Plugins`: collections of Tools and Hooks that can be installed simply by placing files in a designated folder or by using the Admin Portal

## Resources
- [This plugin](https://github.com/pieroit/meow-todo-list/blob/main/meow_todo_list.py) to learn how to do a todo list.
- [UniBO spiegazione](https://amslaurea.unibo.it/id/eprint/33476/1/CHESHIRE_CAT_AI_FRAMEWORK_PER_LA_REALIZZAZIONE_DI_CHATBOT_SPECIALIZZATI.pdf)


## Altri contenuti
- [Integration](docs/integration.md)
- [Tools](docs/tools.md)
- [Hooks](docs/hooks.md)
- [Form](docs/form.md)
- [Logging](docs/logging.md)
- [Settings](docs/settings.md)

## Altro
- https://cheshire-cat-ai.github.io/docs/framework/cat-components/cheshire_cat/stray_cat/