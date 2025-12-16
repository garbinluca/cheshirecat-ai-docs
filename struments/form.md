# Form

I form sono strumenti particolari utili per raccogliere informazioni sugli utenti durante una conversazione.

Un form consente di definire una struttura dati specifica, che il framework cercherà di attivare e completare 
automaticamente in un dialogo multi-turn. 

La differenza tra `@tool` e `@form`è che il tool è one-shot, mentre il form permette più turni conversazionali.

È possibile definire:
- `validators`: Verificano che i dati siano corretti (ad esempio, i numeri di telefono possono variare a seconda del Paese, la pizzeria ha un menù specifico di pizze e la consegna è limitata a determinate zone della città).
- triggers
- fields
- submission callback
- how the Cat expresses missing or invalid fields

[Documentazione completa](https://cheshire-cat-ai.github.io/docs/plugins/forms/)

Esempio caso reale:

```python
import requests
from pydantic import BaseModel
from cat.experimental.form import CatForm, CatFormState, form

class PizzaOrder(BaseModel): #
    pizza_type: str
    phone: str
    address: str


@form #
class PizzaForm(CatForm): #
    description = "Pizza Order" #
    model_class = PizzaOrder #
    start_examples = [ #
        "order a pizza!",
        "I want pizza"
    ]
    stop_examples = [ #
        "stop pizza order",
        "not hungry anymore",
    ]
    ask_confirm = True #

    def submit(self, form_data): #

        # Fake API call to order the pizza
        response = requests.post(
            "https://fakecallpizza/order",
            json={
                "pizza_type": form_data["pizza_type"],
                "phone": form_data["phone"],
                "address": form_data["address"]
            }
        )
        response.raise_for_status()

        time = response.json()["estimated_time"]

        # Return a message to the conversation with the order details and estimated time
        return {
            "output": f"Pizza order on its way: {form_data}. Estimated time: {time}"
        }
```

## Stati del form

Ogni FSM ha una funzione di transizione di stato che descrive quale sarà l'azione successiva da eseguire in 
base all'input fornito. Nel caso dell'implementazione del form del Gatto, l'input è il prompt dell'utente e il metodo 
`def next(self)` funge da funzione di transizione di stato.

Il modulo valuta quattro stati:
- INCOMPLETO
- ATTENDE_CONFERMA
- CHIUSO
- COMPLETARE

Ogni stato esegue una o più fasi:
- Fase di interruzione del modulo utente
- Fase di conferma dell'utente
- Fase di aggiornamento
- Fase di visualizzazione
- Fase di invio

È possibile modificare questa transizione di stato sovrascrivendo il metodo `def next(self)` e accedendo allo stato tramite 
`self._state`. Gli stati sono valori dell'enum `CatFormState`.

## Interazione con il form

### Fase di interruzione del modulo utente 
_Concetto chiave: interruzione del modulo_

La fase di interruzione del modulo da parte dell'utente è quella in cui il modulo verifica se l'utente desidera uscire dal modulo. 
È possibile modificare questa fase sovrascrivendo il `def check_exit_intent(self)`.

### Fase di conferma dell'utente 
_Concetto chiave: Conferma dell'utente_

La fase di conferma dell'utente è quella in cui il modulo chiede all'utente di confermare le informazioni fornite, se `ask_confirm` impostata su `true`. 
È possibile modificare questa fase sovrascrivendo il metodo `def confirm(self)`.

### Fase di aggiornamento 
_Concetto chiave: estrazione, sanificazione, convalida delle informazioni_

La fase di aggiornamento è quella in cui il modulo esegue la fase di estrazione, la fase di sanificazione e la fase di convalida. 
È possibile modificare questa fase sovrascrivendo il metodo `def update(self)`.

#### Fase di estrazione
La fase di estrazione è quella in cui il modulo estrae tutte le informazioni possibili dal prompt dell'utente. È possibile modificare questa fase sovrascrivendo il La fase di estrazione è quella in cui il modulo estrae tutte le informazioni possibili dal prompt dell'utente. È possibile modificare questa fase sovrascrivendo il metodo `def extract(self)`.

#### Fase di sanificazione
La fase di sanificazione è quella in cui le informazioni vengono sanificate per rimuovere i valori indesiderati (null, None, '', ' ', ecc.). 
È possibile modificare questa fase sovrascrivendo il metodo `def sanitize(self, model)`.

#### Fase di convalida
La fase di convalida è quella in cui il modulo tenta di costruire il modello, consentendo a Pydantic di utilizzare i validatori implementati e controllare ogni campo. È possibile modificare questa fase sovrascrivendo il metodo `def validate(self, model)`.

### Fase di visualizzazione
La fase di visualizzazione è quella in cui il modulo mostra all'utente lo stato del modello visualizzando un messaggio.

Come definire il messaggio di risposta in chat:
```python
# In the form you define 
def message(self): # 
    if self._state == CatFormState.CLOSED: #
        return {
            "output": f"Form {type(self).__name__} closed"
        }
    missing_fields: List[str] = self._missing_fields #
    errors: List[str] = self._errors #
    out: str = f"""
    The missing information is: {missing_fields}.
    These are the invalid ones: {errors}
    """
    if self._state == CatFormState.WAIT_CONFIRM:
        out += "\n --> Confirm? Yes or no?"

    return {
        "output": out
    }
```
### Fase finale: invio

La fase di invio è quella in cui il modulo conclude il processo eseguendo tutte le istruzioni definite con le informazioni raccolte dalla conversazione dell'utente. Il metodo ha due parametri:
- `self` : fornisce accesso alle informazioni sul modulo e StrayCatsull'istanza.
- `form_data` : il modello Pydantic definito formattato come un dizionario Python.

Il metodo deve restituire un dizionario in cui il valore della outputchiave è una stringa che verrà visualizzata nella chat.

