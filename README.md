<h1 align="center">Influencer Outreach Automation</h1>
<h3 align="center">KI-gestützte Influencer-Recherche & -Akquise mit n8n</h3>

<p align="center">
  Vollautomatisierte Pipeline von der Datenerfassung bis zur personalisierten,<br/>
  KI-generierten Ansprache — mit Human-in-the-Loop-Freigabe über Telegram.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/n8n-EA4B71?style=for-the-badge&logo=n8n&logoColor=white" alt="n8n" />
  <img src="https://img.shields.io/badge/Groq_AI-F55036?style=for-the-badge&logo=groq&logoColor=white" alt="Groq" />
  <img src="https://img.shields.io/badge/Telegram-26A5E4?style=for-the-badge&logo=telegram&logoColor=white" alt="Telegram" />
  <img src="https://img.shields.io/badge/Google_Sheets-34A853?style=for-the-badge&logo=googlesheets&logoColor=white" alt="Google Sheets" />
  <img src="https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white" alt="Docker" />
</p>

---

## Überblick

Dieses System automatisiert den gesamten Prozess der Influencer-Akquise — von der Datenerfassung bis zur personalisierten Ansprache. Manuelle Recherche und Erstkontakt werden durch eine dreiteilige n8n-Pipeline ersetzt, bei der die KI die Anschreiben generiert und ein Mensch sie per Telegram final freigibt.

> 🖼️ **Workflow-Canvas:** `assets/workflow-canvas.png`
> <!-- Screenshot des n8n-Editors hier einbetten: ![Workflow](assets/workflow-canvas.png) -->

**Tech-Stack:** n8n · Groq (LLM-Texterstellung) · Telegram Bot API (HITL) · Google Sheets API · ManyChat (Instagram DM, optional) · Modash API (optional) · Docker

---

## Systemarchitektur

Das System besteht aus 3 Ketten:

### Kette 1 — Modash Pipeline (vollautomatisch)
```
Modash API → Webhook → Google Sheets → Edit Fields → Groq AI → Telegram
```
Wird aktiviert, sobald Influencer-Daten von Modash eintreffen.

### Kette 2 — Schedule Pipeline (manuelle Einträge)
```
Schedule Trigger (alle 5 Min) → Google Sheets → Filter → Edit Fields → Groq AI → Telegram
```
Wird aktiviert, wenn das Team manuell Influencer in Google Sheets einträgt.

### Kette 3 — Telegram Approval (Human-in-the-Loop)
```
Telegram Trigger → Code → IF (Genehmigen/Ablehnen) → Telegram Bestätigung
```
Wird aktiviert, wenn das Team auf den Genehmigen/Ablehnen-Button drückt.

---

## Voraussetzungen

### Benötigte API Keys

| Service | Wo besorgen | Wozu |
|---------|-------------|------|
| Groq API | console.groq.com | KI-Texterstellung (kostenlos) |
| Telegram Bot Token | @BotFather auf Telegram | Benachrichtigungen + Genehmigung |
| ManyChat API Key | manychat.com/settings/api | Instagram DM Versand (optional) |
| Modash API Key | modash.io | Automatische Influencer-Daten (optional) |

### Benötigte Software
- n8n (Self-hosted via Docker oder n8n.cloud)
- Google Account (für Google Sheets)

---

## Installation

### Schritt 1 — n8n starten
```bash
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n
```

### Schritt 2 — Workflow importieren
1. n8n öffnen: `http://localhost:5678`
2. Neues Workflow erstellen
3. Oben rechts auf `...` klicken → `Import from File`
4. `influencer-automation.json` hochladen

### Schritt 3 — Credentials einrichten

**Google Sheets:** n8n → Credentials → New → Google Sheets OAuth2 → Google-Account verbinden → in allen Google-Sheets-Nodes auswählen.

**Groq API:** n8n → Credentials → New → Header Auth → Name: `Authorization`, Value: `Bearer <DEIN_GROQ_KEY>`.

**Telegram Bot:** Telegram → @BotFather → `/newbot` → Token kopieren → n8n → Credentials → New → Telegram API → Token eintragen.

### Schritt 4 — Google Sheets verknüpfen
1. Google Sheet öffnen (eigene Kopie der Vorlage)
2. Spreadsheet-ID aus der URL kopieren
3. In allen Google-Sheets-Nodes eintragen

### Schritt 5 — ManyChat einrichten (optional)
Für automatischen Instagram-DM-Versand: manychat.com → Account → Instagram-Business-Account verbinden → Settings → API → API Key → im HTTP-Request-Node (ManyChat) als `Authorization: Bearer <DEIN_MANYCHAT_KEY>` eintragen.

---

## Verwendung

### Manueller Influencer-Eintrag
1. Google Sheet → Tab `Influencer_Datenbank`
2. Neue Zeile ausfüllen (Pflichtfelder: Handle, Stadt, Kategorie, Follower, Inhalte, Engagement)
3. `Anschreiben_Status` auf `Ausstehend` setzen
4. Innerhalb von 5 Minuten erscheint das Anschreiben in Telegram

### Automatischer Modash Import
```bash
curl -X POST https://<DEINE_N8N_URL>/webhook/modash-influencer \
  -H "Content-Type: application/json" \
  -d '{
    "handle": "@influencer_name",
    "followers": 50000,
    "engagement_rate": 4.5,
    "city": "Berlin",
    "bundesland": "Berlin",
    "categories": ["food", "lifestyle"],
    "bio": "Food und Lifestyle in Berlin",
    "website": "www.example.de"
  }'
```

### Genehmigungsprozess
1. Telegram-Nachricht mit Anschreiben empfangen
2. `Genehmigen` oder `Ablehnen` drücken
3. Bei Genehmigung: Nachricht über den konfigurierten Kanal senden

---

## Google Sheets Struktur

### Tab: Influencer_Datenbank
| Spalte | Typ | Beschreibung |
|--------|-----|--------------|
| Instagram_Handle | Pflicht | @username |
| Stadt_Bundesland | Pflicht | z. B. Berlin |
| Kategorie | Pflicht | food, lifestyle, travel |
| Follower_Anzahl | Pflicht | Anzahl der Follower |
| Top_Inhalte | Pflicht | Meistgesehene Inhalte |
| Engagement_Rate | Pflicht | z. B. 4.5 % |
| Kontakt_Email | Optional | E-Mail-Adresse |
| Website | Optional | Bio-Link |
| Anschreiben_Status | Auto | Ausstehend / Erledigt |
| Notizen | Optional | Interne Notizen |
| Erfassung_Datum | Auto | Datum der Erfassung |

### Tab: Generierte_Anschreiben
Alle von der KI erstellten Anschreiben mit Datum.

### Tab: Workflow_Dokumentation
Vollständige Architekturbeschreibung des Systems.

---

## Produktionsumgebung

1. **n8n Cloud** oder Server mit HTTPS-URL (Telegram-Trigger + Webhooks benötigen öffentlich erreichbares HTTPS)
2. **ngrok** für lokale Entwicklung:
   ```bash
   ngrok http 5678
   ```
   Die generierte HTTPS-URL in n8n unter Settings → Webhook URL eintragen.
3. **ManyChat** (Meta-zertifizierter Partner) für automatischen Instagram-DM-Versand ohne Ban-Risiko.

---

## Skalierbarkeit

| Influencer | Verarbeitungszeit |
|------------|------------------|
| 10 | ~1 Minute |
| 100 | ~10 Minuten |
| 1.000 | ~2 Stunden |
| 10.000+ | Modash API + parallele Verarbeitung |

---

## Highlights

- **End-to-End-Automatisierung:** von der Datenerfassung bis zur versandfertigen Ansprache
- **Human-in-the-Loop:** jede KI-Nachricht wird vor dem Versand per Telegram freigegeben
- **Zwei Eingangswege:** vollautomatisch (Modash-Webhook) und manuell (Google Sheets)
- **Kostenbewusst:** Groq als kostenlose LLM-Schicht, ManyChat/Modash optional zuschaltbar
