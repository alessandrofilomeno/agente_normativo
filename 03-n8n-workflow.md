# 03 — Workflow n8n

## Installazione n8n

### Prerequisiti
- Docker Desktop installato su Windows
- WSL 2 abilitato (richiesto da Docker su Windows)

### Comando di avvio (versione definitiva)

```bash
docker run -d --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n --restart unless-stopped docker.n8n.io/n8nio/n8n
```

**Flag importanti:**
- `-d` — esecuzione in background, non occupa il terminale
- `--restart unless-stopped` — riavvio automatico con Docker Desktop
- `-v n8n_data:/home/node/.n8n` — dati persistenti nel volume Docker

### Comandi utili

```bash
docker stop n8n       # Ferma n8n
docker start n8n      # Riavvia n8n
docker ps             # Verifica se n8n è in esecuzione
```

### Accesso
Apri Chrome e vai su: **http://localhost:5678**

## Struttura del workflow

### Nodo 1 — Trigger manuale
Tipo: Manual Trigger
Avvia il workflow manualmente. In futuro sostituibile con Schedule Trigger per esecuzione automatica.

### Nodo 2 — Notion Get Many Database Pages
Tipo: Notion → Get Many Database Pages
Legge tutte le fonti dal database "Fonti da monitorare".

**Configurazione:**
- Database: By ID → `1635ce35-1421-4c31-bada-edcde40f755b`
- Return All: attivo

**Output rilevanti:**
- `property_nome_fonte`
- `property_url`
- `property_nota_ricerca`
- `property_categoria_tematica`
- `property_ambito_territoriale`
- `property_norma_collegata`

### Nodo 3 — Loop Over Items
Tipo: Loop Over Items
Processa una fonte alla volta. Ha due uscite: `loop` (per ogni item) e `done` (fine loop).

### Nodo 4 — HTTP Request
Tipo: HTTP Request
Scarica il contenuto HTML della pagina fonte.

**Configurazione:**
- Method: GET
- URL: `{{ $json.property_url }}`
- Response Format: Text

### Nodo 5 — Anthropic Message a Model
Tipo: Anthropic → Text Actions → Message a Model
Analizza il contenuto della pagina cercando novità normative.

**Configurazione:**
- Model: `claude-sonnet-4-5`
- Prompt: vedi `/prompts/analisi-normativa.md`

### Nodo 6 — Code (parsing JSON)
Tipo: Code (JavaScript)
Estrae i dati strutturati dalla risposta di Anthropic e aggiunge informazioni sulla fonte.

```javascript
const items = [];

for (const item of $input.all()) {
  let text = '';
  if (item.json.text) {
    text = item.json.text;
  } else if (item.json.message && item.json.message.content) {
    text = item.json.message.content[0].text || '';
  } else {
    text = JSON.stringify(item.json);
  }
  
  const clean = text.replace(/```json|```/g, '').trim();
  const match = clean.match(/\{[\s\S]*\}/);
  const jsonStr = match ? match[0] : clean;
  
  let parsed = {};
  try {
    parsed = JSON.parse(jsonStr);
  } catch(e) {}
  
  const fonte = $('Loop Over Items').item.json;
  const ambito = fonte.property_ambito_territoriale || 'Nazionale';
  
  const areeMap = {
    'Piemonte': 'Nord Ovest', 'Valle d Aosta': 'Nord Ovest',
    'Liguria': 'Nord Ovest', 'Lombardia': 'Nord Ovest',
    'Veneto': 'Nord Est', 'Friuli-Venezia Giulia': 'Nord Est',
    'Emilia-Romagna': 'Nord Est', 'PA Trento': 'Nord Est', 'PA Bolzano': 'Nord Est',
    'Toscana': 'Centro', 'Umbria': 'Centro', 'Marche': 'Centro', 'Lazio': 'Centro',
    'Abruzzo': 'Sud', 'Molise': 'Sud', 'Campania': 'Sud',
    'Puglia': 'Sud', 'Basilicata': 'Sud', 'Calabria': 'Sud',
    'Sicilia': 'Isole', 'Sardegna': 'Isole'
  };
  const area = areeMap[ambito] || 'Nazionale';

  items.push({
    json: {
      novita: parsed.novita !== undefined ? parsed.novita : false,
      titolo_novita: parsed.titolo_novita || null,
      descrizione: parsed.descrizione || null,
      azione: parsed.azione || 'NESSUNA_AZIONE',
      nome_fonte: fonte.property_nome_fonte,
      url_fonte: fonte.property_url,
      norma_collegata: fonte.property_norma_collegata,
      categoria: fonte.property_categoria_tematica,
      ambito_territoriale: ambito,
      area_geografica: area
    }
  });
}

return items;
```

### Nodo 7 — Switch
Tipo: Switch (Mode: Rules)
Instrada il flusso in base all'azione da eseguire.

**Regole:**
- `$json.azione` equals `AGGIORNA_NORMA_ESISTENTE` → ramo "Aggiorna"
- `$json.azione` equals `AGGIUNGI_NUOVA_NORMA` → ramo "Nuova"
- `$json.azione` equals `NESSUNA_AZIONE` → ramo "Nessuna"

### Ramo NESSUNA_AZIONE

**Nodo 8a — Notion Update (Ultima verifica fonte)**
Tipo: Notion → Update a Database Page
Aggiorna solo la data di controllo sulla fonte.

- Database Page: By ID → `{{ $('Loop Over Items').item.json.id }}`
- Proprietà: Ultima verifica → `{{ $now.format('yyyy-MM-dd') }}`

### Ramo AGGIORNA_NORMA_ESISTENTE

**Nodo 8b — Notion Update (norma)**
Tipo: Notion → Update a Database Page
Aggiorna la norma esistente in Normativa Ambientale.

- Database Page: By ID → `{{ $json.norma_collegata[0] }}`
- Proprietà:
  - Note aggiornamento → `{{ $json.descrizione || '' }}`
  - Status → Modificata
  - Novità rilevata → true
  - Data ultimo controllo → `{{ $now.format('yyyy-MM-dd') }}`

**Nodo 9b — Notion Update (Ultima verifica fonte)**
Identico al nodo 8a.

### Ramo AGGIUNGI_NUOVA_NORMA

**Nodo 8c — Notion Create Database Page**
Tipo: Notion → Create a Database Page
Crea una nuova voce in Normativa Ambientale.

- Database: By ID → `1a09bb94-1575-43fd-b534-0f7bd4ad2924`
- Title: `{{ $json.titolo_novita || 'Nuova norma da verificare' }}`
- Proprietà:
  - Note aggiornamento → `{{ $json.descrizione || '' }}`
  - Status → Vigente
  - Area geografica → `{{ $json.area_geografica }}`
  - Ambito territoriale → `{{ $json.ambito_territoriale }}`
  - Novità rilevata → true
  - Data ultimo controllo → `{{ $now.format('yyyy-MM-dd') }}`

**Nodo 9c — Notion Update (Ultima verifica fonte)**
Identico al nodo 8a.

### Nodo finale — Code (composizione mail)
Tipo: Code (JavaScript)
Collegato all'uscita **done** del Loop Over Items.

```javascript
const today = new Date().toLocaleDateString('it-IT', { 
  day: '2-digit', month: 'long', year: 'numeric' 
});

const allItems = $input.all();
const novita = allItems.filter(item => item.json.novita === true);
const totale = allItems.length;
const invariate = totale - novita.length;

let corpo = `AGENTE NORMATIVO — Monitoraggio del ${today}\n`;
corpo += '═'.repeat(55) + '\n\n';
corpo += `Fonti analizzate: ${totale}\n`;
corpo += `Invariate: ${invariate} | Novità: ${novita.length}\n\n`;

if (novita.length > 0) {
  corpo += '⚠ NOVITÀ RILEVATE\n';
  corpo += '─'.repeat(40) + '\n\n';
  novita.forEach((item, i) => {
    corpo += `${i + 1}. ${item.json.nome_fonte}\n`;
    corpo += `   Azione: ${item.json.azione}\n`;
    corpo += `   ${item.json.descrizione || ''}\n`;
    corpo += `   Fonte: ${item.json.url_fonte}\n\n`;
  });
} else {
  corpo += '✓ Nessuna novità rilevata in questo ciclo.\n\n';
}

corpo += '─'.repeat(40) + '\n';
corpo += `Database: https://www.notion.so/1a09bb94157543fdb5340f7bd4ad2924`;

return [{
  json: {
    subject: `Agente Normativo — ${novita.length > 0 ? '⚠ ' + novita.length + ' novità rilevate' : '✓ Nessuna novità'} — ${today}`,
    body: corpo,
    novita_count: novita.length
  }
}];
```

## Credenziali configurate

| Credenziale | Tipo | Utilizzo |
|---|---|---|
| Notion account | OAuth (connettore n8n) | Nodi Notion nativi |
| Notion Header Auth | Header Auth (Generic) | HTTP Request verso API Notion |

## Note operative

- Il workflow va lanciato manualmente aprendo http://localhost:5678
- Docker Desktop deve essere avviato (balena verde nella taskbar)
- Il token Notion va rinnovato ogni 7 giorni
- Il report mail viene prodotto ma non inviato automaticamente (Gmail OAuth da configurare)
