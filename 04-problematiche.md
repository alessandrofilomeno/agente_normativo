# 04 — Problematiche emerse e soluzioni

## 1. CORS — Chiamate API bloccate dal browser

**Problema:** L'artifact interattivo costruito in Claude non riusciva a chiamare direttamente l'API Notion. Il browser blocca le chiamate cross-origin per motivi di sicurezza.

**Errore:** `NetworkError when attempting to fetch resource`

**Soluzione:** Abbandonare l'approccio artifact e usare n8n come orchestratore. n8n gira lato server (nel container Docker) dove il CORS non è un problema.

**Lezione:** Gli artifact Claude non possono fare chiamate API dirette a servizi esterni. Per automazioni che richiedono chiamate API serve un orchestratore server-side.

---

## 2. Isolamento accesso Google Drive

**Problema:** Il connettore Google Drive di Claude aveva accesso all'intero Drive personale dell'utente, senza possibilità di isolamento.

**Tentativo fallito:** Creare un Google Drive condiviso — non disponibile con account Gmail gratuito (richiede Google Workspace).

**Soluzione adottata:** Eliminare Google Drive dall'architettura. Notion è sufficiente come unica memoria dell'agente — i PDF delle normative sono documenti pubblici accessibili tramite link, non serve archiviarli localmente.

**Lezione:** Semplificare l'architettura elimina problemi di sicurezza e riduce la complessità.

---

## 3. Token Notion — Scadenza ogni 7 giorni

**Problema:** Il Personal Access Token di Notion con piano gratuito scade ogni 7 giorni. Non esiste opzione per token permanente senza piano a pagamento.

**Soluzione:** Procedura manuale di rinnovo: notion.so/profile/integrations → Regenerate → aggiornare in n8n Credentials.

**Workaround futuro:** Valutare upgrade piano Notion o configurare reminder calendario per il rinnovo.

---

## 4. Nodo Notion Update — Proprietà non si caricano con ID dinamico

**Problema:** Il nodo "Update a Database Page" di n8n non carica le proprietà disponibili quando il Page ID è un'espressione dinamica (`{{ $json.id }}`). Mostra solo "Key Name or ID" senza opzioni.

**Causa:** Bug noto in n8n — le proprietà vengono caricate tramite chiamata API al momento della configurazione, ma falliscono con ID dinamici.

**Soluzione:** Inserire temporaneamente un Page ID statico per far caricare le proprietà, configurarle, poi sostituire con l'espressione dinamica.

---

## 5. Token API Notion non funziona per chiamate dirette

**Problema:** Il token `ntn_...` (Personal Access Token) funziona solo con il connettore MCP di Claude, non con chiamate HTTP dirette all'API Notion.

**Errore:** `API token is invalid` con status 401.

**Tentativo fallito:** Creare un'integration su notion.so/my-integrations — la nuova interfaccia Notion non espone più questa opzione separatamente.

**Soluzione:** Usare il nodo Notion nativo di n8n (che usa OAuth) invece di HTTP Request per le operazioni di scrittura su Notion. L'HTTP Request con autenticazione Header Auth funziona con le credenziali OAuth di n8n.

---

## 6. Anthropic non visita le URL

**Problema:** Il nodo Anthropic analizzava le normative basandosi sulla propria conoscenza interna, non sul contenuto reale delle pagine web. Restituiva sempre `novita: false` perché non aveva accesso ai contenuti aggiornati.

**Soluzione:** Aggiungere un nodo HTTP Request prima di Anthropic che scarica il contenuto HTML della pagina. Il prompt passa i primi 8.000 caratteri del contenuto come contesto.

**Lezione:** Per monitoraggio web affidabile, l'AI deve ricevere il contenuto reale della pagina come input, non fare affidamento sulla propria memoria.

---

## 7. Parsing JSON dalla risposta Anthropic

**Problema:** Anthropic restituisce il JSON avvolto in backtick markdown (` ```json ... ``` `). Il parsing diretto falliva con `Unexpected end of JSON input`.

**Soluzione nel nodo Code:**
```javascript
const clean = text.replace(/```json|```/g, '').trim();
const match = clean.match(/\{[\s\S]*\}/);
const parsed = JSON.parse(match ? match[0] : clean);
```

---

## 8. Nomi campi Notion in n8n

**Problema:** I nomi delle proprietà Notion in n8n seguono una convenzione diversa da quella attesa. Le proprietà vengono prefissate con `property_` e convertite in snake_case.

**Esempi:**
- "Link fonte ufficiale" → `property_link_fonte_ufficiale`
- "Nome norma" → `property_nome_norma`
- "URL" (campo userDefined) → `property_url`

**Soluzione:** Leggere sempre l'output effettivo del nodo Notion prima di scrivere le espressioni nei nodi successivi.

---

## 9. Database vs Pagina come parent in Notion

**Problema:** Tentativo di creare un database dentro un database — restituisce errore `Can't create databases parented by a database`.

**Soluzione:** Creare sempre una pagina normale come contenitore, poi creare il database dentro quella pagina.

---

## 10. Switch — Valore con spazio invece di underscore

**Problema:** La regola del nodo Switch era configurata con `NESSUNA AZIONE` (con spazio) mentre il codice restituiva `NESSUNA_AZIONE` (con underscore). Il ramo non veniva mai raggiunto.

**Soluzione:** Verificare sempre la corrispondenza esatta tra i valori nel nodo Code e le regole nel nodo Switch.
