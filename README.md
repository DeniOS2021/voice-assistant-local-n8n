# Sprachassistent – vollständig lokal mit n8n

**[DE](#deutsch) · [EN](#english)** · Fallstudie auf [denpilot.de](https://denpilot.de/#projekt-voicedesk)

> Bereinigte Kopien: ohne Zugangsdaten, IDs und Kontaktdaten. Fallstudien mit fiktiven Unternehmen, keine Kundendaten.

## Deutsch

### Aufgabe
Ein Handelsunternehmen bekommt viele gleichartige Fragen zu Lagerbestand und Lieferung. Kund:innen sollen per Sprachnachricht fragen, eine gesprochene Antwort aus echten Lagerdaten erhalten und bestellen können – ohne dass Preise oder Bestände vom Modell „erfunden“ werden.

### Architektur
3 n8n-Workflows. Spracherkennung (whisper.cpp large-v3-turbo), Sprachmodell (Gemma in LM Studio), Vektorsuche (Qdrant, bge-m3), Datenbank (Postgres) und Sprachausgabe (Piper) laufen lokal; n8n orchestriert. Diagramm: [`docs/architecture.md`](docs/architecture.md).

### Wer macht was
| KI | Code | Mensch |
|---|---|---|
| Anliegen und Produkt erkennen, Antwort sprachlich formulieren | Preise, Bestände, Relevanz-Schwelle (0,55 / Abstand 0,05), Bestell-Assistent in 8 Schritten | Übernahme nach zwei erfolglosen Versuchen und bei Beschwerden |

### Zuverlässigkeit
- Abbruch nach 2 Fehlversuchen – Übergabe statt Endlosschleife.
- Watchdog prüft alle 10 min Erkennung, Modell, Suche und Sprachausgabe.
- Fällt die lokale Sprachausgabe aus, gibt es einen Ersatzweg; fallen beide aus: Text an Kunde, Alarm ans Team.
- Fangfragen-Set gegen sichere Antworten mit fremden Zahlen.
- Ein zweites Prüfmodell im Live-Dialog wurde bewusst verworfen (Latenz, korrelierte Fehler); stattdessen 5 Prüfstufen im Code.

### DSGVO / KI-Verordnung
Stimme und personenbezogene Daten verlassen die eigene Infrastruktur nicht – keine Übermittlung an KI-Anbieter in Drittländern (Art. 44 ff. DSGVO). Audio wird nicht gespeichert, personenbezogene Daten werden vor dem Protokoll entfernt. KI-Hinweis nach Art. 50 KI-Verordnung per Code.

### Demo starten
1. n8n (self-hosted, aktuelle 1.x/2.x) starten.
2. **Neuen, leeren** Workflow anlegen → Menü **⋯ → Import from File** → JSON aus `workflows/` wählen.
   Wichtig: Import in einen bereits gefüllten Workflow fügt Knoten hinzu, statt ihn zu ersetzen.
3. Zugangsdaten (Credentials) in n8n anlegen und an den markierten Knoten auswählen – im JSON sind sie bewusst leer.
4. Platzhalter ersetzen (siehe `.env.example`): `YOUR_LOCAL_HOST`, `YOUR_CHAT_ID`, `YOUR_SHEET_ID` usw.
5. Erst testen, dann aktivieren. Alle Workflows sind im Export **inaktiv**.
6. Lokale Dienste: LM Studio (1234), Qdrant (6333), Sprachdienst mit `/stt`, `/tts`, `/health` (8088), Postgres mit Tabellen für Katalog, Bestellungen und Protokoll.
7. `P14_Ingestion_FAQ-to-Qdrant` einmal ausführen (eigene FAQ als PDF).

### Grenzen
- Der Sprachdienst (HTTP-Wrapper um whisper.cpp und Piper) und das Datenbankschema sind nicht enthalten.
- Kein Lasttest; Messwerte zur Antwortzeit sind nicht veröffentlicht.
- Ersatzwege über Cloud-Dienste (ElevenLabs, Whisper-API) sind im Export deaktiviert.

---

## English

### Task
A retail company receives many repetitive questions about stock and delivery. Customers should ask by voice message, get a spoken answer from real inventory data and be able to order – without the model “inventing” prices or stock.

### Architecture
3 n8n workflows. Speech-to-text (whisper.cpp large-v3-turbo), language model (Gemma in LM Studio), vector search (Qdrant, bge-m3), database (Postgres) and text-to-speech (Piper) run locally; n8n orchestrates. Diagram: [`docs/architecture.md`](docs/architecture.md).

### Who does what
| AI | Code | Human |
|---|---|---|
| Detect intent and product, phrase the spoken answer | Prices, stock, relevance threshold (0.55 / gap 0.05), 8-step order assistant | Takes over after two failed attempts and for complaints |

### Reliability
- Stop after 2 failed attempts – hand-over instead of an endless loop.
- Watchdog checks speech recognition, model, search and speech output every 10 min.
- If local speech output fails there is a fallback; if both fail: text to the customer, alert to the team.
- Trick-question set against confident answers with foreign numbers.
- A second judge model in the live dialogue was deliberately rejected (latency, correlated errors); 5 code-based checks instead.

### GDPR / AI Act
Voice and personal data never leave own infrastructure – no transfer to AI providers in third countries (Art. 44 et seq. GDPR). Audio is not stored; personal data is removed before logging. AI disclosure under Art. 50 AI Act by code.

### Run the demo
1. Start n8n (self-hosted, current 1.x/2.x).
2. Create a **new, empty** workflow → menu **⋯ → Import from File** → pick a JSON from `workflows/`.
   Note: importing into a non-empty workflow adds nodes instead of replacing them.
3. Create the credentials in n8n and select them on the marked nodes – they are deliberately empty in the JSON.
4. Replace the placeholders (see `.env.example`): `YOUR_LOCAL_HOST`, `YOUR_CHAT_ID`, `YOUR_SHEET_ID`, etc.
5. Test first, then activate. All workflows are exported **inactive**.
6. Local services: LM Studio (1234), Qdrant (6333), voice service with `/stt`, `/tts`, `/health` (8088), Postgres with tables for catalogue, orders and log.
7. Run `P14_Ingestion_FAQ-to-Qdrant` once (your own FAQ as PDF).

### Limitations
- The voice service (HTTP wrapper around whisper.cpp and Piper) and the database schema are not included.
- No load test; response-time measurements are not published.
- Cloud fallbacks (ElevenLabs, Whisper API) are disabled in the export.

---
Cleaned copies: no credentials, IDs or contact details. Case studies with fictitious companies, no customer data. · Licence: MIT · [LinkedIn](https://www.linkedin.com/in/denys-kopyl-ai-automation/)
