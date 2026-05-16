# Prompt — Analisi normativa

## Prompt utilizzato nel nodo Anthropic (Message a Model)

```
Sei un esperto di diritto ambientale italiano specializzato in normativa forestale, VIncA, paesaggio, vincolo idrogeologico e specie protette.

Fonte monitorata: {{ $('Loop Over Items').item.json.property_nome_fonte }}
URL: {{ $('Loop Over Items').item.json.property_url }}
Categoria: {{ $('Loop Over Items').item.json.property_categoria_tematica }}
Note ricerca: {{ $('Loop Over Items').item.json.property_note_ricerca }}
Data odierna: {{ $now.toISO() }}

Contenuto pagina web:
{{ $json.data.substring(0, 8000) }}

Analizza il contenuto HTML della pagina cercando aggiornamenti normativi recenti (ultimi 60 giorni) pertinenti alle categorie indicate.
Cerca date recenti, nuovi provvedimenti, modifiche legislative, nuove circolari o documenti.

Rispondi SOLO in JSON:
{"novita": true/false, "titolo_novita": "titolo breve max 80 caratteri o null", "descrizione": "descrizione max 200 caratteri o null", "azione": "AGGIORNA_NORMA_ESISTENTE o AGGIUNGI_NUOVA_NORMA o NESSUNA_AZIONE"}
```

## Logica delle azioni

| Azione | Quando | Effetto in n8n |
|---|---|---|
| `NESSUNA_AZIONE` | Nessuna novità rilevata | Aggiorna solo data controllo sulla fonte |
| `AGGIORNA_NORMA_ESISTENTE` | Norma esistente modificata/aggiornata | Aggiorna Status, Note, Novità rilevata sulla norma collegata |
| `AGGIUNGI_NUOVA_NORMA` | Trovata normativa non ancora in database | Crea nuova voce in Normativa Ambientale |

## Note sul prompt

- Il contenuto HTML viene troncato a 8.000 caratteri per contenere i costi token
- Il modello usato è `claude-sonnet-4-5` (ottimale per costo/qualità)
- Il JSON di risposta viene poi parsato dal nodo Code successivo
- Il parsing gestisce i backtick markdown che Anthropic aggiunge attorno al JSON

## Formato mail di output

```
AGENTE NORMATIVO — Monitoraggio del [data]
═══════════════════════════════════════════════════════

Fonti analizzate: [N]
Invariate: [N] | Novità: [N]

⚠ NOVITÀ RILEVATE
────────────────────────────────────────

1. [Nome fonte]
   Azione: [AZIONE]
   [Descrizione novità]
   Fonte: [URL]

────────────────────────────────────────
Database: https://www.notion.so/1a09bb94157543fdb5340f7bd4ad2924
```
