# ZAK AI – Setup & Automatisierungs-Anleitung

## 📋 Inhaltsverzeichnis

1. [Schnellstart](#schnellstart)
2. [Firebase einrichten](#firebase-einrichten)
3. [Newsletter-Automatisierung](#newsletter-automatisierung)
4. [Kundendatenbank](#kundendatenbank)
5. [Admin-Bereich](#admin-bereich)
6. [Automatisierungen mit Make.com](#automatisierungen)
7. [Wartung & Updates](#wartung)

---

## 🚀 Schnellstart

### Dateien hochladen

1. Erstelle ein neues GitHub Repository: `final-zak-ai`
2. Lade alle Dateien aus diesem Ordner hoch
3. Aktiviere GitHub Pages: Settings → Pages → Source: main
4. Deine Seite ist live unter: `https://[username].github.io/final-zak-ai/`

### Dateistruktur

```
final-zak-ai/
├── index.html              # Hauptseite mit Login & Newsletter
├── karriere.html           # Karriere-Seite
├── pages/
│   ├── impressum.html      # Impressum (Jan Hundsdorff)
│   ├── ueber-uns.html      # Über uns Seite
│   └── datenschutz.html    # Datenschutzerklärung
├── demo/
│   ├── admin/
│   │   ├── login.html      # Admin-Login
│   │   └── dashboard.html  # Admin-Dashboard
│   ├── podologie.html      # Öffentliche Demo
│   ├── shop.html           # Login-geschützt
│   ├── baecker.html        # Login-geschützt
│   ├── restaurant.html     # Login-geschützt
│   ├── immobilien.html     # Login-geschützt
│   └── fitness.html        # Login-geschützt
└── SETUP_ANLEITUNG.md      # Diese Datei
```

---

## 🔥 Firebase einrichten

### 1. Firebase Projekt erstellen

1. Gehe zu [console.firebase.google.com](https://console.firebase.google.com)
2. Klicke "Projekt hinzufügen"
3. Name: `zak-ai-production`
4. Google Analytics: Optional (kann deaktiviert werden)

### 2. Authentication aktivieren

1. Im Firebase Dashboard: **Build → Authentication**
2. Klicke "Erste Schritte"
3. Aktiviere folgende Anbieter:
   - **E-Mail/Passwort**: Einfach aktivieren
   - **Google**:
     - Aktivieren
     - Support-E-Mail: deine E-Mail eintragen
   - **Apple** (optional):
     - Aktivieren
     - Apple Developer Account erforderlich
     - Services ID und Team ID eintragen

### 3. Firestore Datenbank erstellen

1. **Build → Firestore Database**
2. Klicke "Datenbank erstellen"
3. Wähle "Produktionsmodus starten"
4. Region: `europe-west3` (Frankfurt)

### 4. Sicherheitsregeln konfigurieren

Gehe zu Firestore → Regeln und füge ein:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Benutzer können nur ihr eigenes Dokument lesen/schreiben
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }

    // Newsletter: Jeder kann sich eintragen, nur Admin kann lesen
    match /newsletter/{docId} {
      allow create: if true;
      allow read: if request.auth != null;
    }

    // Kontaktanfragen
    match /contacts/{docId} {
      allow create: if true;
      allow read: if request.auth != null;
    }
  }
}
```

### 5. Firebase Config in index.html eintragen

1. Gehe zu Projekteinstellungen (Zahnrad)
2. Scrolle zu "Ihre Apps"
3. Klicke auf Web-Icon `</>`
4. App-Name: `ZAK AI Web`
5. Kopiere die Config-Werte

In `index.html` ersetze:

```javascript
const firebaseConfig = {
    apiKey: "DEIN_API_KEY",
    authDomain: "DEIN_PROJEKT.firebaseapp.com",
    projectId: "DEIN_PROJECT_ID",
    storageBucket: "DEIN_PROJEKT.appspot.com",
    messagingSenderId: "DEINE_SENDER_ID",
    appId: "DEINE_APP_ID"
};
```

---

## 📧 Newsletter-Automatisierung

### Option A: Mit Make.com (empfohlen)

1. Erstelle einen Account auf [make.com](https://make.com)
2. Neues Szenario erstellen

**Trigger: Firebase Webhook**
- Wenn neuer Newsletter-Eintrag in Firestore

**Aktion 1: E-Mail senden (Bestätigung)**
- An: {{email}}
- Betreff: "Willkommen bei ZAK AI Newsletter"
- Inhalt: Begrüßungstext

**Aktion 2: Google Sheets (Optional)**
- Neue Zeile mit E-Mail und Datum hinzufügen

### Option B: Mit Firebase Cloud Functions

```javascript
// functions/index.js
const functions = require('firebase-functions');
const nodemailer = require('nodemailer');

exports.onNewSubscriber = functions.firestore
  .document('newsletter/{docId}')
  .onCreate(async (snap, context) => {
    const data = snap.data();
    // E-Mail senden...
  });
```

---

## 👥 Kundendatenbank

### Datenstruktur in Firestore

**Collection: `users`**
```json
{
  "uid": "auto-generiert",
  "email": "kunde@email.de",
  "displayName": "Max Mustermann",
  "company": "Firma GmbH",
  "phone": "+49 123 456789",
  "createdAt": "timestamp",
  "plan": "free | basic | standard | premium",
  "lastLogin": "timestamp"
}
```

**Collection: `newsletter`**
```json
{
  "email": "subscriber@email.de",
  "subscribedAt": "timestamp",
  "source": "website"
}
```

### Daten exportieren

Im Firebase Console:
1. Firestore → Collection auswählen
2. Drei-Punkte-Menü → Daten exportieren
3. Oder mit CLI: `firebase firestore:export gs://bucket-name`

---

## 🔐 Admin-Bereich

### Login-Daten

| Feld | Wert |
|------|------|
| URL | `/demo/admin/login.html` |
| Benutzername | `admin` |
| Passwort | `zakai2026` |

### Passwort ändern

In `demo/admin/login.html` Zeile ~50:

```javascript
const validUsername = 'admin';
const validPassword = 'NEUES_PASSWORT';
```

### Admin-Funktionen

- Alle Demos einsehen (auch gesperrte)
- Statistiken anzeigen
- Direktlinks zu allen Seiten

---

## ⚡ Automatisierungen mit Make.com

### Szenario 1: Neuer Benutzer → Willkommens-E-Mail

```
Trigger: Firebase - Neues Dokument in /users
↓
Filter: Nur wenn createdAt = heute
↓
Aktion: E-Mail senden
  - An: {{email}}
  - Betreff: "Willkommen bei ZAK AI, {{displayName}}!"
  - Template: Willkommens-E-Mail
↓
Aktion: Slack-Nachricht (optional)
  - Channel: #neue-kunden
  - Text: "Neuer Kunde: {{displayName}} ({{email}})"
```

### Szenario 2: Newsletter → Google Sheets

```
Trigger: Firebase Webhook
↓
Aktion: Google Sheets - Zeile hinzufügen
  - Tabelle: "Newsletter Abonnenten"
  - Spalten: E-Mail, Datum, Quelle
```

### Szenario 3: Kontaktanfrage → CRM + E-Mail

```
Trigger: Firebase - Neues Dokument in /contacts
↓
Aktion: E-Mail an Admin
  - Betreff: "Neue Anfrage von {{name}}"
↓
Aktion: Notion/Airtable - Neuer Eintrag
```

---

## 🔧 Wartung & Updates

### Regelmäßige Aufgaben

**Wöchentlich:**
- Newsletter-Liste prüfen
- Neue Registrierungen checken
- Kapazitätsanzeige aktualisieren

**Monatlich:**
- Firebase-Kosten prüfen
- Backups der Firestore-Daten
- Sicherheitsregeln überprüfen

### Kapazität anpassen

In `index.html` suche nach:

```html
<span>Belegt: 17/20</span>
```

Und ändere die Zahlen entsprechend.

### Preise ändern

In `index.html` im Pricing-Bereich:

```html
<span class="text-3xl font-bold">79€</span>
```

---

## 📞 Support

Bei Fragen:
- E-Mail: kontakt@zak-ai.de
- Telefon: Auf Anfrage

---

## 📝 Changelog

### Version 2.0 (Januar 2026)
- Firebase Authentication (Google, Apple, E-Mail)
- Newsletter-System
- Kundendatenbank
- Limitierte Kapazitätsanzeige
- Demo-Zugang nur nach Login (außer Podologie)
- Impressum, Datenschutz, Über uns Seiten
- Admin-Dashboard

### Version 1.0 (Januar 2026)
- Erste Version
- 6 Demo-Chatbots
- Basic Admin-Login

---

© 2026 ZAK AI – Jan Hundsdorff, Winterlingen
