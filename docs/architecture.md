# Architektur / Architecture

```mermaid
flowchart LR
  K[Kunde · Sprachnachricht] --> STT[whisper.cpp lokal]
  STT --> GR[Guardrails · Jailbreak-Prüfung]
  GR --> CL[Klassifikator · Gemma strukturiert]
  CL --> SW{Intent}
  SW --> ST[Bestand: Postgres]
  SW --> FAQ[FAQ: Qdrant + bge-m3<br/>Schwelle 0,55 · Abstand 0,05]
  SW --> ORD[Bestell-Assistent · 8 Schritte]
  SW --> ESC[Beschwerde → Eskalation]:::human
  ST & FAQ --> ANS[Antwort aus Fakten · Code]
  ANS --> PII[Filter personenbezogener Daten]
  PII --> TTS[Piper lokal]
  TTS -- Ausfall --> FB[Ersatzweg / Text + Alarm]
  ANS --> LOG[(Protokoll ohne PII)]
  classDef human stroke-dasharray: 5 5
```

| Workflow | Aufgabe / Purpose | Trigger |
|---|---|---|
| `P14_VoiceDesk_Full-local` | Sprachdialog, Bestand, FAQ, Bestellung / voice dialogue | Telegram |
| `P14_Ingestion_FAQ-to-Qdrant` | FAQ-PDF → Qdrant | manuell / Formular |
| `P14_Watchdog` | prüft alle 10 min STT, Modell, Suche, TTS / health check | Zeitplan, Error |
