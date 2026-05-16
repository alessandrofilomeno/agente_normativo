# 05 — Roadmap

## Stato attuale

L'agente normativo è funzionante con copertura nazionale. Il workflow n8n monitora 10 fonti ufficiali e aggiorna il database Notion con novità normative.

## Prossimi step — Agente normativo

### Popolamento regionale
Da completare in sessioni successive, una macro-area per volta:

- [ ] Nord Ovest — Piemonte, Valle d'Aosta, Liguria, Lombardia
- [ ] Nord Est — Veneto, Friuli-VG, Emilia-Romagna, PA Trento, PA Bolzano
- [ ] Centro — Toscana, Umbria, Marche, Lazio
- [ ] Sud — Abruzzo, Molise, Campania, Puglia, Basilicata, Calabria
- [ ] Isole — Sicilia, Sardegna

Per ogni regione vanno aggiunte:
1. Le normative regionali pertinenti nel database "Normativa Ambientale"
2. Le fonti da monitorare (BUR regionale, portali regionali) nel database "Fonti da monitorare"
3. I collegamenti relation tra fonti e normative

### Automazione invio mail
Configurare Gmail OAuth in n8n per l'invio automatico del report al termine di ogni esecuzione.

**Prerequisiti:** Progetto Google Cloud con credenziali OAuth2.

### Scheduler automatico
Sostituire il trigger manuale con uno Schedule Trigger in n8n per esecuzione automatica mensile.

**Prerequisito:** PC acceso o migrazione a VPS (5-10€/mese su Hetzner/DigitalOcean).

---

## Agente redattore (fase 2)

Agente in grado di compilare in modo semiautomatico le sezioni normative delle relazioni tecniche.

### Flusso previsto

```
Input: regione + tipo opera + categorie rilevanti
      ↓
Query database Notion (normative pertinenti filtrate)
      ↓
[Opzionale] RAG su PDF normative per citazioni puntuali
      ↓
Bozza paragrafo normativo
      ↓
Revisione e approvazione utente
```

### Tecnologie necessarie

**Per il livello base (solo database Notion):**
- Connettore Notion già configurato
- Prompt strutturato per la generazione del testo
- Nessuna tecnologia aggiuntiva

**Per il livello avanzato (RAG su PDF):**
- Vector store (Chroma in locale — gratuito, oppure Pinecone)
- Pipeline di indicizzazione PDF
- Ricerca semantica sui paragrafi rilevanti

### Priorità

Il livello base con solo il database Notion è sufficiente per la maggior parte dei casi d'uso. Il RAG va introdotto solo quando serve citare articoli specifici con precisione.

---

## Agenti futuri (fase 3+)

### Agente VIncA
Supporto alla compilazione degli studi di incidenza — struttura dello studio, check-list specie, riferimenti normativi per sito specifico.

### Agente modulistica
Monitoraggio e compilazione automatica dei format regionali per le istanze di trasformazione bosco e autorizzazione paesaggistica.

### Orchestratore
Quando gli agenti sono più di due, valutare un orchestratore che li coordini in base al tipo di incarico ricevuto.

---

## Note tecniche future

### Migrazione a VPS
Se si vuole esecuzione automatica senza tenere acceso il PC:
- Hetzner CX11: ~4€/mese (2GB RAM, sufficiente per n8n)
- Setup: Docker + n8n identico al locale
- Accesso remoto tramite browser

### Token Notion permanente
Valutare upgrade piano Notion Plus (10$/mese) per eliminare la rotazione manuale del token ogni 7 giorni.

### Monitoring
In futuro aggiungere notifica in caso di errori nell'esecuzione del workflow (email o Telegram).
