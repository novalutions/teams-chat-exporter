# Teams Chat Exporter - Technische Dokumentation

## Überblick

Der Teams Chat Exporter ist eine Power Platform-Lösung, die den Export von Microsoft Teams-Chats und Kanal-Konversationen ermöglicht. Die Lösung besteht aus einer Power App und drei Power Automate-Workflows. Exportierte Inhalte werden als interaktive HTML-Dateien im OneDrive des Benutzers gespeichert und bieten Such-, Filter- und Navigationsfunktionen. Nach dem Export bestehen keine Cloud-Verbindungen oder Abhängigkeiten zur bestehenden Microsoft-365-Installation.

## Architektur

Die Lösung setzt sich aus folgenden Komponenten zusammen:

- **1 Power App** – Benutzeroberfläche zur Chat-Auswahl und Exportsteuerung, eingebettet in Microsoft Teams
- **3 Power Automate-Workflows:**
  - **GetChats** – Lädt die Chat-Liste des Benutzers mit Paginierung über die Microsoft Graph API
  - **Chat-Export** – Exportiert einen ausgewählten 1:1-, Gruppen- oder Meeting-Chat
  - **Kanal-Export** – Exportiert einen oder mehrere Konversations-Threads aus einem Teams-Kanal

> **Hinweis:** Die Workflows, die direkt aus dem Chat aufgerufen werden, sind seit dem 01.05.2025 veraltet und werden nicht weiter unterstützt. Der Export erfolgt ausschließlich über die Power App.

## Voraussetzungen

Für die Installation des Teams Chat Exporters sind folgende Voraussetzungen erforderlich:

- **Administrative Berechtigungen:** System Administrator, System Customizer oder Environment Admin
- **Lizenz:** Mindestens eine Microsoft 365 Business Basic Lizenz (Teams Essentials wird nicht unterstützt)
- **Umgebung:** Microsoft Power Platform-Umgebung im Tenant

Ohne diese Rollen ist der Export nicht möglich. Mit dem Kauf bestätigt der Nutzer, über die nötigen Rechte zu verfügen und interne sowie gesetzliche Vorgaben einzuhalten. Details hierzu finden sich in unseren [Allgemeinen Geschäfts- und Nutzungsbedingungen](https://www.novalutions.de/wp-content/uploads/2026/03/Allgemeine-Geschaefts-und-Nutzungsbedingungen-Teams-Chat-Exporter.pdf).

## Installation & Bereitstellung

### Erstinstallation

Die Installation erfolgt über den Import der Lösung in die Power Platform-Umgebung. Eine Schritt-für-Schritt-Anleitung finden Sie in unserem [Video-Tutorial](https://www.youtube.com/watch?v=aTV7nPn8j9I) und in der [schriftlichen Installationsanleitung](https://www.novalutions.de/teams-chat-exporter-freigabe/).

### Workflow-Freigabe

Nach der erstmaligen Installation müssen die Workflows für Mitarbeitende freigegeben werden:

1. Navigieren Sie zu [make.powerautomate.com](https://make.powerautomate.com) mit dem Account, der die Lösung importiert hat
2. Wählen Sie „Meine Flows" aus
3. Öffnen Sie jeden Workflow und bearbeiten Sie unter „Nur Ausführen Benutzer" die Berechtigungen
4. Vergeben Sie den Zugriff an einzelne Nutzer oder Gruppen (z. B. „All Company" für organisationsweite Freigabe)

### Organisationsweite Bereitstellung über Teams Admin Center

Für eine automatische Bereitstellung an alle oder ausgewählte Benutzer:

1. Navigieren Sie im Power Apps Maker Portal zum Teams Chat Exporter → Freigeben → Zu Teams hinzufügen
2. Laden Sie die App-ZIP-Datei herunter
3. Melden Sie sich im Teams Admin Center an → Teams-Apps → Apps verwalten
4. Laden Sie die ZIP-Datei als neue App hoch
5. Konfigurieren Sie die App-Einrichtungsrichtlinie: Fügen Sie den Teams Chat Exporter als angeheftete App hinzu

> **Hinweis:** Es kann bis zu 48 Stunden dauern, bis die Änderungen der App-Einrichtungsrichtlinie für die Endnutzer wirksam werden.

## Unterstützte Exporttypen

| Typ | Beschreibung |
|-----|-------------|
| 1:1-Chats | Direkte Chats zwischen zwei Personen |
| Gruppenchats | Chats mit mehreren Teilnehmern |
| Meeting-Chats | Chats im Kontext eines Teams-Meetings (optional ein-/ausblendbar) |
| Kanal-Konversationen | Thread-basierte Konversationen aus Teams-Kanälen inkl. aller Antworten |
| Chats mit Externen | Chats mit externen Teilnehmern, sofern deren Organisation dies nicht explizit unterbindet |
| Massenexport | Mehrere Chats und Kanäle können gleichzeitig in einem Vorgang exportiert werden |

## Unterstützte Inhaltstypen

### Nachrichten
- Textnachrichten mit vollständiger Formatierung
- Inline-Bilder (Hosted Content) – werden automatisch heruntergeladen und im Export eingebettet (seit Version 1.3.0.0)
- Dateianhänge aus SharePoint – alle Dateien, die im Chat unter dem Tab „Freigegeben" sichtbar sind
- Nachrichtenreferenzen (zitierte Nachrichten)

### Systemereignisse
Der Export erkennt und stellt folgende Teams-Systemereignisse dar:
- Teilnehmer beigetreten / hinzugefügt
- Teilnehmer verlassen
- Anruf gestartet / beendet
- Chat umbenannt

### Anhänge
- Automatischer Download von SharePoint-gehosteten Dateianhängen
- Automatischer Download von Inline-Hosted-Content (eingebettete Bilder)
- Anhänge werden im Unterordner „Dateien" gespeichert und im HTML-Export verlinkt
- Original-Dateinamen bleiben erhalten

### Bekannte Einschränkungen
- Bestimmte Grafiken können bei Fehlercode 403 (Forbidden) nicht exportiert werden – dies liegt an Tenant-spezifischen Sicherheits-, Zugriffs- oder Compliance-Richtlinien
- Gelöschte Nachrichten können nur exportiert werden, wenn sie zuvor gesichert wurden
- Online-GIFs werden nicht als Datei heruntergeladen

## Benutzerberechtigungen & Datenzugriff

Benutzer können ausschließlich Inhalte exportieren, auf die sie selbst Zugriff haben:
- Eigene Chat-Konversationen, an denen sie teilnehmen
- Kanal-Konversationen in Teams, in denen sie Mitglied sind
- Versteckte Chats werden automatisch ausgefiltert

### Erforderliche Konnektoren & API-Berechtigungen

| Konnektor | Verwendete Berechtigungen | Zweck |
|-----------|--------------------------|--------|
| Microsoft Teams | Chat.Read, ChannelMessage.Read.All | Lesen von Chat-Nachrichten, Kanal-Threads und Mitgliederlisten über die Graph API |
| SharePoint Online | Files.ReadWrite.All | Download von Dateianhängen, die in SharePoint gespeichert sind |
| OneDrive for Business | Files.ReadWrite | Speicherung der HTML-Exporte und heruntergeladenen Anhänge |
| Office 365 Users | User.Read.All | Auflösung von Benutzerprofilen und Display-Namen |

Alle Konnektoren verwenden **Invoker-Berechtigungen** – der Workflow läuft im Kontext des angemeldeten Benutzers und greift nur auf Daten zu, die diesem Benutzer zugänglich sind.

## Funktionsumfang im Detail

### Chat-Listing (GetChats-Workflow)
- Abruf aller Chats des Benutzers über die Microsoft Graph API mit automatischer Paginierung
- Erkennung des Chat-Typs: 1:1-Chat, Gruppenchat oder Meeting-Chat
- Optionaler Filter zum Ein-/Ausblenden von Meeting-Chats
- Anzeige von Chat-Thema, Teilnehmerzahl und Typ in der Power App
- Ausblendung versteckter Chats

### Chat-Export-Workflow
- Iterativer Abruf aller Nachrichten mit Paginierung (keine Nachrichtenlimits)
- Auflösung aller Chat-Teilnehmer mit Display-Namen
- Erkennung und Verarbeitung von sechs Systemereignistypen
- Automatischer Download und Einbettung von Inline-Bildern (Hosted Content)
- Automatischer Download von SharePoint-Anhängen
- Generierung einer vollständigen, interaktiven HTML-Exportdatei
- Speicherung im OneDrive des Benutzers mit automatischer Ordnerstruktur
- Erstellung eines organisationsweiten Freigabelinks
- Benachrichtigung des Benutzers per Teams-Nachricht mit Download-Link
- Mehrsprachige Unterstützung (Deutsch/Englisch) für Statusmeldungen

### Kanal-Export-Workflow
- Export eines oder mehrerer Konversations-Threads aus einem Kanal
- Abruf der Thread-Starternachricht und aller Antworten
- Auflösung von Team- und Kanalnamen für die Export-Metadaten
- Verarbeitung von Anhängen und Hosted Content analog zum Chat-Export
- Fehlerbehandlung bei fehlenden Berechtigungen
- Gleiche HTML-Ausgabe und Benachrichtigungslogik wie beim Chat-Export

### Exportdauer
Die Dauer hängt von der Datenmenge ab. Ein Chat mit 100 Nachrichten benötigt etwa 1–2 Minuten. Chats mit tausenden Nachrichten können bis zu einer Stunde benötigen. Der Export läuft im Hintergrund – der Computer oder die App muss dabei nicht geöffnet bleiben. Nach Abschluss wird der Benutzer automatisch per Teams-Nachricht benachrichtigt.

## HTML-Export

### Features der Exportdatei
- Responsives Design mit Tailwind CSS
- Volltextsuche über alle Nachrichten
- Filterung nach Datum
- Filterung nach Benutzer
- Zeitstempel und Benutzeridentifikation pro Nachricht
- Direkte Links zu heruntergeladenen Anhängen
- Eingebettete Inline-Bilder

### Dateistruktur im OneDrive

```
TeamsChatExport/
├── YYYY-MM-DD - Chatname/
│   ├── Export - YYYY-MM-DD - Chatname.html
│   └── Dateien/
│       ├── dokument.docx
│       ├── bild.jpg
│       └── ...
└── YYYY-MM-DD - Teamname/
    ├── Export - YYYY-MM-DD - Teamname.html
    └── Dateien/
        └── ...
```

## Lizenzprüfung

Die Lösung enthält eine integrierte Lizenzprüfung, die die Anzahl aktiver Benutzer (ohne Gastkonten) im Tenant mit der lizenzierten Menge abgleicht. Bei Überschreitung wird der Benutzer benachrichtigt und der Export als Evaluierung markiert.

## Anpassungen & Support

Eine individuelle Anpassung des Teams Chat Exporters ist nicht gestattet (siehe [AGB](https://www.novalutions.de/wp-content/uploads/2026/03/Allgemeine-Geschaefts-und-Nutzungsbedingungen-Teams-Chat-Exporter.pdf)). Für individuelle Anpassungswünsche kann eine Support-Anfrage gestellt werden. Individueller Support erfolgt nach Aufwand mit einem Stundensatz von 140 €. Enterprise-Kunden erhalten regelmäßige, kostenlose Updates mit Verbesserungen und neuen Funktionen sowie exklusiven Zugang zum Support-Postfach.

## Ressourcen & Weiterführende Informationen

### Video-Tutorials

#### Installations-Anleitung (aktuell)
[![Teams Chat Exporter Installation](https://img.youtube.com/vi/aTV7nPn8j9I/0.jpg)](https://www.youtube.com/watch?v=aTV7nPn8j9I)

#### Produktvorstellung
[![Teams Chat Exporter Vorstellung](https://img.youtube.com/vi/ANSKcxP64bU/0.jpg)](https://www.youtube.com/watch?v=ANSKcxP64bU)

### Weitere Informationen
- [Offizielle Produktseite](https://www.novalutions.de/teams-chats-exportieren/) – Detaillierte Produktinformationen und Preisgestaltung
- [Installationsanleitung & Freigabe](https://www.novalutions.de/teams-chat-exporter-freigabe/) – Schritt-für-Schritt-Anleitung zur Einrichtung und Freigabe
- [Allgemeine Geschäfts- und Nutzungsbedingungen (AGB)](https://www.novalutions.de/wp-content/uploads/2026/03/Allgemeine-Geschaefts-und-Nutzungsbedingungen-Teams-Chat-Exporter.pdf) – Vertragsbedingungen und Nutzungsrechte
- [Support & Kontakt](https://www.novalutions.de/teams-chats-exportieren/#contact) – Direkter Kontakt zum Support-Team
- [Kostenlose Demo](https://www.novalutions.de/teams-chats-exportieren/#demo) – Vereinbaren Sie eine persönliche Produktvorführung

### Lizenzierung & Preise
Der Teams Chat Exporter ist als Einmallizenz verfügbar – keine laufenden Kosten oder Abonnements. Optional kann ein Installationsservice hinzugebucht werden. Enterprise-Lizenzen beinhalten 60 Minuten Installationssupport (Remote), regelmäßige Updates sowie Zugang zum Support-Postfach. Für detaillierte Preisinformationen besuchen Sie die [Produktseite](https://www.novalutions.de/teams-chats-exportieren/). Es gelten die [Allgemeinen Geschäfts- und Nutzungsbedingungen](https://www.novalutions.de/wp-content/uploads/2026/03/Allgemeine-Geschaefts-und-Nutzungsbedingungen-Teams-Chat-Exporter.pdf).
