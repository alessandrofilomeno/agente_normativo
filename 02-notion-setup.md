# 02 — Setup Notion

## Prerequisiti

- Account Notion (piano gratuito sufficiente)
- Integration "Agente normativo" creata su notion.so/profile/integrations
- Token di tipo Personal Access Token (scade ogni 7 giorni con piano gratuito)

> **Nota:** Il token permanente richiede piano Notion a pagamento. Con il piano gratuito è necessario rigenerare il token ogni 7 giorni e aggiornarlo nelle credenziali n8n.

## Struttura pagine Notion

```
Agente normativo (pagina padre)
├── Normativa Ambientale (database)
└── Fonti da monitorare (database)
```

## Database 1 — Normativa Ambientale

**ID:** `1a09bb94-1575-43fd-b534-0f7bd4ad2924`
**Data source ID:** `620092c3-bd62-4376-a18b-b37190e59219`

### Schema proprietà

| Proprietà | Tipo | Note |
|---|---|---|
| Nome norma | Title | |
| Descrizione pertinente | Rich text | Rilevanza per il lavoro specifico |
| Categoria tematica | Multi-select | Bosco, VIncA, Paesaggio, Vincolo idrogeologico, Specie protette, Procedure e modulistica, Verde urbano, Documenti operativi |
| Ambito territoriale | Select | Nazionale + tutte le regioni + PA Trento/Bolzano |
| Area geografica | Select | Nazionale, Nord Ovest, Nord Est, Centro, Sud, Isole |
| Data emanazione | Date | |
| Data ultimo controllo | Date | Aggiornata ad ogni ciclo |
| Status | Select | Vigente, Modificata, Abrogata |
| Novità rilevata | Checkbox | Spuntato quando agente trova aggiornamento |
| Link fonte ufficiale | URL | |
| Note aggiornamento | Rich text | Descrizione novità nell'ultimo ciclo |

### Viste configurate

| Vista | Filtro | Ordinamento |
|---|---|---|
| 🇮🇹 Nazionale | Area geografica = Nazionale | Categoria tematica ASC |
| 🟢 Nord Ovest | Area geografica = Nord Ovest | Ambito territoriale ASC |
| 🔵 Nord Est | Area geografica = Nord Est | Ambito territoriale ASC |
| 🟡 Centro | Area geografica = Centro | Ambito territoriale ASC |
| 🟠 Sud | Area geografica = Sud | Ambito territoriale ASC |
| 🏝️ Isole | Area geografica = Isole | Ambito territoriale ASC |

### Normative nazionali inserite (16 voci)

**Bosco**
- D.lgs 34/2018 (TUFF) — Testo Unico Foreste
- D.M. 7 ottobre 2020 — Esonero interventi compensativi
- D.M. 5 aprile 2023 n. 193945 — Rete boschi vetusti

**Vincolo idrogeologico**
- R.D. 30 dicembre 1923 n. 3267

**VIncA**
- DPR 8 settembre 1997 n. 357 (mod. DPR 120/2003)
- Linee Guida Nazionali VIncA — Intesa Stato-Regioni 28/11/2019
- Direttiva 92/43/CEE — Direttiva Habitat
- Direttiva 2009/147/CE — Direttiva Uccelli
- D.lgs 3 aprile 2006 n. 152 — Codice Ambiente

**Paesaggio**
- D.lgs 22 gennaio 2004 n. 42 — Codice Urbani ⚠️ *Modificato L. 40/2026*
- DPR 13 febbraio 2017 n. 31 — Autorizzazione paesaggistica semplificata ⚠️ *DDL 1372 in iter*

**Specie protette**
- Lista Rossa Flora Italiana — IUCN
- Lista Rossa Vertebrati Italiani — IUCN (2022)

**Verde urbano**
- Quadro normativo nazionale (L. 10/2013, DM 2/3/2023, Codice Civile artt. 892-894)

**Documenti operativi**
- Schede Natura 2000 — Trasmissione annuale MASE ⚠️ *Ultima trasmissione dicembre 2025*
- Piani di Monitoraggio Ambientale (PMA) — D.lgs 152/2006

## Database 2 — Fonti da monitorare

**ID:** `1635ce35-1421-4c31-bada-edcde40f755b`
**Data source ID:** `9c55cd25-2b53-4cf2-a978-e062da2cb065`

### Schema proprietà

| Proprietà | Tipo | Note |
|---|---|---|
| Nome fonte | Title | |
| URL | URL | |
| Categoria tematica | Multi-select | Stesse opzioni di Normativa Ambientale |
| Ambito territoriale | Select | |
| Tipo fonte | Select | Portale ministeriale, GU, BUR regionale, Ente scientifico, EUR-Lex, Normattiva |
| Frequenza controllo | Select | Ad ogni ciclo, Mensile, Annuale |
| Ultima verifica | Date | Aggiornata ad ogni esecuzione |
| Note ricerca | Rich text | Istruzioni specifiche per l'agente |
| Norma collegata | Relation | Collegamento a Normativa Ambientale |

### Fonti nazionali inserite (10 voci)

| Fonte | URL | Frequenza |
|---|---|---|
| Normattiva — TUFF | normattiva.it | Ad ogni ciclo |
| Normattiva — Codice Urbani | normattiva.it | Ad ogni ciclo |
| Normattiva — DPR 357/97 | normattiva.it | Ad ogni ciclo |
| Normattiva — D.lgs 152/2006 | normattiva.it | Ad ogni ciclo |
| MASE — Portale VIncA | mase.gov.it | Ad ogni ciclo |
| MASE — Schede Natura 2000 | mase.gov.it | Annuale |
| Gazzetta Ufficiale | gazzettaufficiale.it | Ad ogni ciclo |
| EUR-Lex — Direttive Habitat/Uccelli | eur-lex.europa.eu | Mensile |
| IUCN Italia — Liste Rosse | iucn.it | Annuale |
| MASE — Portale VIA-VAS | va.mite.gov.it | Ad ogni ciclo |

## Gestione token

Il Personal Access Token di Notion scade ogni 7 giorni con il piano gratuito.

**Procedura di rinnovo:**
1. Vai su notion.so/profile/integrations
2. Apri "Agente normativo"
3. Clicca "Regenerate token"
4. Aggiorna il token in n8n → Credentials → "Notion account"

> **Attenzione:** Non incollare mai il token in chat o in documenti condivisi.
