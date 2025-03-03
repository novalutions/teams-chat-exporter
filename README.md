# Teams Chat Exporter - Technische Dokumentation

## Überblick
Der Teams Chat Exporter ist eine Lösung bestehend aus zwei Power Automate Workflows, die es Benutzern ermöglichen, Teams-Chats und Kanal-Konversationen zu exportieren. Die Exporte werden als interaktive HTML-Dateien gespeichert und können durchsucht und gefiltert werden.

## Benutzerberechtigungen & Datenzugriff

Benutzer können folgende Inhalte exportieren:
- Eigene Chat-Konversationen, an denen sie teilnehmen
- Kanal-Konversationen in Teams, in denen sie Mitglied sind
- Antworten auf Nachrichten in Kanälen (Thread-Export)

### Erforderliche API-Berechtigungen

| API | Berechtigung | Gelesene Informationen | Zweck |
|-----|--------------|------------------------|--------|
| Microsoft Teams | Chat.Read | - Nachrichteninhalte<br>- Zeitstempel<br>- Absender<br>- Anhänge | Extraktion der Chat-Verläufe |
| Microsoft Teams | ChannelMessage.Read.All | - Kanal-Nachrichten<br>- Thread-Antworten<br>- Nachrichtenmetadaten | Export von Kanal-Konversationen |
| SharePoint Online | Files.ReadWrite.All | - Dateianhänge<br>- Datei-Metadaten | Speicherung von Chat-Anhängen |
| OneDrive for Business | Files.ReadWrite | - Export-Dateien<br>- Ordnerstrukturen | Speicherung der HTML-Exporte |
| Azure AD | User.Read | - Benutzerprofile<br>- E-Mail-Adressen<br>- Display Names | Benutzeridentifikation |

## Komponenten

Die Lösung besteht aus zwei Workflows:
1. Chat-Export Workflow (`TeamsExporter-Chatexportieren`)
2. Kanal-Export Workflow (`TeamsExporter-Kanalchatexportieren`) 

## Workflow 1: Chat-Export

### Trigger
- Manueller Trigger über Teams-Menü
- Adaptive Card Interface mit Exportbestätigung
- Benutzer können nur Chats exportieren, an denen sie aktiv teilnehmen

### Hauptfunktionen
1. **Benutzeridentifikation**
   - Erfasst den aktuellen Benutzer
   - Sammelt Teilnehmer des Chats für die Export-Metadaten
   - Validiert Zugriffsberechtigungen

2. **Nachrichtenextraktion**
   - Iterative Abfrage aller Chatnachrichten via Microsoft Graph API
   - Verarbeitung von Text und Anhängen
   - Unterstützung für verschiedene Nachrichtentypen
   - Berücksichtigung von Chat-Berechtigungen

3. **Anhangverarbeitung**
   - Automatische Extraktion von Anhängen
   - Speicherung in separatem "Dateien"-Ordner
   - Verlinkung der Anhänge im Export
   - Beibehaltung der Original-Dateinnamen

4. **Export-Generierung**
   - Erstellung einer interaktiven HTML-Datei
   - Responsive Design mit Tailwind CSS
   - Such- und Filterfunktionen:
     - Volltextsuche in Nachrichten
     - Filterung nach Datum
     - Filterung nach Benutzer
   - Zeitstempel und Benutzeridentifikation

### Ausgabe
- HTML-Datei im OneDrive-Ordner "TeamsChatExport"
- Benachrichtigung an den Benutzer mit Download-Link
- Organisationsweiter Lesezugriff auf den Export

## Workflow 2: Kanal-Export

### Trigger
- Kontextmenü einer Kanal-Nachricht
- Exportiert den kompletten Thread
- Nur verfügbar für Teammitglieder

### Hauptfunktionen
1. **Thread-Identifikation**
   - Erfassung der Ursprungsnachricht
   - Sammlung aller Antworten
   - Validierung der Teamzugehörigkeit

2. **Team-Kontext**
   - Erfassung des Team- und Kanalnamens
   - Integration in Export-Metadaten
   - Dokumentation der Teamstruktur

3. **Nachrichtenverarbeitung**
   - Ähnlich wie Chat-Export
   - Spezielle Behandlung von Kanal-spezifischen Formaten
   - Erhalt der Thread-Struktur

### Ausgabe
- HTML-Datei mit Thread-Kontext
- Gleiche Funktionalität wie Chat-Export
- Team- und Kanalname in Metadaten

## Technische Details

### Dateistruktur 

## Ressourcen & Weiterführende Informationen

### Video-Tutorials

#### Installations-Anleitung
[![Teams Chat Exporter Installation](https://img.youtube.com/vi/BV5YA9Lu-mY/0.jpg)](https://www.youtube.com/watch?v=BV5YA9Lu-mY)

#### Produktvorstellung
[![Teams Chat Exporter Vorstellung](https://img.youtube.com/vi/ANSKcxP64bU/0.jpg)](https://www.youtube.com/watch?v=ANSKcxP64bU)

### Weitere Informationen
- [Offizielle Produktseite](https://www.novalutions.de/teams-chats-exportieren/) - Detaillierte Produktinformationen und Preisgestaltung
- [Support & Kontakt](https://www.novalutions.de/teams-chats-exportieren/#contact) - Direkter Kontakt zum Support-Team
- [Kostenlose Demo](https://www.novalutions.de/teams-chats-exportieren/#demo) - Vereinbaren Sie eine persönliche Produktvorführung

### Lizenzierung & Preise
Der Teams Chat Exporter ist als Einmallizenz für 119€ verfügbar. Optional kann ein Installationsservice hinzugebucht werden.

Für detaillierte Preisinformationen und Enterprise-Lizenzen besuchen Sie bitte die [Produktseite](https://www.novalutions.de/teams-chats-exportieren/). 
