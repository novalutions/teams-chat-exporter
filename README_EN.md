# Teams Chat Exporter – Technical Documentation

🇩🇪 [Deutsche Version](README.md)

## Contents

1. [Overview](#overview)
2. [Architecture](#architecture)
3. [Prerequisites](#prerequisites)
4. [Installation](#installation)
5. [Usage](#usage)
6. [Supported Export Types](#supported-export-types)
7. [Supported Content Types](#supported-content-types)
8. [Export Structure](#export-structure)
9. [Permissions & Data Access](#permissions--data-access)
10. [Known Limitations](#known-limitations)
11. [Troubleshooting](#troubleshooting)
12. [Licensing](#licensing)
13. [Customization & Support](#customization--support)
14. [Version History](#version-history)
15. [Resources](#resources)

## Overview

Teams Chat Exporter is a Power Platform solution by [novalutions](https://www.novalutions.de) that lets employees export their own Microsoft Teams chats and channel conversations, including attachments and embedded images. Each export is saved as an interactive HTML file with full-text search and a date filter in the user's own OneDrive.

The solution is installed by the customer in their own Microsoft 365 tenant. All data stays in the tenant; novalutions has no access to chats or exports. Once downloaded, an exported HTML file can be opened and archived without a Microsoft 365 account.

Current version: **1.3.3.5**

## Architecture

The solution is delivered as the managed Power Platform solution "TeamsChatExporter" by the publisher "novalutions" and consists of the following components:

| Component | Type | Purpose |
|---|---|---|
| Teams Chat Exporter | Canvas app (embedded in Teams) | User interface: chat list with filter by chat type and search, multi-select, selection of team, channel and conversations, link to previous exports. UI available in German and English. |
| Teams Chat Exporter – Get Chats | Power Automate flow | Loads the user's chat list page by page via Microsoft Graph and returns the link to the export folder in OneDrive. |
| Teams Exporter – InvokedWorkflowChat | Power Automate flow | Exports a 1:1, group or meeting chat. Started once for each selected chat. |
| Teams Exporter – InvokedWorkflowConversation | Power Automate flow | Exports one or more conversations (threads) of a Teams channel. |
| 4 connection references | Office 365 Users, OneDrive for Business, SharePoint, Microsoft Teams | Access to Microsoft Graph, files and Teams notifications. |

Export flow:

1. The user selects one or more chats or channel conversations in the app and starts the export.
2. The app starts one flow per chat (or per channel export). The export continues in the background; the app can be closed.
3. The flow reads messages, members, attachments and embedded images via Microsoft Graph, builds the HTML file and saves everything in the user's OneDrive.
4. The user receives a Teams message from Flow bot with a link to the export folder.

All flows run with the connections of the calling user. No app registration in Microsoft Entra ID is required, and no data is transferred to systems outside the tenant.

> **Note:** The earlier workflows that were started directly from a chat's message menu are no longer supported since version 1.2.0.0 (deprecated by Microsoft since May 1, 2025). Exports are started exclusively from the app.

## Prerequisites

### For installation (administrator)

| Prerequisite | Details |
|---|---|
| Microsoft 365 license | At least Microsoft 365 Business Basic (or higher, e.g. Business Standard/Premium, E3, E5) including Teams, OneDrive and SharePoint. **Microsoft Teams Essentials is not supported.** |
| Power Platform environment | An environment that solutions can be imported into (with Dataverse), e.g. the tenant's default environment. |
| Role in the environment | System Administrator, System Customizer or Environment Admin. This role is only needed for import and sharing, not for using the solution afterwards. |
| Teams administrator | Only required for organization-wide deployment via the Teams admin center (Teams Administrator or Global Administrator role). |
| Custom Teams apps | The org-wide app settings in the Teams admin center must allow custom apps. |
| Connectors | The Office 365 Users, OneDrive for Business, SharePoint and Microsoft Teams connectors must not be blocked or separated from each other by a data loss prevention (DLP) policy of the environment. |
| Solution file | You receive the solution file (ZIP) from novalutions after purchase or for the free trial. |

Only standard connectors are used. No Power Apps or Power Automate premium license is required.

### For use (employees)

- Microsoft 365 license with Teams and OneDrive (e.g. Business Basic)
- The app and the three flows shared by the administrator (see [Installation](#installation))
- One-time consent to the connections in the app on first launch
- A current browser to view the export. For full styling, the HTML file loads a stylesheet from the internet (cdnjs.cloudflare.com). Without an internet connection all content remains readable, only the layout is simplified.

By purchasing, the customer confirms that they hold the necessary rights and comply with internal and legal requirements. For details, see the [General Terms and Conditions of Business and Use](https://www.novalutions.de/wp-content/uploads/2026/03/Allgemeine-Geschaefts-und-Nutzungsbedingungen-Teams-Chat-Exporter.pdf) (German).

## Installation

All steps are performed with the administrator account that imports the solution. A video guide is available under [Resources](#resources).

### Step 1: Import the solution

1. Open [make.powerapps.com](https://make.powerapps.com) and select the target environment in the top right corner.
2. Select "Solutions" in the left menu, then click "Import solution" at the top.
3. Click "Browse", select the solution file (ZIP) and click "Next".
4. On the "Connections" page, a connection is required for each of the four connectors (Office 365 Users, OneDrive for Business, SharePoint, Microsoft Teams). Select an existing connection for each, or create one via "New connection" and sign in.
5. Click "Import" and wait for the success message. The import can take a few minutes.

### Step 2: Check the flows

1. Under "Solutions", open the "Teams Chat Exporter" solution.
2. Select "Cloud flows" on the left.
3. Make sure all three flows have the status "On". If a flow is turned off, select it and click "Turn on".

### Step 3: Share the flows with employees

Each of the three flows must be shared with users as "run only":

- Teams Chat Exporter – Get Chats
- Teams Exporter – InvokedWorkflowChat
- Teams Exporter – InvokedWorkflowConversation

For each flow:

1. Open [make.powerautomate.com](https://make.powerautomate.com) in the same environment and open the flow (via "Solutions" or "My flows").
2. On the details page, click "Edit" in the "Run only users" section.
3. Add the desired users or a group; for organization-wide sharing, e.g. a group containing all employees.
4. The connections are already configured in the solution so that the flow runs with each user's own connection. Leave this setting unchanged and click "Save".

### Step 4: Share the app

1. Open the "Apps" area in [make.powerapps.com](https://make.powerapps.com).
2. Click the three dots next to "Teams Chat Exporter", then click "Share".
3. Add the same users or groups as for the flows (without co-owner rights) and click "Share".

### Step 5: Deploy the app in Teams

Choose one of the two options.

**Option A: For individual users (e.g. testing)**

1. In [make.powerapps.com](https://make.powerapps.com) under "Apps", click the three dots next to "Teams Chat Exporter", then "Add to Teams".
2. Click "Add to Teams" in the dialog. Teams opens and the app can be added for your own account.

**Option B: Organization-wide via the Teams admin center (recommended)**

1. In [make.powerapps.com](https://make.powerapps.com) under "Apps", click the three dots next to "Teams Chat Exporter", then "Add to Teams", and select "Download app" in the dialog. You receive the app package as a ZIP file.
2. Sign in to the [Teams admin center](https://admin.teams.microsoft.com) and open "Teams apps", then "Manage apps".
3. Click "Actions", then "Upload new app", and select the ZIP file. It can take up to one hour until the app appears in the list.
4. Under "Teams apps", open "Setup policies" and select the desired policy, e.g. "Global (Org-wide default)".
5. In the "Pinned apps" section, click "Add apps", search for "Teams Chat Exporter", click "Add" and then "Save".

> **Note:** Changes to setup policies can take up to 48 hours to take effect for all users.

### Step 6: First launch by the user

1. The user opens "Teams Chat Exporter" in the Teams left bar. If the app is not pinned, it can be found via the three dots or "Apps".
2. On first launch, the app asks for consent to the connections (Office 365 Users, OneDrive for Business, SharePoint, Microsoft Teams). Confirm with "Allow".
3. The app then loads the chat list. The first load may take a moment depending on the number of chats; afterwards the list is cached locally.

### Updates

New versions are also installed into the same environment via "Solutions" and "Import solution". The existing solution is upgraded; connections and sharing settings are usually retained. After the update, make sure all three flows are turned on (step 2).

### Uninstallation

Delete the "Teams Chat Exporter" solution under "Solutions" and remove the app in the Teams admin center. Existing exports remain in the users' OneDrive.

## Usage

**Export chats**

1. In the chats area, filter the list by type (1:1, group, meeting chats) if needed, or narrow it down using the search.
2. Select one or more chats and start the export.

**Export channel conversations**

1. In the channels area, select a team and a channel.
2. Select individual conversations or all conversations of the channel and start the export.

**Open the export**

1. When the export is complete, Flow bot sends a Teams message with the link "Open download".
2. Open the link and save the folder as a ZIP file via "Download".
3. Extract the ZIP file and double-click the HTML file to open it in the browser. The HTML file and the "Dateien" subfolder must stay together so that images and attachments remain linked.

The "Open exported chats" button in the app takes you to the "TeamsChatExport" folder in your own OneDrive at any time.

**Export duration:** About 1 to 2 minutes per 100 messages. Chats with several thousand messages can take up to an hour. The export runs in the background; Teams and the computer do not need to stay open.

## Supported Export Types

| Type | Description |
|---|---|
| 1:1 chats | Chats between two people |
| Group chats | Chats with multiple participants, with or without a topic |
| Meeting chats | Chats belonging to Teams meetings. Can be shown or hidden in the app. |
| Channel conversations | Individual or all conversations (threads) of a channel, each with the starting message and all replies |
| Chats with external users | Chats with people from other organizations, unless their organization prevents it |
| Bulk export | Multiple chats or multiple channel conversations in one run |

Hidden chats are not shown in the chat list.

## Supported Content Types

| Content | Chat export | Channel export | Notes |
|---|:---:|:---:|---|
| Text messages with formatting | ✅ | ✅ | Formatting, links and mentions are preserved. Links are marked with 🔗. |
| Sender and timestamp | ✅ | ✅ | Per message |
| Conversation subject | – | ✅ | Shown as the heading of the conversation |
| Quoted messages (replies) | ✅ | – | With sender and preview; clicking jumps to the original message |
| Embedded images (e.g. pasted screenshots) | ✅ | ✅ | Downloaded, stored locally and shown in the export (since version 1.3.0.0) |
| File attachments | ✅ | ✅ | Files from OneDrive or SharePoint shared in the chat or channel are downloaded with their original names and linked. Image attachments are displayed directly. |
| System events | ✅ | – | Members joined, added, left; call started, ended; chat renamed |
| Reactions (emoji reactions) | – | – | Not exported |
| Deleted messages | – | – | Cannot be exported |
| Online GIFs | – | – | Not downloaded |
| Adaptive Cards, Loop components, polls | – | – | Not rendered as content |
| Meeting recordings and transcripts | – | – | Not part of the export |

## Export Structure

### Storage in OneDrive

All exports are stored in the user's OneDrive in the `TeamsChatExport` folder. Each export gets its own subfolder with the export date.

```
TeamsChatExport/
├── 2026-10-12 - Project Alpha/                 ← Chat export
│   ├── Export - 2026-10-12 - Project Alpha.html
│   ├── metadata.txt
│   └── Dateien/
│       ├── Quote.pdf                           ← Attachment (original name)
│       └── 0-weu-d1-3f2a….png                  ← Embedded image
└── 2026-10-12- Sales - General/                ← Channel export (Team - Channel)
    ├── Export-2026-10-12.html
    └── Dateien/
        └── …
```

- **Chat export folder name:** Export date and chat name. The chat name is the chat's topic or, if none is set, the participants' names (truncated to 125 characters). Characters that are not allowed in file names are removed.
- **Channel export folder name:** Export date, team name and channel name. All conversations selected in one run are combined into one HTML file.
- **metadata.txt** (chat export only): Chat creation date, link to the chat in Teams, number of participants and number of messages.
- **Dateien/** ("files"): Attachments with their original names, plus embedded images named with a unique identifier.
- **Sharing link:** A view link to the export folder is created that only works for people in your own organization. It is sent exclusively to the exporting user.

### HTML file layout

- Header with chat or channel name and export date
- Search box for full-text search across message content and sender names
- Date filter with start and end date
- Messages in Teams style: own messages on the right, others' messages on the left, each with sender and timestamp
- Embedded images; clicking opens the image in full size
- Linked attachments from the "Dateien" folder
- Quoted messages with a jump to the original message (chat export)
- Channel export: conversations grouped with subject, number of messages and time span, each collapsible
- Print view: the search and filter bar is hidden when printing or saving as PDF

## Permissions & Data Access

Users can only export content they have access to themselves:

- Chats they participate in
- Channel conversations in teams they are a member of
- Attachments they are allowed to access in SharePoint or OneDrive

All connections run in the context of the signed-in user (delegated permissions). The solution requires no app registration and no admin consent for additional Graph permissions.

### Connectors and interfaces used

| Connector | Actions used | Purpose |
|---|---|---|
| Microsoft Teams | HTTP request to Microsoft Graph, Get team, Get channel, Post message as Flow bot | Read chats, chat members, messages, channel conversations with replies and embedded images (`/me/chats`, `/me/chats/{id}/messages`, `/teams/{id}/channels/{id}/messages/{id}/replies`, `…/hostedContents`); notify the user |
| Office 365 Users | HTTP request to Microsoft Graph | Profile of the exporting user (`/users/{id}`) and number of active accounts for the license check (`/users/$count`) |
| SharePoint | Get file content using path | Download shared file attachments |
| OneDrive for Business | Create file, Create share link | Save HTML file, attachments and metadata.txt; link to the export folder |

## Known Limitations

- **Deleted messages** cannot be exported.
- **Online GIFs** are not downloaded.
- **Reactions** to messages are not exported.
- **Error 403 for images or attachments:** Individual items can be blocked by the tenant's security, access or compliance policies (see [Troubleshooting](#troubleshooting)).
- **Chat list:** The app shows up to 1,250 chats per user.
- **Channel export:** Up to 1,250 replies are exported per conversation. System events are not shown in channel exports.
- **Chats with external users:** Only possible if the external organization does not restrict access.
- **Display:** For the full layout, the HTML file needs an internet connection when opened (see [Prerequisites](#prerequisites)).

## Troubleshooting

### Images or attachments are missing (error 403)

A 403 error means Microsoft 365 denied access to an item. This is almost always caused by policies in the tenant. Check these three areas:

1. **Conditional Access:** Policies in Microsoft Entra ID that restrict access to Microsoft Graph or SharePoint, e.g. by device, location or app.
2. **Cross-tenant access (B2B):** Content shared by guests or external users is stored in the other organization's tenant. Its settings can prevent access.
3. **Microsoft Purview:** DLP policies or sensitivity labels can block downloading certain files.

### The app does not appear in Teams

- Allow for waiting time: up to one hour after upload, up to 48 hours after changing a setup policy.
- Check that the user is assigned to the correct setup policy.
- Check that custom apps are allowed in the org-wide app settings and that the app has the status "Allowed" under "Manage apps".

### The app reports an error while loading or exporting

- Check that the app and all three flows are shared with the user (Installation, steps 3 and 4).
- Check that all three flows are turned on (Installation, step 2).
- Check whether a DLP policy of the environment blocks one of the four connectors.
- Errors during an export are reported by Flow bot with an error description via Teams message.

## Licensing

Teams Chat Exporter is offered as a one-time license with no recurring costs:

| Edition | Active accounts | Included services |
|---|---|---|
| S | 1–49 | 15 minutes of installation support |
| L | 50–250 | 15 minutes of installation support |
| Enterprise | 250+ | 60 minutes of installation support (remote), support mailbox, regular updates |

Active user accounts in the tenant are counted, excluding guest accounts. Current prices are listed on the [product page](https://www.novalutions.de/teams-chats-exportieren/) (German). A free trial version is available on request.

With every export, the solution checks whether the number of active accounts is covered by the license. If not, the user receives a notice via Teams message and the export file is marked as an evaluation version. In this case, please contact [teamschatexporter@novalutions.de](mailto:teamschatexporter@novalutions.de).

## Customization & Support

Customers are not permitted to modify the solution themselves (see [Terms](https://www.novalutions.de/wp-content/uploads/2026/03/Allgemeine-Geschaefts-und-Nutzungsbedingungen-Teams-Chat-Exporter.pdf)). Customization requests can be submitted as a support request.

Support beyond the included installation support is billed by effort at an hourly rate of €140 (excl. VAT). The response time is usually 2 to 3 business days.

**Contact:** [teamschatexporter@novalutions.de](mailto:teamschatexporter@novalutions.de) · novalutions UG (haftungsbeschränkt), Cologne, Germany

## Version History

| Version | Change |
|---|---|
| 1.3.3.5 | Current version |
| 1.3.0.0 | Export of embedded images (hosted contents) |
| 1.2.0.0 | Workflows started directly from the chat removed (deprecated by Microsoft since May 1, 2025); exports exclusively via the app |
| 1.1.0.16 | "Get Teams" flow removed |

## Resources

### Videos

**Installation guide**

[![Teams Chat Exporter Installation](https://img.youtube.com/vi/aTV7nPn8j9I/0.jpg)](https://www.youtube.com/watch?v=aTV7nPn8j9I)

**Product demo**

[![Teams Chat Exporter Demo](https://img.youtube.com/vi/1Y-eobVuMUk/0.jpg)](https://www.youtube.com/watch?v=1Y-eobVuMUk)

**Product presentation**

[![Teams Chat Exporter Presentation](https://img.youtube.com/vi/ANSKcxP64bU/0.jpg)](https://www.youtube.com/watch?v=ANSKcxP64bU)

### Links

- [Product page](https://www.novalutions.de/teams-chats-exportieren/) – features, pricing, trial (German)
- [Installation & sharing with screenshots](https://www.novalutions.de/teams-chat-exporter-freigabe/) – supplementary illustrated guide (German)
- [General Terms and Conditions of Business and Use](https://www.novalutions.de/wp-content/uploads/2026/03/Allgemeine-Geschaefts-und-Nutzungsbedingungen-Teams-Chat-Exporter.pdf) (German)
- [Privacy policy](https://www.novalutions.de/datenschutzerklaerung/) (German)
- [Contact by email](mailto:teamschatexporter@novalutions.de)
