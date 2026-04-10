# Teams Chat Exporter - Technical Documentation

## Overview

The Teams Chat Exporter is a Power Platform solution that enables users to export Microsoft Teams chats and channel conversations. The solution consists of a Power App and three Power Automate workflows. Exported content is saved as interactive HTML files in the user's OneDrive, providing search, filter, and navigation capabilities. After export, there are no cloud connections or dependencies on the existing Microsoft 365 installation.

## Architecture

The solution is composed of the following components:

- **1 Power App** – User interface for chat selection and export control, embedded in Microsoft Teams
- **3 Power Automate workflows:**
  - **GetChats** – Retrieves the user's chat list with pagination via the Microsoft Graph API
  - **Chat Export** – Exports a selected 1:1, group, or meeting chat
  - **Channel Export** – Exports one or more conversation threads from a Teams channel

> **Note:** The workflows that were invoked directly from within a chat have been deprecated since May 1, 2025, and are no longer supported. Export is performed exclusively via the Power App.

## Prerequisites

The following prerequisites are required to install the Teams Chat Exporter:

- **Administrative permissions:** System Administrator, System Customizer, or Environment Admin
- **License:** At least one Microsoft 365 Business Basic license (Teams Essentials is not supported)
- **Environment:** Microsoft Power Platform environment in the tenant

Without these roles, export is not possible. By purchasing, the user confirms that they have the necessary permissions and comply with internal and legal requirements. For details, please refer to our [General Terms and Conditions (GTC)](https://www.novalutions.de/wp-content/uploads/2026/03/Allgemeine-Geschaefts-und-Nutzungsbedingungen-Teams-Chat-Exporter.pdf).

## Installation & Deployment

### Initial Installation

Installation is performed by importing the solution into the Power Platform environment. A step-by-step guide is available in our [video tutorial](https://www.youtube.com/watch?v=aTV7nPn8j9I) and in the [written installation guide](https://www.novalutions.de/teams-chat-exporter-freigabe/).

### Workflow Sharing

After the initial installation, the workflows must be shared with employees:

1. Navigate to [make.powerautomate.com](https://make.powerautomate.com) using the account that imported the solution
2. Select "My Flows"
3. Open each workflow and edit the permissions under "Run only users"
4. Grant access to individual users or groups (e.g., "All Company" for organization-wide access)

### Organization-Wide Deployment via Teams Admin Center

For automatic deployment to all or selected users:

1. Navigate to the Power Apps Maker Portal → select the Teams Chat Exporter → Share → Add to Teams
2. Download the app ZIP file
3. Sign in to the Teams Admin Center → Teams apps → Manage apps
4. Upload the ZIP file as a new app
5. Configure the app setup policy: Add the Teams Chat Exporter as a pinned app

> **Note:** It can take up to 48 hours for app setup policy changes to take effect for end users.

## Supported Export Types

| Type | Description |
|------|-------------|
| 1:1 Chats | Direct chats between two people |
| Group Chats | Chats with multiple participants |
| Meeting Chats | Chats created in the context of a Teams meeting (can be toggled on/off) |
| Channel Conversations | Thread-based conversations from Teams channels including all replies |
| External Chats | Chats with external participants, unless their organization explicitly blocks this |
| Bulk Export | Multiple chats and channels can be exported simultaneously in a single operation |

## Supported Content Types

### Messages
- Text messages with full formatting
- Inline images (hosted content) – automatically downloaded and embedded in the export (since version 1.3.0.0)
- File attachments from SharePoint – all files visible under the "Shared" tab of the respective chat
- Message references (quoted messages)

### System Events
The export recognizes and renders the following Teams system events:
- Members joined / added
- Members left
- Call started / ended
- Chat renamed

### Attachments
- Automatic download of SharePoint-hosted file attachments
- Automatic download of inline hosted content (embedded images)
- Attachments are stored in a "Dateien" (Files) subfolder and linked in the HTML export
- Original file names are preserved

### Known Limitations
- Certain images may fail to export with error code 403 (Forbidden) – this is caused by tenant-specific security, access, or compliance policies
- Deleted messages can only be exported if they were previously saved
- Online GIFs are not downloaded as files

## User Permissions & Data Access

Users can only export content they have access to:
- Their own chat conversations in which they participate
- Channel conversations in Teams where they are a member
- Hidden chats are automatically filtered out

### Required Connectors & API Permissions

| Connector | Permissions Used | Purpose |
|-----------|-----------------|---------|
| Microsoft Teams | Chat.Read, ChannelMessage.Read.All | Reading chat messages, channel threads, and member lists via the Graph API |
| SharePoint Online | Files.ReadWrite.All | Downloading file attachments stored in SharePoint |
| OneDrive for Business | Files.ReadWrite | Storing HTML exports and downloaded attachments |
| Office 365 Users | User.Read.All | Resolving user profiles and display names |

All connectors use **invoker permissions** – the workflow runs in the context of the signed-in user and only accesses data available to that user.

## Feature Details

### Chat Listing (GetChats Workflow)
- Retrieves all of the user's chats via the Microsoft Graph API with automatic pagination
- Detection of chat type: 1:1 chat, group chat, or meeting chat
- Optional filter to include/exclude meeting chats
- Displays chat topic, participant count, and type in the Power App
- Hidden chats are excluded

### Chat Export Workflow
- Iterative retrieval of all messages with pagination (no message limits)
- Resolution of all chat participants with display names
- Recognition and processing of six system event types
- Automatic download and embedding of inline images (hosted content)
- Automatic download of SharePoint attachments
- Generation of a complete, interactive HTML export file
- Storage in the user's OneDrive with automatic folder structure
- Creation of an organization-wide sharing link
- User notification via Teams message with download link
- Multilingual support (German/English) for status messages

### Channel Export Workflow
- Export of one or more conversation threads from a channel
- Retrieval of the thread starter message and all replies
- Resolution of team and channel names for export metadata
- Attachment and hosted content processing identical to chat export
- Error handling for missing permissions
- Same HTML output and notification logic as chat export

### Export Duration
Duration depends on the volume of data. A chat with 100 messages takes approximately 1–2 minutes. Chats with thousands of messages can take up to one hour. The export runs in the background – the computer or app does not need to remain open. The user is automatically notified via Teams message once the export is complete.

## HTML Export

### Export File Features
- Responsive design with Tailwind CSS
- Full-text search across all messages
- Filter by date
- Filter by user
- Timestamp and user identification per message
- Direct links to downloaded attachments
- Embedded inline images

### OneDrive File Structure

```
TeamsChatExport/
├── YYYY-MM-DD - ChatName/
│   ├── Export - YYYY-MM-DD - ChatName.html
│   └── Dateien/
│       ├── document.docx
│       ├── image.jpg
│       └── ...
└── YYYY-MM-DD - TeamName/
    ├── Export - YYYY-MM-DD - TeamName.html
    └── Dateien/
        └── ...
```

## License Validation

The solution includes a built-in license check that compares the number of active users (excluding guest accounts) in the tenant against the licensed quantity. If the limit is exceeded, the user is notified and the export is marked as an evaluation.

## Customizations & Support

Individual customization of the Teams Chat Exporter is not permitted (see [GTC](https://www.novalutions.de/wp-content/uploads/2026/03/Allgemeine-Geschaefts-und-Nutzungsbedingungen-Teams-Chat-Exporter.pdf)). For custom requirements, a support request can be submitted. Individual support is billed at an hourly rate of €140. Enterprise customers receive regular, free updates with improvements and new features, as well as exclusive access to the support inbox.

## Resources & Further Information

### Video Tutorials

#### Installation Guide (current)
[![Teams Chat Exporter Installation](https://img.youtube.com/vi/aTV7nPn8j9I/0.jpg)](https://www.youtube.com/watch?v=aTV7nPn8j9I)

#### Product Presentation
[![Teams Chat Exporter Presentation](https://img.youtube.com/vi/ANSKcxP64bU/0.jpg)](https://www.youtube.com/watch?v=ANSKcxP64bU)

### More Information
- [Official Product Page](https://www.novalutions.de/teams-chats-exportieren/) – Detailed product information and pricing
- [Installation Guide & Sharing](https://www.novalutions.de/teams-chat-exporter-freigabe/) – Step-by-step setup and sharing instructions
- [General Terms and Conditions (GTC)](https://www.novalutions.de/wp-content/uploads/2026/03/Allgemeine-Geschaefts-und-Nutzungsbedingungen-Teams-Chat-Exporter.pdf) – Contract terms and usage rights
- [Support & Contact](https://www.novalutions.de/teams-chats-exportieren/#contact) – Direct contact with the support team
- [Free Demo](https://www.novalutions.de/teams-chats-exportieren/#demo) – Schedule a personal product demonstration

### Licensing & Pricing
The Teams Chat Exporter is available as a one-time license – no recurring costs or subscriptions. An optional installation service can be added. Enterprise licenses include 60 minutes of installation support (remote), regular updates, and access to the support inbox. For detailed pricing information, visit the [product page](https://www.novalutions.de/teams-chats-exportieren/). The [General Terms and Conditions](https://www.novalutions.de/wp-content/uploads/2026/03/Allgemeine-Geschaefts-und-Nutzungsbedingungen-Teams-Chat-Exporter.pdf) apply.
