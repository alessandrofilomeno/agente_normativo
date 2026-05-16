# Agente Normativo

Sistema di monitoraggio automatico della normativa ambientale italiana pertinente alle attività di consulenza forestale e ambientale per infrastrutture a rete.

## Contesto

Il progetto nasce dall'esigenza di un dottore forestale abilitato che svolge attività di consulenza connessa alla realizzazione di infrastrutture a rete (prevalentemente elettriche) su tutto il territorio nazionale. L'attività richiede un costante aggiornamento normativo su:

- Trasformazione del bosco
- Valutazione di Incidenza (VIncA)
- Paesaggio e autorizzazioni paesaggistiche
- Vincolo idrogeologico
- Specie protette e liste rosse
- Verde urbano
- Documenti operativi (schede Natura 2000, PMA)
- Procedure e modulistica regionali

## Architettura del sistema

```
Fonti da monitorare (Notion)
      ↓
n8n — Loop Over Items
      ↓
HTTP Request — scarica contenuto pagina web
      ↓
Claude (Anthropic API) — analizza aggiornamenti
      ↓
Switch
  ├── NESSUNA_AZIONE → aggiorna data controllo fonte
  ├── AGGIORNA_NORMA_ESISTENTE → aggiorna norma in Normativa Ambientale
  └── AGGIUNGI_NUOVA_NORMA → crea nuova voce in Normativa Ambientale
      ↓ (fine loop)
Code — compone report mail
```

## Database Notion

Due database interconnessi:

### Normativa Ambientale
Archivio di tutte le normative pertinenti, organizzato per ambito territoriale e categoria tematica. Viste filtrate per area geografica (Nazionale, Nord Ovest, Nord Est, Centro, Sud, Isole).

**Proprietà:** Nome norma, Descrizione pertinente, Categoria tematica, Ambito territoriale, Area geografica, Data emanazione, Data ultimo controllo, Status, Novità rilevata, Link fonte ufficiale, Note aggiornamento.

### Fonti da monitorare
Elenco delle fonti ufficiali controllate ad ogni ciclo. Ogni fonte è collegata tramite relation alla norma corrispondente in Normativa Ambientale.

**Proprietà:** Nome fonte, URL, Categoria tematica, Ambito territoriale, Tipo fonte, Frequenza controllo, Ultima verifica, Note ricerca, Norma collegata.

## Tecnologie utilizzate

| Strumento | Utilizzo |
|---|---|
| Claude (Anthropic) | Analisi normativa e rilevamento novità |
| n8n (Docker) | Orchestrazione workflow |
| Notion | Database normativo e memoria dell'agente |
| HTTP Request | Fetch contenuto pagine web |

## Struttura repository

```
agente-normativo/
├── README.md
├── docs/
│   ├── 01-architettura.md
│   ├── 02-notion-setup.md
│   ├── 03-n8n-workflow.md
│   ├── 04-problematiche.md
│   └── 05-roadmap.md
├── workflows/
│   └── agente-normativo.json
└── prompts/
    └── analisi-normativa.md
```

## Stato del progetto

- [x] Database Notion strutturato
- [x] Popolamento normative nazionali (16 voci)
- [x] Database fonti da monitorare (10 fonti nazionali)
- [x] Workflow n8n funzionante
- [ ] Popolamento normative regionali (Nord Ovest)
- [ ] Popolamento normative regionali (Nord Est)
- [ ] Popolamento normative regionali (Centro)
- [ ] Popolamento normative regionali (Sud)
- [ ] Popolamento normative regionali (Isole)
- [ ] Automazione invio mail (Gmail OAuth)
- [ ] Agente redattore relazioni tecniche
