# Teams Chat Exporter – Technische Dokumentation

🇬🇧 [English version](README_EN.md)

## Inhalt

1. [Überblick](#überblick)
2. [Architektur](#architektur)
3. [Voraussetzungen](#voraussetzungen)
4. [Installation](#installation)
5. [Bedienung](#bedienung)
6. [Unterstützte Exporttypen](#unterstützte-exporttypen)
7. [Unterstützte Inhaltstypen](#unterstützte-inhaltstypen)
8. [Struktur des Exports](#struktur-des-exports)
9. [Berechtigungen & Datenzugriff](#berechtigungen--datenzugriff)
10. [Bekannte Einschränkungen](#bekannte-einschränkungen)
11. [Fehlerbehebung](#fehlerbehebung)
12. [Lizenzierung](#lizenzierung)
13. [Anpassungen & Support](#anpassungen--support)
14. [Versionshistorie](#versionshistorie)
15. [Ressourcen](#ressourcen)

## Überblick

Der Teams Chat Exporter ist eine Power-Platform-Lösung von [novalutions](https://www.novalutions.de), mit der Mitarbeitende ihre Microsoft-Teams-Chats und Kanalunterhaltungen inklusive Anhängen und eingebetteter Bilder selbstständig exportieren. Jeder Export wird als interaktive HTML-Datei mit Volltextsuche und Datumsfilter im OneDrive des jeweiligen Benutzers abgelegt.

Die Lösung wird vom Kunden im eigenen Microsoft-365-Tenant installiert. Alle Daten bleiben im Tenant, novalutions hat keinen Zugriff auf Chats oder Exporte. Eine exportierte HTML-Datei kann nach dem Herunterladen ohne Microsoft-365-Konto geöffnet und archiviert werden.

Aktuelle Version: **1.3.3.5**

## Architektur

Die Lösung wird als verwaltete Power-Platform-Lösung (Managed Solution) „TeamsChatExporter“ des Herausgebers „novalutions“ ausgeliefert und besteht aus folgenden Komponenten:

| Komponente | Typ | Aufgabe |
|---|---|---|
| Teams Chat Exporter | Canvas-App (in Teams eingebettet) | Benutzeroberfläche: Chat-Liste mit Filter nach Chat-Typ und Suche, Mehrfachauswahl, Auswahl von Team, Kanal und Unterhaltungen, Link zu den bisherigen Exporten. Oberfläche auf Deutsch und Englisch. |
| Teams Chat Exporter – Get Chats | Power-Automate-Flow | Lädt die Chat-Liste des Benutzers seitenweise über Microsoft Graph und liefert den Link zum Exportordner im OneDrive. |
| Teams Exporter – InvokedWorkflowChat | Power-Automate-Flow | Exportiert einen 1:1-, Gruppen- oder Besprechungs-Chat. Wird für jeden ausgewählten Chat einzeln gestartet. |
| Teams Exporter – InvokedWorkflowConversation | Power-Automate-Flow | Exportiert eine oder mehrere Unterhaltungen (Threads) eines Teams-Kanals. |
| 4 Verbindungsreferenzen | Office 365 Users, OneDrive for Business, SharePoint, Microsoft Teams | Zugriff auf Microsoft Graph, Dateien und Teams-Benachrichtigungen. |

Ablauf eines Exports:

1. Der Benutzer wählt in der App einen oder mehrere Chats bzw. Kanalunterhaltungen aus und startet den Export.
2. Die App startet pro Chat (bzw. pro Kanal-Export) einen Flow. Der Export läuft im Hintergrund weiter, die App kann geschlossen werden.
3. Der Flow liest Nachrichten, Teilnehmer, Anhänge und eingebettete Bilder über Microsoft Graph, erzeugt die HTML-Datei und speichert alles im OneDrive des Benutzers.
4. Der Benutzer erhält eine Teams-Nachricht vom Flow bot mit einem Link zum Exportordner.

Alle Flows laufen mit den Verbindungen des aufrufenden Benutzers. Es wird keine App-Registrierung in Microsoft Entra ID benötigt und es werden keine Daten an Systeme außerhalb des Tenants übertragen.

> **Hinweis:** Die früheren Workflows, die direkt aus dem Nachrichtenmenü eines Chats aufgerufen wurden, werden seit Version 1.2.0.0 nicht mehr unterstützt (von Microsoft seit 01.05.2025 abgekündigt). Der Export erfolgt ausschließlich über die App.

## Voraussetzungen

### Für die Installation (Administrator)

| Voraussetzung | Details |
|---|---|
| Microsoft-365-Lizenz | Mindestens Microsoft 365 Business Basic (oder höher, z. B. Business Standard/Premium, E3, E5) mit Teams, OneDrive und SharePoint. **Microsoft Teams Essentials wird nicht unterstützt.** |
| Power-Platform-Umgebung | Eine Umgebung, in die Lösungen importiert werden können (mit Dataverse), z. B. die Standardumgebung des Tenants. |
| Rolle in der Umgebung | System Administrator, System Customizer oder Environment Admin. Diese Rolle wird nur für Import und Freigabe benötigt, nicht für die spätere Nutzung. |
| Teams-Administrator | Nur für die organisationsweite Verteilung über das Teams Admin Center erforderlich (Rolle Teams-Administrator oder Globaler Administrator). |
| Benutzerdefinierte Teams-Apps | Die organisationsweite App-Einstellung im Teams Admin Center muss benutzerdefinierte Apps zulassen. |
| Konnektoren | Die Konnektoren Office 365 Users, OneDrive for Business, SharePoint und Microsoft Teams dürfen nicht durch eine Data-Loss-Prevention-Richtlinie (DLP) der Umgebung blockiert oder voneinander getrennt sein. |
| Lösungsdatei | Die Lösungsdatei (ZIP) erhalten Sie von novalutions nach dem Kauf oder für die kostenlose Testversion. |

Es werden ausschließlich Standard-Konnektoren verwendet. Eine Power-Apps- oder Power-Automate-Premiumlizenz ist nicht erforderlich.

### Für die Nutzung (Mitarbeitende)

- Microsoft-365-Lizenz mit Teams und OneDrive (z. B. Business Basic)
- Freigabe der App und der drei Flows durch den Administrator (siehe [Installation](#installation))
- Beim ersten Start einmalige Zustimmung zu den Verbindungen in der App
- Zum Ansehen des Exports ein aktueller Browser. Für die vollständige Darstellung lädt die HTML-Datei ein Stylesheet aus dem Internet (cdnjs.cloudflare.com). Ohne Internetverbindung bleiben alle Inhalte lesbar, nur das Layout ist vereinfacht.

Mit dem Kauf bestätigt der Kunde, über die nötigen Rechte zu verfügen und interne sowie gesetzliche Vorgaben einzuhalten. Details siehe [Allgemeine Geschäfts- und Nutzungsbedingungen](https://www.novalutions.de/wp-content/uploads/2026/03/Allgemeine-Geschaefts-und-Nutzungsbedingungen-Teams-Chat-Exporter.pdf).

## Installation

Alle Schritte werden mit dem Administratorkonto ausgeführt, das die Lösung importiert. Eine Videoanleitung finden Sie unter [Ressourcen](#ressourcen).

### Schritt 1: Lösung importieren

1. Öffnen Sie [make.powerapps.com](https://make.powerapps.com) und wählen Sie oben rechts die Zielumgebung aus.
2. Klicken Sie im linken Menü auf „Lösungen“ und anschließend oben auf „Lösung importieren“.
3. Klicken Sie auf „Durchsuchen“, wählen Sie die Lösungsdatei (ZIP) aus und klicken Sie auf „Weiter“.
4. Auf der Seite „Verbindungen“ wird für jeden der vier Konnektoren (Office 365 Users, OneDrive for Business, SharePoint, Microsoft Teams) eine Verbindung benötigt. Wählen Sie jeweils eine vorhandene Verbindung aus oder legen Sie über „Neue Verbindung“ eine an und melden Sie sich an.
5. Klicken Sie auf „Importieren“ und warten Sie, bis die Erfolgsmeldung erscheint. Der Import kann einige Minuten dauern.

### Schritt 2: Flows prüfen

1. Öffnen Sie unter „Lösungen“ die Lösung „Teams Chat Exporter“.
2. Wählen Sie links den Bereich „Cloud-Flows“ aus.
3. Prüfen Sie, dass alle drei Flows den Status „Ein“ haben. Ist ein Flow ausgeschaltet, markieren Sie ihn und klicken Sie auf „Einschalten“.

### Schritt 3: Flows für Mitarbeitende freigeben

Jeder der drei Flows muss für die Benutzer als „Nur ausführen“ freigegeben werden:

- Teams Chat Exporter – Get Chats
- Teams Exporter – InvokedWorkflowChat
- Teams Exporter – InvokedWorkflowConversation

Für jeden Flow:

1. Öffnen Sie [make.powerautomate.com](https://make.powerautomate.com) in derselben Umgebung und öffnen Sie den Flow (über „Lösungen“ oder „Meine Flows“).
2. Klicken Sie auf der Detailseite im Abschnitt „Nur-Ausführen-Benutzer“ auf „Bearbeiten“.
3. Fügen Sie die gewünschten Benutzer oder eine Gruppe hinzu, für eine organisationsweite Freigabe z. B. eine Gruppe mit allen Mitarbeitenden.
4. Die Verbindungen sind in der Lösung bereits so eingestellt, dass der Flow mit der Verbindung des jeweiligen Benutzers läuft. Lassen Sie diese Einstellung unverändert und klicken Sie auf „Speichern“.

### Schritt 4: App freigeben

1. Öffnen Sie in [make.powerapps.com](https://make.powerapps.com) den Bereich „Apps“.
2. Klicken Sie bei „Teams Chat Exporter“ auf die drei Punkte und anschließend auf „Freigeben“.
3. Fügen Sie dieselben Benutzer oder Gruppen hinzu wie bei den Flows (ohne Miteigentümer-Rechte) und klicken Sie auf „Freigeben“.

### Schritt 5: App in Teams bereitstellen

Wählen Sie eine der beiden Varianten.

**Variante A: Für einzelne Benutzer (z. B. Test)**

1. Klicken Sie in [make.powerapps.com](https://make.powerapps.com) unter „Apps“ bei „Teams Chat Exporter“ auf die drei Punkte, dann auf „Zu Teams hinzufügen“.
2. Klicken Sie im Dialog auf „Zu Teams hinzufügen“. Teams öffnet sich und die App kann für das eigene Konto hinzugefügt werden.

**Variante B: Organisationsweit über das Teams Admin Center (empfohlen)**

1. Klicken Sie in [make.powerapps.com](https://make.powerapps.com) unter „Apps“ bei „Teams Chat Exporter“ auf die drei Punkte, dann auf „Zu Teams hinzufügen“, und wählen Sie im Dialog „App herunterladen“. Sie erhalten das App-Paket als ZIP-Datei.
2. Melden Sie sich im [Teams Admin Center](https://admin.teams.microsoft.com) an und öffnen Sie „Teams-Apps“ und dort „Apps verwalten“.
3. Klicken Sie auf „Aktionen“ und dann auf „Neue App hochladen“. Wählen Sie die ZIP-Datei aus. Es kann bis zu einer Stunde dauern, bis die App in der Liste erscheint.
4. Öffnen Sie unter „Teams-Apps“ den Bereich „Einrichtungsrichtlinien“ und wählen Sie die gewünschte Richtlinie, z. B. „Global (organisationsweiter Standard)“.
5. Klicken Sie im Abschnitt „Angeheftete Apps“ auf „Apps hinzufügen“, suchen Sie „Teams Chat Exporter“, klicken Sie auf „Hinzufügen“ und anschließend auf „Speichern“.

> **Hinweis:** Änderungen an Einrichtungsrichtlinien können bis zu 48 Stunden dauern, bis sie bei allen Benutzern wirksam sind.

### Schritt 6: Erster Start durch den Benutzer

1. Der Benutzer öffnet in der linken Teams-Leiste den „Teams Chat Exporter“. Ist die App nicht angeheftet, findet er sie über die drei Punkte bzw. „Apps“.
2. Beim ersten Start fragt die App nach der Zustimmung zu den Verbindungen (Office 365 Users, OneDrive for Business, SharePoint, Microsoft Teams). Mit „Zulassen“ bestätigen.
3. Die App lädt anschließend die Chat-Liste. Beim ersten Laden kann das je nach Anzahl der Chats etwas dauern, danach wird die Liste lokal zwischengespeichert.

### Updates

Neue Versionen werden ebenfalls über „Lösungen“ und „Lösung importieren“ in dieselbe Umgebung eingespielt. Die vorhandene Lösung wird dabei aktualisiert, Verbindungen und Freigaben bleiben in der Regel erhalten. Prüfen Sie nach dem Update, dass alle drei Flows eingeschaltet sind (Schritt 2).

### Deinstallation

Löschen Sie unter „Lösungen“ die Lösung „Teams Chat Exporter“ und entfernen Sie die App im Teams Admin Center. Bereits erstellte Exporte bleiben im OneDrive der Benutzer erhalten.

## Bedienung

**Chats exportieren**

1. Im Bereich der Chats die Liste bei Bedarf nach Typ (Einzel-, Gruppen-, Besprechungs-Chats) filtern oder über die Suche eingrenzen.
2. Einen oder mehrere Chats auswählen und den Export starten.

**Kanalunterhaltungen exportieren**

1. Im Bereich der Kanäle ein Team und einen Kanal auswählen.
2. Einzelne Unterhaltungen oder alle Unterhaltungen des Kanals auswählen und den Export starten.

**Export öffnen**

1. Nach Abschluss sendet der Flow bot in Teams eine Nachricht mit dem Link „Download öffnen“.
2. Den Link öffnen und den Ordner über „Herunterladen“ als ZIP-Datei speichern.
3. Die ZIP-Datei entpacken und die HTML-Datei per Doppelklick im Browser öffnen. Die HTML-Datei und der Unterordner „Dateien“ müssen zusammenbleiben, damit Bilder und Anhänge verlinkt bleiben.

Über die Schaltfläche „Exportierte Chats öffnen“ in der App gelangen Sie jederzeit zum Ordner „TeamsChatExport“ im eigenen OneDrive.

**Exportdauer:** Etwa 1 bis 2 Minuten pro 100 Nachrichten. Chats mit mehreren tausend Nachrichten können bis zu einer Stunde benötigen. Der Export läuft im Hintergrund, Teams und der Computer müssen dafür nicht geöffnet bleiben.

## Unterstützte Exporttypen

| Typ | Beschreibung |
|---|---|
| 1:1-Chats | Chats zwischen zwei Personen |
| Gruppenchats | Chats mit mehreren Teilnehmern, mit oder ohne Thema |
| Besprechungs-Chats | Chats zu Teams-Besprechungen. Können in der App ein- oder ausgeblendet werden. |
| Kanalunterhaltungen | Einzelne oder alle Unterhaltungen (Threads) eines Kanals, jeweils mit Startnachricht und allen Antworten |
| Chats mit Externen | Chats mit Personen anderer Organisationen, sofern deren Organisation dies nicht unterbindet |
| Mehrfachexport | Mehrere Chats bzw. mehrere Kanalunterhaltungen in einem Vorgang |

Ausgeblendete Chats werden in der Chat-Liste nicht angezeigt.

## Unterstützte Inhaltstypen

| Inhalt | Chat-Export | Kanal-Export | Hinweise |
|---|:---:|:---:|---|
| Textnachrichten mit Formatierung | ✅ | ✅ | Formatierung, Links und Erwähnungen bleiben erhalten. Links werden mit 🔗 gekennzeichnet. |
| Absender und Zeitstempel | ✅ | ✅ | Pro Nachricht |
| Betreff der Unterhaltung | – | ✅ | Wird als Überschrift der Unterhaltung angezeigt |
| Zitierte Nachrichten (Antworten) | ✅ | – | Mit Absender und Vorschau; Klick springt zur Originalnachricht |
| Eingebettete Bilder (z. B. eingefügte Screenshots) | ✅ | ✅ | Werden heruntergeladen, lokal gespeichert und im Export angezeigt (seit Version 1.3.0.0) |
| Dateianhänge | ✅ | ✅ | Dateien aus OneDrive bzw. SharePoint, die im Chat oder Kanal geteilt wurden, werden mit Originalnamen heruntergeladen und verlinkt. Bildanhänge werden direkt angezeigt. |
| Systemereignisse | ✅ | – | Teilnehmer beigetreten, hinzugefügt, verlassen; Anruf gestartet, beendet; Chat umbenannt |
| Reaktionen (Emoji-Reaktionen) | – | – | Werden nicht exportiert |
| Gelöschte Nachrichten | – | – | Können nicht exportiert werden |
| Online-GIFs | – | – | Werden nicht heruntergeladen |
| Adaptive Cards, Loop-Komponenten, Umfragen | – | – | Werden nicht als Inhalt dargestellt |
| Besprechungsaufzeichnungen und Transkripte | – | – | Nicht Bestandteil des Exports |

## Struktur des Exports

### Ablage im OneDrive

Alle Exporte werden im OneDrive des Benutzers im Ordner `TeamsChatExport` abgelegt. Für jeden Export wird ein eigener Unterordner mit dem Exportdatum angelegt.

```
TeamsChatExport/
├── 2026-10-12 - Projekt Alpha/                 ← Chat-Export
│   ├── Export - 2026-10-12 - Projekt Alpha.html
│   ├── metadata.txt
│   └── Dateien/
│       ├── Angebot.pdf                         ← Anhang (Originalname)
│       └── 0-weu-d1-3f2a….png                  ← Eingebettetes Bild
└── 2026-10-12- Vertrieb - Allgemein/           ← Kanal-Export (Team - Kanal)
    ├── Export-2026-10-12.html
    └── Dateien/
        └── …
```

- **Ordnername Chat-Export:** Exportdatum und Chat-Name. Der Chat-Name ist das Thema des Chats oder, falls keines vergeben ist, die Namen der Teilnehmenden (gekürzt auf 125 Zeichen). Zeichen, die in Dateinamen nicht erlaubt sind, werden entfernt.
- **Ordnername Kanal-Export:** Exportdatum, Teamname und Kanalname. Alle in einem Vorgang ausgewählten Unterhaltungen landen in einer HTML-Datei.
- **metadata.txt** (nur Chat-Export): Erstellungsdatum des Chats, Link zum Chat in Teams, Anzahl der Teilnehmenden und Anzahl der Nachrichten.
- **Dateien/**: Anhänge mit ihrem Originalnamen sowie eingebettete Bilder, die mit einer eindeutigen Kennung benannt werden.
- **Freigabelink:** Für den Exportordner wird ein Ansichtslink erstellt, der nur für Personen in der eigenen Organisation gilt. Er wird ausschließlich an den exportierenden Benutzer gesendet.

### Aufbau der HTML-Datei

- Kopfzeile mit Chat- bzw. Kanalname und Exportdatum
- Suchfeld für die Volltextsuche über Nachrichteninhalt und Absendernamen
- Datumsfilter mit Start- und Enddatum
- Nachrichten im Teams-Stil: eigene Nachrichten rechts, Nachrichten anderer links, jeweils mit Absender und Zeitstempel
- Eingebettete Bilder, ein Klick öffnet das Bild in voller Größe
- Verlinkte Anhänge aus dem Ordner „Dateien“
- Zitierte Nachrichten mit Sprung zur Originalnachricht (Chat-Export)
- Kanal-Export: Unterhaltungen gruppiert mit Betreff, Anzahl der Nachrichten und Zeitraum, einzeln auf- und zuklappbar
- Druckansicht: Such- und Filterleiste werden beim Drucken bzw. beim Speichern als PDF ausgeblendet

## Berechtigungen & Datenzugriff

Benutzer können ausschließlich Inhalte exportieren, auf die sie selbst Zugriff haben:

- Chats, an denen sie teilnehmen
- Kanalunterhaltungen in Teams, in denen sie Mitglied sind
- Anhänge, auf die sie in SharePoint bzw. OneDrive zugreifen dürfen

Alle Verbindungen laufen im Kontext des angemeldeten Benutzers (delegierte Berechtigungen). Die Lösung benötigt keine App-Registrierung und keine Administratorzustimmung für zusätzliche Graph-Berechtigungen.

### Verwendete Konnektoren und Schnittstellen

| Konnektor | Verwendete Aktionen | Zweck |
|---|---|---|
| Microsoft Teams | HTTP-Anfrage an Microsoft Graph, Team abrufen, Kanal abrufen, Nachricht als Flow bot senden | Chats, Chat-Teilnehmer, Nachrichten, Kanalunterhaltungen mit Antworten und eingebettete Bilder lesen (`/me/chats`, `/me/chats/{id}/messages`, `/teams/{id}/channels/{id}/messages/{id}/replies`, `…/hostedContents`); Benachrichtigung an den Benutzer |
| Office 365 Users | HTTP-Anfrage an Microsoft Graph | Profil des exportierenden Benutzers (`/users/{id}`) und Anzahl der aktiven Konten für die Lizenzprüfung (`/users/$count`) |
| SharePoint | Dateiinhalt über Pfad abrufen | Herunterladen geteilter Dateianhänge |
| OneDrive for Business | Datei erstellen, Freigabelink erstellen | Speichern von HTML-Datei, Anhängen und metadata.txt; Link zum Exportordner |

## Bekannte Einschränkungen

- **Gelöschte Nachrichten** können nicht exportiert werden.
- **Online-GIFs** werden nicht heruntergeladen.
- **Reaktionen** auf Nachrichten werden nicht exportiert.
- **Fehler 403 bei Bildern oder Anhängen:** Einzelne Inhalte können durch Sicherheits-, Zugriffs- oder Compliance-Richtlinien des Tenants blockiert sein (siehe [Fehlerbehebung](#fehlerbehebung)).
- **Chat-Liste:** Die App zeigt bis zu 1.250 Chats pro Benutzer an.
- **Kanal-Export:** Pro Unterhaltung werden bis zu 1.250 Antworten exportiert. Systemereignisse werden im Kanal-Export nicht dargestellt.
- **Chats mit Externen:** Nur möglich, wenn die externe Organisation den Zugriff nicht einschränkt.
- **Darstellung:** Für das vollständige Layout benötigt die HTML-Datei beim Öffnen eine Internetverbindung (siehe [Voraussetzungen](#voraussetzungen)).

## Fehlerbehebung

### Bilder oder Anhänge fehlen (Fehler 403)

Ein 403-Fehler bedeutet, dass Microsoft 365 den Zugriff auf einen Inhalt verweigert. Das liegt fast immer an Richtlinien im Tenant. Prüfen Sie diese drei Bereiche:

1. **Bedingter Zugriff (Conditional Access):** Richtlinien in Microsoft Entra ID, die den Zugriff auf Microsoft Graph oder SharePoint einschränken, etwa nach Gerät, Standort oder App.
2. **Mandantenübergreifender Zugriff (Cross-Tenant/B2B):** Inhalte, die von Gästen oder externen Personen geteilt wurden, liegen im Tenant der anderen Organisation. Deren Einstellungen können den Zugriff verhindern.
3. **Microsoft Purview:** DLP-Richtlinien oder Vertraulichkeitsbezeichnungen (Sensitivity Labels) können das Herunterladen bestimmter Dateien blockieren.

### Die App erscheint nicht in Teams

- Wartezeit beachten: bis zu einer Stunde nach dem Hochladen, bis zu 48 Stunden nach Änderung einer Einrichtungsrichtlinie.
- Prüfen, ob der Benutzer der richtigen Einrichtungsrichtlinie zugewiesen ist.
- Prüfen, ob benutzerdefinierte Apps in den organisationsweiten App-Einstellungen zugelassen sind und die App unter „Apps verwalten“ den Status „Zugelassen“ hat.

### Die App meldet einen Fehler beim Laden oder Exportieren

- Prüfen, ob die App und alle drei Flows für den Benutzer freigegeben sind (Installation, Schritte 3 und 4).
- Prüfen, ob alle drei Flows eingeschaltet sind (Installation, Schritt 2).
- Prüfen, ob eine DLP-Richtlinie der Umgebung einen der vier Konnektoren blockiert.
- Fehler während eines Exports meldet der Flow bot mit einer Fehlerbeschreibung per Teams-Nachricht.

## Lizenzierung

Der Teams Chat Exporter wird als Einmallizenz ohne laufende Kosten angeboten:

| Edition | Aktive Konten | Enthaltene Leistungen |
|---|---|---|
| S | 1–49 | 15 Minuten Installationssupport |
| L | 50–250 | 15 Minuten Installationssupport |
| Enterprise | ab 250 | 60 Minuten Installationssupport (remote), Support-Postfach, regelmäßige Updates |

Gezählt werden aktive Benutzerkonten im Tenant ohne Gastkonten. Aktuelle Preise finden Sie auf der [Produktseite](https://www.novalutions.de/teams-chats-exportieren/). Eine kostenlose Testversion ist auf Anfrage erhältlich.

Die Lösung prüft bei jedem Export, ob die Anzahl aktiver Konten von der Lizenz abgedeckt ist. Ist das nicht der Fall, erhält der Benutzer einen Hinweis per Teams-Nachricht und die Exportdatei wird als Evaluierungsversion gekennzeichnet. In diesem Fall wenden Sie sich bitte an [teamschatexporter@novalutions.de](mailto:teamschatexporter@novalutions.de).

## Anpassungen & Support

Individuelle Anpassungen der Lösung durch den Kunden sind nicht gestattet (siehe [AGB](https://www.novalutions.de/wp-content/uploads/2026/03/Allgemeine-Geschaefts-und-Nutzungsbedingungen-Teams-Chat-Exporter.pdf)). Anpassungswünsche können als Support-Anfrage gestellt werden.

Support über den enthaltenen Installationssupport hinaus erfolgt nach Aufwand mit einem Stundensatz von 140 € netto. Die Antwortzeit beträgt in der Regel 2 bis 3 Werktage.

**Kontakt:** [teamschatexporter@novalutions.de](mailto:teamschatexporter@novalutions.de) · novalutions UG (haftungsbeschränkt), Köln

## Versionshistorie

| Version | Änderung |
|---|---|
| 1.3.3.5 | Aktuelle Version |
| 1.3.0.0 | Export eingebetteter Bilder (Hosted Contents) |
| 1.2.0.0 | Direkt aus dem Chat aufgerufene Workflows entfallen (von Microsoft seit 01.05.2025 abgekündigt), Export ausschließlich über die App |
| 1.1.0.16 | Flow „Get Teams“ entfällt |

## Ressourcen

### Videos

**Installationsanleitung**

[![Teams Chat Exporter Installation](https://img.youtube.com/vi/aTV7nPn8j9I/0.jpg)](https://www.youtube.com/watch?v=aTV7nPn8j9I)

**Produktdemo**

[![Teams Chat Exporter Demo](https://img.youtube.com/vi/1Y-eobVuMUk/0.jpg)](https://www.youtube.com/watch?v=1Y-eobVuMUk)

**Produktvorstellung**

[![Teams Chat Exporter Vorstellung](https://img.youtube.com/vi/ANSKcxP64bU/0.jpg)](https://www.youtube.com/watch?v=ANSKcxP64bU)

### Links

- [Produktseite](https://www.novalutions.de/teams-chats-exportieren/) – Funktionen, Preise, Testversion
- [Installation & Freigabe mit Screenshots](https://www.novalutions.de/teams-chat-exporter-freigabe/) – ergänzende bebilderte Anleitung
- [Allgemeine Geschäfts- und Nutzungsbedingungen](https://www.novalutions.de/wp-content/uploads/2026/03/Allgemeine-Geschaefts-und-Nutzungsbedingungen-Teams-Chat-Exporter.pdf)
- [Datenschutzerklärung](https://www.novalutions.de/datenschutzerklaerung/)
- [Kontakt per E-Mail](mailto:teamschatexporter@novalutions.de)
