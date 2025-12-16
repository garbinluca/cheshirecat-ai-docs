# Integration

- [Container API Docs](http://localhost:1865/docs).

## Websockets
```javascript
let ws = new WebSocket("ws://localhost:1865/ws")

let humanTurn = function(){
    let msg = prompt("Human message:")
    if(msg) {
        ws.send(JSON.stringify({"text": msg}))
    }
}

ws.onopen = function(){
    humanTurn()
}

ws.onmessage = function(msg){
    let msg_parsed = JSON.parse(msg.data)
    console.log(msg_parsed.content)
    if(msg_parsed.type == "chat") {
        humanTurn()   
    }
}
```

## Authentication
For protect the API calls, set those `.env` settings:
```
CCAT_API_KEY=a-very-long-and-alphanumeric-secret
CCAT_API_KEY_WS=another-very-long-and-alphanumeric-secret
CCAT_JWT_SECRET=yet-another-very-long-and-alphanumeric-secret
```

Authenticated calls:

```javascript
# REST api
let response = await fetch(
    "http://localhost:1865",
    {
        "headers": {
            "Authorization": "Bearer meow",
            "user_id": "Caterpillar" // optional
        }
    }
)
let json = await response.json()
console.log(response.status, json)

# WS 
let ws = new WebSocket("ws://localhost:1865/ws")

ws.onopen = function() {
    ws.send(JSON.stringify({"text": "It's late"}))
}

ws.onmessage = function(msg){
    console.log(JSON.parse(msg.data))
}
```