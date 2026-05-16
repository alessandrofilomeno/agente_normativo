# 01 — Architettura e logica del progetto

## Obiettivo

Costruire un sistema di agenti AI che automatizzi processi ripetitivi nell'attività di consulenza forestale e ambientale. Il primo agente — l'agente normativo — monitora gli aggiornamenti della normativa ambientale nazionale e regionale pertinente.

## Decisioni architetturali

### Perché Sonnet e non Opus o Haiku

Sonnet è il punto di equilibrio ottimale per questo use case:
- Abbastanza capace per ragionamento su testi legali e normativi con sfumature
- Costo significativamente inferiore a Opus
- Haiku sarebbe insufficiente per interpretare normative complesse

### Perché Notion come memoria

Notion offre:
- API ufficiali stabili con integrazione diretta in n8n
- Isolamento dei permessi per integration (l'agente vede solo le pagine condivise esplicitamente)
- Viste filtrate per navigazione rapida
- Possibilità di collegamento relation tra database

### Perché n8n su Docker

- Orchestrazione visuale dei workflow senza codice
- Esecuzione in locale sul PC dell'utente
- Persistenza dei dati tramite volume Docker (`n8n_data`)
- Avvio automatico con Docker Desktop (`--restart unless-stopped`)
- Costo zero

### Perché HTTP Request + Anthropic invece di agent con web search

- Costo controllato: fetch diretto delle fonti note invece di ricerca libera
- Risultati più affidabili: fonti ufficiali predefinite
- Nessun servizio terzo (SerpAPI, Tavily, ecc.)
- L'approccio "opzione B" — scraping strutturato di fonti specifiche — è più preciso dell'opzione A (ricerca libera)

## Perimetro normativo monitorato

### Categorie tematiche

| Categoria | Descrizione |
|---|---|
| Bosco | D.lgs 34/2018 (TUFF), leggi regionali trasformazione bosco |
| VIncA | DPR 357/97, Linee Guida 2019, Direttive Habitat e Uccelli |
| Paesaggio | D.lgs 42/2004 (Codice Urbani), DPR 31/2017 |
| Vincolo idrogeologico | RD 3267/1923, leggi regionali attuative |
| Specie protette | Liste Rosse IUCN, allegati Direttive Habitat/Uccelli |
| Verde urbano | L. 10/2013, DM 2/3/2023, Codice Civile artt. 892-894 |
| Documenti operativi | Schede Natura 2000, Piani di Monitoraggio Ambientale |
| Procedure e modulistica | Format istanze regionali, portali, modulistica |

### Copertura geografica

- Livello nazionale
- 20 regioni + Province Autonome di Trento e Bolzano

## Flusso di esecuzione

### Prima esecuzione (popolamento)
1. Inserimento manuale normative nazionali nel database Notion
2. Inserimento fonti da monitorare con relation alle normative
3. Esecuzione workflow per verifica funzionamento

### Ciclo di monitoraggio (esecuzione periodica)
1. n8n legge tutte le fonti dal database "Fonti da monitorare"
2. Per ogni fonte: HTTP Request scarica il contenuto della pagina
3. Claude analizza il contenuto cercando novità negli ultimi 60 giorni
4. In base alla risposta (JSON strutturato) il workflow:
   - Aggiorna la norma esistente se modificata
   - Crea una nuova voce se trovata norma non censita
   - Aggiorna solo la data di controllo se nessuna novità
5. Il nodo finale compone il report testuale pronto per mail

## Costi stimati

| Scenario | Token per esecuzione | Costo stimato |
|---|---|---|
| Solo nazionali (16 norme) | ~70.000 | ~0,20$/esecuzione |
| Nazionale + tutte le regioni (~200 norme) | ~900.000 | ~2,50$/esecuzione |
| Mensile a regime completo | — | ~5$/mese |

Costi calcolati con claude-sonnet-4-5 (maggio 2026).

## Evoluzione futura

### Agente redattore (prossima fase)
Agente in grado di compilare la sezione normativa delle relazioni tecniche attingendo dal database Notion. Utilizzerà RAG (Retrieval Augmented Generation) su PDF delle normative per citazioni puntuali.

### Automazione calendario
Integrazione con n8n scheduler per esecuzione automatica mensile senza intervento manuale.
