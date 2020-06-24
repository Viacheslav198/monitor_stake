# Monitoraggio del palo

Per richiedere lo stato con un validatore, utilizzare Near-Shell o il metodo validatorsin RPC JSON:

| Action | near-shell | validators JSON RPC |
| ------ | ---------- | -------- |
| current set (t0) | `near validators current` | `result.current_validators` |
| next set (t+1) | `near validators next` | `result.next_validators` |
| proposals (t+2) | `near proposals` | `result.current_proposals` |

Dove, l' `t0` era attuale e l' `t+n` era del futuro.

## Monitoraggio degli attuali validatori con RPC

Questo comando richiede che RPC JSON emetta il numero di token:

```
curl -d '{"jsonrpc": "2.0", "method": "validators", "id": "dontcare", "params": [null]}' -H 'Content-Type: application/json' https://rpc.betanet.near.org | jq -c '.result.current_validators[] | select(.account_id | contains ("<POOL_ID>"))' | jq .stake
```

`"method": "validators"` - metodo

`jq -c '.result.current_validators` - visualizza i validatori correnti

`select(.account_id | contains ("<POOL_ID>"))'` - filtraggio per <POOL_ID>

`jq .stake` - filtra nuovamente i risultati tramite jq e accetta solo la condivisione totale in YoctoNEAR

Rispetto a Near-Shell, questo metodo produce un numero più accurato di token Near nel pool. <POOL_ID>

È possibile utilizzare un filtro simile per verificare se ci sarà un pool nei seguenti validatori o meno.


## Monitoraggio degli attuali validatori con near-shell

Visualizza il numero di token vicini in numeri interi:

```
near validators current | awk '/<POOL_ID>/ {print $4}'
```

`<POOL_ID> `- piscina per picchetti

`near validators current` - visualizza i validatori correnti

`awk '/<POOL_ID> {print $4}'` - filtra per POOL_ID e stampa un numero intero con la bistecca corrente


## Monitoraggio dei futuri validatori tramite RPC

Simile ai comandi sopra, usa il comando:

```
curl -d '{"jsonrpc": "2.0", "method": "validators", "id": "dontcare", "params": [null]}' -H 'Content-Type: application/json' https://rpc.betanet.near.org | jq -c '.result.next_validators[] | select(.account_id | contains ("<POOL_ID>"))'
```

Se l'output non è vuoto, <POOL_ID> avrà lo stato di Rollover e ne salverà il posto come validatore.
Verranno visualizzate le seguenti informazioni: nome del pool, public_key, dimensione della bistecca e numero del frammento

RPC fornisce dati di un'era precedente per esaminare i dati ricevuti e comprendere il motivo per cui il nodo non si trova negli attuali validatori:

```
curl -d '{"jsonrpc": "2.0", "method": "validators", "id": "dontcare", "params": [null]}' -H 'Content-Type: application/json' https://rpc.betanet.near.org | jq -c '.result.prev_epoch_kickout[] | select(.account_id | contains ("<POOL_ID>"))' | jq .reason
```

La perdita minima consentita per blocco non supera il 10% del previsto!

Simile ad altri comandi sopra:

 `jq -c '.result.prev_epoch_kickout` - filtro kickout precedente
 
 `jq .reason` - filtra il motivo, ad esempio un numero insufficiente di token per una bistecca o un numero insufficiente di blocchi generati


## Monitoraggio dei futuri validatori con near-shell

Per vedere se un nodo perderà il suo posto nella prossima era:

```
near validators next | grep "Kicked out" | grep "<POOL_ID>"
```

Se l'output non è vuoto, il posto andrà perso.

In alternativa, è possibile utilizzare il comando:

```
near proposals | grep "Rollover" | grep "<POOL_ID>"
```

Se l'output non è vuoto, <POOL_ID> avrà lo stato di Rollover e salverà il suo posto come validatore


## Monitoraggio del progresso di un'era

Determina l'altezza del blocco corrente:

```
curl https://rpc.betanet.near.org/status | jq .sync_info.latest_block_height
```

Ad oggi, l'inizio dell'era epoch_startpuò essere ottenuto da JSON RPC:

```
curl -d '{"jsonrpc": "2.0", "method": "validators", "id": "dontcare", "params": [null]}' -H 'Content-Type: application / json' https : //rpc.betanet.near.org | jq .result.epoch_start_height
```

Questa query genererà un numero intero con il numero di blocco da cui è iniziata l'era corrente

Per capire quanti blocchi restano da realizzare entro la fine del periodo, è necessario sottrarre `latest_block_height` dalla `epoch_start_height + 10000` e ottenere il numero di blocchi che devono ancora essere prodotte al fine di completare l'epoca.

Il numero di blocchi nell'era per le reti Near:

| Network | Epoch Blocks |
| ------- | ------ |
| BetaNet | 10,000 |
| TestNet | 43,200 |
| MainNet | 43,200 |

## Posiziona il monitoraggio dei prezzi

Per misurare o calcolare il costo di un luogo per diventare un validatore, puoi usare, ad esempio, Near-Shell:

prezzo attuale nell'era: 
```
near validators current | awk '/price/ {print substr($6, 1, length($6)-2)}'
```
prezzo di un posto nella prossima era: 
```
near validators next | awk '/price/ {print substr($7, 1, length($7)-2)}'
```
Prezzo stimato del luogo t + 2: 
```
near proposals | awk '/price =/ {print substr($15, 1, length($15)-1)}'
```
