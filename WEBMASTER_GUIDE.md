# IEEE-HKN Chapter Suggestion System
## Complete Technical Documentation for Webmaster

**Version:** 2.0
**Last Updated:** November 2025

---

## Table of Contents

1. [System Overview](#1-system-overview)
2. [Architecture & Components](#2-architecture--components)
3. [Configuration Guide](#3-configuration-guide)
4. [Setup & Installation](#4-setup--installation)
5. [User Workflow](#5-user-workflow)
6. [Privacy & Security](#6-privacy--security)
7. [Platform Integrations](#7-platform-integrations)
8. [Troubleshooting](#8-troubleshooting)
9. [Maintenance & Operations](#9-maintenance--operations)
10. [Testing & Diagnostics](#10-testing--diagnostics)
11. [Advanced Features](#11-advanced-features)
12. [FAQ](#12-faq)

---

## 1. System Overview

### 1.1 Purpose

The IEEE-HKN Chapter Suggestion System is an automated workflow that processes member feedback and suggestions submitted through Google Forms. The system routes submissions to Discord and Slack channels in real-time, enabling leadership to review, discuss, and act on community input efficiently.

### 1.2 Key Features

- **Automated Processing**: Instant posting to Discord and Slack when forms are submitted
- **Privacy Controls**: Support for anonymous submissions and leadership-only routing
- **Tracking System**: Unique IDs for every suggestion (format: `HKN-YYYY-XXXX`)
- **Priority Visualization**: Color-coded embeds based on urgency (1-10 scale)
- **Email Notifications**: Automatic confirmation emails for confidential submissions
- **Dual-Platform Support**: Simultaneous posting to Discord and Slack
- **Error Handling**: Comprehensive logging and automatic error notifications
- **Leadership Channel**: Separate routing for sensitive feedback

### 1.3 Technology Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Backend** | Google Apps Script (V8 Runtime) | Core automation engine |
| **Data Storage** | Google Sheets | Form responses and tracking |
| **Form Interface** | Google Forms | User submission interface |
| **Communication** | Discord Webhooks | Community channel posting |
| **Communication** | Slack Web API | Leadership channel posting |
| **Email** | Gmail API | Confirmation notifications |
| **Triggers** | Apps Script Event System | Automatic execution |

### 1.4 Data Flow

```
Member Submission (Google Form)
         ↓
Google Sheets (Data Storage)
         ↓
Apps Script Trigger (onFormSubmit)
         ↓
   Data Processing
    ↙        ↘
Discord     Slack
Webhook    Web API
    ↓          ↓
Community  Private
Channel    Channel
```

---

## 2. Architecture & Components

### 2.1 Core Modules

The system is organized into modular classes following object-oriented design principles:

#### **SecureConfig Class**
- **Purpose**: Manages all sensitive credentials and API keys
- **Storage**: Script Properties Service (encrypted by Google)
- **Methods**:
  - `getDiscordMainWebhook()` - Main channel webhook URL
  - `getDiscordLeadershipWebhook()` - Leadership channel webhook URL
  - `getSlackBotToken()` - Slack OAuth token
  - `getSlackChannelId()` - Target Slack channel
  - `getSlackErrorWebhook()` - Fallback error notifications

#### **DataValidator Class**
- **Purpose**: Input validation and sanitization
- **Key Functions**:
  - `validatePriority(priority)` - Ensures 1-10 range
  - `safeExtract(data, index, default)` - Safe field extraction
  - `validateSuggestionData(data)` - Required field validation
  - `processTimestamp(timestamp)` - Date parsing and normalization
  - `escapeSlackMarkdown(text)` - Prevents injection attacks

#### **SpreadsheetManager Class**
- **Purpose**: All spreadsheet operations
- **Key Functions**:
  - `connectToSpreadsheet()` - Establishes connection
  - `addTrackingId(sheet, row, trackingId)` - Appends tracking ID
  - `logToDebugSheet(action, details, status)` - Audit logging

#### **TrackingIdGenerator Class**
- **Purpose**: Generates unique suggestion identifiers
- **Format**: `HKN-YYYY-XXXX` (year + random 4-digit number)
- **Example**: `HKN-2025-7843`

#### **PrivacyHandler Class**
- **Purpose**: Manages anonymity preferences
- **Key Functions**:
  - `getDisplayName(preference, name)` - Determines visible name
  - `isLeadershipOnly(preference)` - Routes to private channel

#### **DiscordIntegration Class**
- **Purpose**: Discord webhook posting
- **Features**:
  - Rich embeds with color coding
  - Priority-based colors (Red/Orange/Maroon)
  - Separate leadership channel support
  - Reaction prompts for community engagement

#### **SlackIntegration Class**
- **Purpose**: Slack API posting
- **Features**:
  - Block Kit UI formatting
  - Message timestamp storage
  - Web API (not legacy webhooks)
  - Support for slash commands

#### **EmailNotifier Class**
- **Purpose**: Confirmation emails for leadership submissions
- **Triggers**: Only for "Contact Leadership" preference
- **Content**: Tracking ID and acknowledgment message

#### **ErrorNotifier Class**
- **Purpose**: Admin error notifications
- **Channels**: Both Discord and Slack
- **Triggers**: Processing failures, validation errors, API failures

### 2.2 Column Mapping

The system expects Google Forms to populate columns in this order:

| Index | Column | Form Field | Notes |
|-------|--------|------------|-------|
| 0 | A | Timestamp | Auto-generated by Google Forms |
| 1 | B | Email Address | Required for leadership submissions |
| 2 | C | Suggestion Text | Main proposal content |
| 3 | D | Category | Area of improvement |
| 4 | E | Importance | Impact explanation |
| 5 | F | Implementation | How it could work |
| 6 | G | Priority (1-10) | User-assessed urgency |
| 7 | H | Member Status | Active/Alumni/Other |
| 8 | I | Submitter Name | For non-anonymous posts |
| 9 | J | Privacy Preference | Anonymous/Name/Leadership |
| 10 | K | Tracking ID | Auto-generated by script |

**Important**: Column K (Tracking ID) must be added manually to your spreadsheet header row.

### 2.3 Privacy Preference Options

| Preference | Display Behavior | Routing |
|------------|------------------|---------|
| "Post with my name in Discord" | Shows actual name | Public channels |
| "Post anonymously" | "Anonymous Member" | Public channels |
| "Contact Leadership" | Name visible to leadership only | Leadership channel only |

---

## 3. Configuration Guide

### 3.1 Required Configuration

#### **Step 1: Spreadsheet ID**

Locate your Google Sheets spreadsheet ID:

1. Open your form response spreadsheet
2. Look at the URL: `https://docs.google.com/spreadsheets/d/[SPREADSHEET_ID]/edit`
3. Copy the ID between `/d/` and `/edit`
4. Replace in [Suggestions.js:13](Suggestions.js#L13):

```javascript
const SPREADSHEET_ID = 'YOUR_ACTUAL_SPREADSHEET_ID';
```

#### **Step 2: Platform Toggles**

Enable or disable platforms in [Suggestions.js:15-17](Suggestions.js#L15-L17):

```javascript
const ENABLE_DISCORD = true;  // Set to false to disable
const ENABLE_SLACK = true;    // Set to false to disable
```

#### **Step 3: Priority Thresholds**

Adjust color-coding thresholds in [Suggestions.js:19-21](Suggestions.js#L19-L21):

```javascript
const PRIORITY_HIGH_THRESHOLD = 8;     // Red: ≥8
const PRIORITY_MEDIUM_THRESHOLD = 5;   // Orange: 5-7
// Below 5: Maroon (standard)
```

### 3.2 Credential Storage

All sensitive credentials are stored in Script Properties (encrypted by Google). To configure:

#### **Discord Webhooks**

1. Navigate to your Discord server
2. Go to Server Settings → Integrations → Webhooks
3. Create two webhooks:
   - **Main Channel**: For public suggestions
   - **Leadership Channel**: For confidential feedback
4. Store the URLs:

```javascript
// Run once in Apps Script editor:
function setupDiscord() {
  const scriptProperties = PropertiesService.getScriptProperties();
  scriptProperties.setProperty('DISCORD_WEBHOOK_URL', 'https://discord.com/api/webhooks/...');
  scriptProperties.setProperty('DISCORD_LEADERSHIP_WEBHOOK_URL', 'https://discord.com/api/webhooks/...');
}
```

#### **Slack Credentials**

1. Create a Slack App at [api.slack.com/apps](https://api.slack.com/apps)
2. Enable OAuth and add bot token scopes:
   - `chat:write`
   - `chat:write.public`
3. Install app to workspace and copy Bot User OAuth Token
4. Find your target channel ID (right-click channel → View Details)
5. Store credentials:

```javascript
// Run once in Apps Script editor:
function setupSlack() {
  const scriptProperties = PropertiesService.getScriptProperties();
  scriptProperties.setProperty('SLACK_BOT_TOKEN', 'xoxb-...');
  scriptProperties.setProperty('SLACK_CHANNEL_ID', 'C...');
}
```

### 3.3 Security Best Practices

1. **Never commit credentials to version control**
2. **Use Script Properties for all sensitive data**
3. **Restrict spreadsheet access to leadership only**
4. **Enable 2FA on Google account managing the script**
5. **Audit Script Properties quarterly** using `runSecurityAudit()`
6. **Remove hardcoded values** from `SecureConfig` after initial setup

---

## 4. Setup & Installation

### 4.1 Prerequisites

- Google account with access to IEEE-HKN organizational workspace
- Editor access to the form response spreadsheet
- Admin permissions in Discord server
- Admin permissions in Slack workspace
- Basic understanding of Google Apps Script

### 4.2 Initial Setup (Step-by-Step)

#### **1. Prepare the Spreadsheet**

```
a. Open your Google Form's response spreadsheet
b. Ensure Column K exists (add if needed) with header "Tracking ID"
c. Note the spreadsheet ID from the URL
d. Share the spreadsheet with all webmasters (Edit access)
```

#### **2. Open Apps Script Editor**

```
a. In spreadsheet: Extensions → Apps Script
b. Delete any existing code
c. Copy the entire Suggestions.js content
d. Paste into the editor
e. Name the project: "IEEE-HKN Suggestion System"
f. Save (Ctrl/Cmd + S)
```

#### **3. Configure Spreadsheet ID**

```javascript
// Line 13 in Suggestions.js
const SPREADSHEET_ID = 'PASTE_YOUR_ID_HERE';
```

#### **4. Run Quick Setup**

```
a. In Apps Script editor, select "quickSetup" from function dropdown
b. Click Run (▶ button)
c. Authorize the script when prompted:
   - Review permissions
   - Click "Advanced" if warned
   - Select "Go to IEEE-HKN Suggestion System (unsafe)"
   - Click "Allow"
d. Check execution log (View → Logs) for results
```

The `quickSetup()` function will:
- ✅ Verify spreadsheet connection
- ✅ Install form submit trigger
- ✅ Test Discord webhook
- ✅ Test Slack API
- ✅ Create Debug sheet if needed

#### **5. Configure Credentials**

Run these functions one at a time:

```javascript
// In Apps Script editor:

// For Discord
function setupDiscordWebhooks() {
  const scriptProperties = PropertiesService.getScriptProperties();
  scriptProperties.setProperty('DISCORD_WEBHOOK_URL', 'YOUR_MAIN_WEBHOOK');
  scriptProperties.setProperty('DISCORD_LEADERSHIP_WEBHOOK_URL', 'YOUR_LEADERSHIP_WEBHOOK');
}

// For Slack
setupSlackCredentials(); // Built-in function
```

#### **6. Verify Installation**

```javascript
// Run this function to check everything:
runFullDiagnostics();
```

Expected output:
```
✅ Triggers: OK
✅ Webhooks: OK
✅ Spreadsheet: Connected
✅ Credentials: Stored
```

### 4.3 Test Submission

Submit a test form entry:

```javascript
// Run in Apps Script editor:
testFormSubmission();
```

Check:
1. Discord channel for new post
2. Slack channel for new post
3. Spreadsheet column K for tracking ID
4. Debug sheet for log entries

### 4.4 Deployment Checklist

- [ ] Spreadsheet ID configured
- [ ] Column K added and labeled
- [ ] Form submit trigger installed
- [ ] Discord main webhook stored in Script Properties
- [ ] Discord leadership webhook stored in Script Properties
- [ ] Slack bot token stored in Script Properties
- [ ] Slack channel ID stored in Script Properties
- [ ] Test submission successful
- [ ] Error notifications working
- [ ] Debug sheet created
- [ ] Hardcoded credentials removed from code

---

## 5. User Workflow

### 5.1 Submission Process

**From Member Perspective:**

1. Member accesses Google Form (public link or QR code)
2. Fills out required fields:
   - Email address
   - Suggestion text
   - Category selection
   - Importance explanation
   - Implementation ideas (optional)
   - Priority rating (1-10)
   - Member status
   - Name (if not anonymous)
   - Privacy preference
3. Clicks Submit
4. Receives confirmation page

**Behind the Scenes:**

```
Form Submit
    ↓
Google Sheets (row added)
    ↓
Apps Script Trigger Fires
    ↓
onFormSubmit() executes
    ↓
Generate Tracking ID (HKN-2025-XXXX)
    ↓
Validate Data
    ↓
Check Privacy Preference
    ↓
Route to Channels:
  - Public → Discord Main + Slack
  - Leadership → Discord Leadership Only
    ↓
Send Email (if leadership-only)
    ↓
Log to Debug Sheet
```

### 5.2 Discord Message Format

**Public Suggestion:**
```
🗳️ NEW SUGGESTION - HKN-2025-1234

ID: HKN-2025-1234 | Category: Academic Support
Priority: 8/10 | Submitted by: John Smith

📝 Proposal
[Suggestion text here...]

📈 Impact
[Importance explanation...]

🔧 Implementation
[Implementation details...]

💬 Community Support
👍 Support | ❤️ Love this | 🔥 High Priority | 💡 Great Idea

✅ Leadership Acceptance
(indicates that leadership has accepted for review)

Tracking ID: HKN-2025-1234 • 11/5/2025 • Priority: 🔴
[Form Link] | [Policy Document]
```

**Color Coding:**
- 🔴 Red: Priority ≥ 8 (High)
- 🟠 Orange: Priority 5-7 (Medium)
- 🟤 Maroon: Priority < 5 (Standard)

### 5.3 Slack Message Format

Slack messages use Block Kit for interactive elements:

```
━━━━━━━━━━━━━━━━━━━━━━━━━
🗳️ NEW SUGGESTION - HKN-2025-1234
━━━━━━━━━━━━━━━━━━━━━━━━━

ID: HKN-2025-1234        Category: Academic Support
Priority: 8/10 🔴        Submitted by: John Smith

────────────────────────

📝 Proposal
[Suggestion text here...]

📈 Impact
[Importance explanation...]

────────────────────────

✅ Leadership Acceptance (indicates review acceptance)

Tracking ID: HKN-2025-1234 • 11/5/2025 • Priority: 🔴

Suggestions/Feedback Form | Policy Document
```

### 5.4 Email Confirmation (Leadership-Only)

When a member selects "Contact Leadership", they receive:

```
Subject: Suggestion Confirmation - HKN-2025-1234

Dear [Member Name],

Your confidential suggestion HKN-2025-1234 has been submitted
to IEEE-HKN leadership for private review. You can reference
this ID when following up.

Best regards,
IEEE-HKN Chapter Suggestion System
```

---

## 6. Privacy & Security

### 6.1 Privacy Controls

The system implements three privacy levels:

#### **Level 1: Public with Name**
- **User Selection**: "Post with my name in Discord"
- **Behavior**: Full name displayed in public channels
- **Routing**: Discord main + Slack
- **Use Case**: Members comfortable with public attribution

#### **Level 2: Anonymous**
- **User Selection**: "Post anonymously"
- **Behavior**: Displays "Anonymous Member"
- **Routing**: Discord main + Slack
- **Use Case**: Members wanting to contribute without identification

#### **Level 3: Leadership Confidential**
- **User Selection**: "Contact Leadership"
- **Behavior**:
  - Name visible only to leadership
  - Email included for follow-up
  - Tracking ID emailed to submitter
- **Routing**: Discord leadership channel ONLY (not public)
- **Use Case**: Sensitive feedback, complaints, confidential suggestions

### 6.2 Data Access

| Data | Stored Where | Access Level |
|------|--------------|--------------|
| Form Responses | Google Sheets | Webmaster + Leadership |
| Tracking IDs | Script Properties | Script only |
| Slack Message IDs | Script Properties | Script only |
| Discord Webhooks | Script Properties | Script only |
| Slack Tokens | Script Properties | Script only |
| Debug Logs | Debug Sheet | Webmaster + Leadership |

### 6.3 Security Features

1. **No Hardcoded Credentials**: All sensitive data in Script Properties
2. **Input Sanitization**: Escapes markdown and special characters
3. **Error Isolation**: Try-catch blocks prevent data exposure
4. **Audit Logging**: All operations logged to Debug sheet
5. **Least Privilege**: Script requests minimum necessary permissions
6. **Webhook Validation**: Response codes checked before marking success

### 6.4 GDPR/Privacy Compliance

- Members opt-in by submitting the form
- Anonymous option available
- Email only collected if member provides it
- Leadership can delete suggestions on request
- No third-party analytics or tracking
- Data retained only in IEEE-HKN Google Workspace

### 6.5 Incident Response

**If credentials are compromised:**

1. Immediately run:
   ```javascript
   function emergencyLockdown() {
     const scriptProperties = PropertiesService.getScriptProperties();
     scriptProperties.deleteAllProperties();
     console.log('All credentials cleared');
   }
   ```

2. Regenerate all webhooks and tokens
3. Re-run credential setup functions
4. Audit all recent form submissions
5. Notify leadership of potential data exposure

---

## 7. Platform Integrations

### 7.1 Discord Integration

#### **Webhook Setup**

1. **Main Channel Webhook:**
   ```
   Server Settings → Integrations → Webhooks → New Webhook
   Name: Suggesty
   Channel: #suggestions (or your public channel)
   Copy Webhook URL
   ```

2. **Leadership Channel Webhook:**
   ```
   Server Settings → Integrations → Webhooks → New Webhook
   Name: Suggesty for Leaders
   Channel: #leadership-suggestions (private channel)
   Copy Webhook URL
   ```

3. **Store in Script:**
   ```javascript
   setDiscordLeadershipWebhook('YOUR_LEADERSHIP_WEBHOOK_URL');
   ```

#### **Webhook Structure**

Discord uses rich embeds (JSON format):

```javascript
{
  username: "Suggesty",
  embeds: [{
    author: { name: "IEEE-HKN Chapter Suggestion System" },
    title: "🗳️ NEW SUGGESTION - HKN-2025-1234",
    description: "ID: ... | Category: ... | Priority: ...",
    color: 0xD00000,  // Red for high priority
    fields: [
      { name: "📝 Proposal", value: "...", inline: false },
      { name: "📈 Impact", value: "...", inline: false }
    ],
    footer: { text: "Tracking ID: HKN-2025-1234" },
    timestamp: "2025-11-05T12:00:00.000Z"
  }],
  content: "[Form Link] | [Policy Document]"
}
```

#### **Color Scheme**

Colors are determined by `DiscordIntegration.getPriorityColor()`:

```javascript
Priority 8-10  → 0xD00000 (Red)
Priority 5-7   → 0xFFA500 (Orange)
Priority 1-4   → 0x8C1D40 (ASU Maroon)
```

### 7.2 Slack Integration

#### **App Configuration**

1. **Create Slack App:**
   - Go to [api.slack.com/apps](https://api.slack.com/apps)
   - Click "Create New App" → "From scratch"
   - Name: "IEEE-HKN Suggesty"
   - Workspace: Your IEEE-HKN workspace

2. **Configure OAuth Scopes:**
   ```
   OAuth & Permissions → Scopes → Bot Token Scopes:
   - chat:write
   - chat:write.public
   - channels:read
   - groups:read
   ```

3. **Install to Workspace:**
   ```
   Install App → Install to Workspace → Allow
   Copy the "Bot User OAuth Token" (starts with xoxb-)
   ```

4. **Invite Bot to Channel:**
   ```
   In Slack: /invite @IEEE-HKN Suggesty
   Or: Right-click channel → View Details → Integrations → Add Apps
   ```

#### **API Endpoints Used**

| Endpoint | Purpose | Method |
|----------|---------|--------|
| `chat.postMessage` | Send new suggestions | POST |
| `chat.delete` | Remove suggestions | POST |
| `auth.test` | Verify credentials | POST |

#### **Message Timestamp Storage**

Slack message IDs are stored for future operations:

```javascript
// After posting to Slack:
scriptProperties.setProperty(`slack_msg_${suggestionID}`, messageTimestamp);

// Later, to delete:
const messageTs = scriptProperties.getProperty(`slack_msg_${suggestionID}`);
```

#### **Block Kit Structure**

Slack uses Block Kit (JSON format):

```javascript
{
  channel: "C09F4GYL8E9",
  blocks: [
    { type: "header", text: { type: "plain_text", text: "🗳️ NEW SUGGESTION" }},
    { type: "section", fields: [
      { type: "mrkdwn", text: "*ID:* HKN-2025-1234" },
      { type: "mrkdwn", text: "*Priority:* 8/10 🔴" }
    ]},
    { type: "divider" },
    { type: "section", text: { type: "mrkdwn", text: "*📝 Proposal*\n..." }}
  ],
  text: "New suggestion: HKN-2025-1234"  // Fallback for notifications
}
```

### 7.3 Gmail Integration

#### **Automatic Configuration**

Gmail API is automatically available in Apps Script. No additional setup needed.

#### **Email Trigger Conditions**

Emails are sent ONLY when:
1. Privacy preference is "Contact Leadership"
2. Valid email address provided
3. Submitter name is not empty/N/A

#### **Email Template**

Located in `EmailNotifier.sendTrackingIdConfirmation()` [Suggestions.js:674-720](Suggestions.js#L674-L720):

```javascript
const subject = `Suggestion Confirmation - ${trackingID}`;
const body = `${greeting}

Your confidential suggestion ${trackingID} has been submitted to
IEEE-HKN leadership for private review. You can reference this ID
when following up.

Best regards,
IEEE-HKN Chapter Suggestion System`;
```

To customize, edit the template in the `EmailNotifier` class.

---

## 8. Troubleshooting

### 8.1 Common Issues

#### **Issue: Script Not Running on Form Submit**

**Symptoms:**
- Form submitted but no Discord/Slack post
- No tracking ID in spreadsheet
- No execution log in Apps Script

**Diagnosis:**
```javascript
checkTriggers();  // Run this function
```

**Solution:**
```javascript
installTrigger();  // Reinstalls the trigger
```

**Root Causes:**
- Trigger was accidentally deleted
- Script editor was disconnected from spreadsheet
- Form was recreated (new form ID)

---

#### **Issue: Discord Webhook Failing (Error 4xx)**

**Symptoms:**
- "❌ Discord webhook failed with code: 401" or 404
- Posts not appearing in Discord

**Diagnosis:**
```javascript
testWebhookDirect();  // Tests webhook directly
```

**Solution:**

1. Verify webhook still exists in Discord:
   ```
   Server Settings → Integrations → Webhooks
   Check if "Suggesty" webhook is present
   ```

2. Regenerate webhook if deleted:
   ```
   Create new webhook → Copy URL
   ```

3. Update Script Properties:
   ```javascript
   function updateDiscordWebhook() {
     const scriptProperties = PropertiesService.getScriptProperties();
     scriptProperties.setProperty('DISCORD_WEBHOOK_URL', 'NEW_WEBHOOK_URL');
   }
   ```

---

#### **Issue: Slack API Error "invalid_auth"**

**Symptoms:**
- "❌ Slack API failed: invalid_auth"
- Slack messages not posting

**Diagnosis:**
```javascript
checkSlackConfig();  // Verifies credentials
```

**Solution:**

1. Check token validity:
   ```javascript
   setupSlackCredentials();  // Runs auth.test
   ```

2. If token expired, regenerate:
   ```
   Slack API Dashboard → OAuth & Permissions
   → Reinstall to Workspace → Copy new token
   ```

3. Update Script Properties:
   ```javascript
   function updateSlackToken() {
     const scriptProperties = PropertiesService.getScriptProperties();
     scriptProperties.setProperty('SLACK_BOT_TOKEN', 'xoxb-NEW-TOKEN');
   }
   ```

---

#### **Issue: Wrong Data in Discord/Slack Posts**

**Symptoms:**
- Fields showing "undefined" or wrong content
- Missing data in embeds

**Diagnosis:**
```javascript
verifyColumnMapping();  // Shows current vs expected columns
```

**Solution:**

1. Check if form was edited (new questions added/removed)
2. Compare header row to `COLUMN_INDEX` in [Suggestions.js:24-36](Suggestions.js#L24-L36)
3. Update column indices if form structure changed:
   ```javascript
   const COLUMN_INDEX = {
     TIMESTAMP: 0,
     EMAIL: 1,
     SUGGESTION: 2,  // Update these if form changed
     CATEGORY: 3,
     // ... etc
   };
   ```

---

#### **Issue: Tracking ID Not Added to Spreadsheet**

**Symptoms:**
- Column K remains empty after submission
- Error: "Could not add tracking ID"

**Diagnosis:**
```javascript
verifyColumnMapping();  // Check if Column K exists
```

**Solution:**

1. Verify Column K exists:
   ```
   Open spreadsheet → Check if column K header is "Tracking ID"
   ```

2. If missing, add it:
   ```
   Click column K header cell → Type "Tracking ID"
   ```

3. Check script permissions:
   ```
   Apps Script → Run → Check if "Edit spreadsheet" permission granted
   ```

---

#### **Issue: Email Confirmations Not Sending**

**Symptoms:**
- Leadership-only submissions not receiving email
- No error in logs

**Diagnosis:**
```javascript
// Check if email is valid format
Logger.log(email.includes('@'));  // Should be true
```

**Solution:**

1. Verify form collects email:
   ```
   Google Form → Settings → Collect email addresses = ON
   ```

2. Check Gmail quota:
   ```
   Apps Script → Executions → Check for quota errors
   Daily limit: 100 emails for consumer accounts
   ```

3. Test email function:
   ```javascript
   EmailNotifier.sendTrackingIdConfirmation(
     'test@asu.edu',
     'Test User',
     'HKN-2025-TEST'
   );
   ```

---

### 8.2 Error Messages Reference

| Error Message | Meaning | Solution |
|---------------|---------|----------|
| `❌ Failed to connect. Check your SPREADSHEET_ID` | Invalid or inaccessible spreadsheet | Verify ID and permissions |
| `No valid suggestion text found` | Suggestion field empty | Check form required fields |
| `Category is required` | Category field empty | Ensure dropdown has default |
| `No data rows found` | Spreadsheet has only headers | Normal for new forms |
| `Discord webhook failed with code: 401` | Webhook unauthorized/deleted | Regenerate webhook |
| `Slack API failed: invalid_auth` | Bot token invalid/expired | Regenerate token |
| `Slack API failed: channel_not_found` | Bot not in channel | Invite bot to channel |
| `Could not add tracking ID` | No write permission | Check script authorization |

### 8.3 Debug Logging

#### **View Execution Logs**

```
Apps Script Editor → View → Logs (Ctrl/Cmd + Enter)
```

#### **Check Debug Sheet**

The Debug sheet records all operations:

| Column A | Column B | Column C | Column D |
|----------|----------|----------|----------|
| Timestamp | Action | Details | Status |
| 2025-11-05 12:00 | Discord Success | HKN-2025-1234 | 204 |
| 2025-11-05 12:01 | Slack Success | HKN-2025-1234 | ts:1234.56 |

#### **Enable Verbose Logging**

Add this to any function for detailed debugging:

```javascript
function onFormSubmit(e) {
  console.log('=== VERBOSE DEBUG START ===');
  console.log('Event object:', JSON.stringify(e));
  console.log('Timestamp:', new Date().toISOString());
  // ... rest of function
  console.log('=== VERBOSE DEBUG END ===');
}
```

---

## 9. Maintenance & Operations

### 9.1 Regular Maintenance Tasks

#### **Weekly Tasks**
- [ ] Check Debug sheet for errors
- [ ] Verify Discord/Slack channels active
- [ ] Review new suggestions for follow-up

#### **Monthly Tasks**
- [ ] Run `runFullDiagnostics()`
- [ ] Check Script Properties for expired tokens
- [ ] Review execution quota usage
- [ ] Archive old Debug sheet entries (if >1000 rows)

#### **Quarterly Tasks**
- [ ] Run `runSecurityAudit()`
- [ ] Update documentation with any changes
- [ ] Review form questions for relevance
- [ ] Check if any members need access updates
- [ ] Test leadership webhook routing

#### **Annual Tasks**
- [ ] Rotate Slack bot token
- [ ] Regenerate Discord webhooks
- [ ] Archive old suggestions spreadsheet
- [ ] Update year in tracking ID format (automatic)
- [ ] Review privacy policy compliance

### 9.2 Quota Management

Google Apps Script has daily quotas:

| Resource | Consumer Account | Workspace Account |
|----------|------------------|-------------------|
| Script Runtime | 90 min/day | 6 hrs/day |
| Triggers | 20 triggers | 20 triggers |
| URL Fetch Calls | 20,000/day | 20,000/day |
| Email Recipients | 100/day | 1,500/day |

**Monitoring:**
```
Apps Script → Dashboard → Quotas
Check "Executions" and "URL Fetch" usage
```

**Optimization Tips:**
- Disable Slack if not needed: `const ENABLE_SLACK = false;`
- Reduce Debug sheet logging for high-volume forms
- Archive old spreadsheet data annually

### 9.3 Data Archival

#### **Archive Old Suggestions (Annual)**

```javascript
/**
 * Archives suggestions older than 1 year to a new sheet
 */
function archiveOldSuggestions() {
  const ss = SpreadsheetApp.openById(SPREADSHEET_ID);
  const activeSheet = ss.getSheets()[0];
  const archiveSheet = ss.getSheetByName('Archive 2024') ||
                       ss.insertSheet('Archive 2024');

  const data = activeSheet.getDataRange().getValues();
  const oneYearAgo = new Date();
  oneYearAgo.setFullYear(oneYearAgo.getFullYear() - 1);

  const toArchive = data.filter((row, index) => {
    if (index === 0) return true;  // Keep headers
    return row[0] < oneYearAgo;    // Timestamp column
  });

  if (toArchive.length > 1) {
    archiveSheet.getRange(1, 1, toArchive.length, toArchive[0].length)
                .setValues(toArchive);
    console.log(`Archived ${toArchive.length - 1} old suggestions`);
  }
}
```

#### **Clean Script Properties**

```javascript
/**
 * Removes outdated Slack message timestamps (>90 days)
 */
function cleanOldMessageTimestamps() {
  const scriptProperties = PropertiesService.getScriptProperties();
  const keys = scriptProperties.getKeys();
  let cleaned = 0;

  keys.forEach(key => {
    if (key.startsWith('slack_msg_HKN-')) {
      // Extract year from tracking ID
      const year = key.match(/HKN-(\d{4})-/)[1];
      const currentYear = new Date().getFullYear();

      // Delete if older than current year
      if (parseInt(year) < currentYear) {
        scriptProperties.deleteProperty(key);
        cleaned++;
      }
    }
  });

  console.log(`Cleaned ${cleaned} old message timestamps`);
}
```

### 9.4 Updating the System

#### **Modify Discord Embed Format**

Edit `DiscordIntegration.post()` at [Suggestions.js:390-494](Suggestions.js#L390-L494):

```javascript
// Example: Add a new field
embed.fields.push({
  name: "🆕 New Field",
  value: data.newField,
  inline: false
});
```

#### **Modify Slack Block Layout**

Edit `SlackIntegration.post()` at [Suggestions.js:522-656](Suggestions.js#L522-L656):

```javascript
// Example: Add a new block
blocks.push({
  type: "section",
  text: {
    type: "mrkdwn",
    text: `*New Field*\n${data.newField}`
  }
});
```

#### **Change Tracking ID Format**

Edit `TrackingIdGenerator.generate()` at [Suggestions.js:313-318](Suggestions.js#L313-L318):

```javascript
// Current: HKN-2025-1234
// New Example: SUG-2025-ABC123
static generate() {
  const year = new Date().getFullYear();
  const random = Math.random().toString(36).substr(2, 6).toUpperCase();
  return `SUG-${year}-${random}`;
}
```

---

## 10. Testing & Diagnostics

### 10.1 Built-in Test Functions

All test functions are available in the Apps Script editor:

#### **Quick Setup Test**
```javascript
quickSetup();
```
**Purpose**: Initial setup validation
**Checks**: Spreadsheet, triggers, webhooks
**Duration**: ~30 seconds
**Output**: Setup checklist with ✅/❌ status

---

#### **Full Diagnostics**
```javascript
runFullDiagnostics();
```
**Purpose**: Comprehensive system check
**Checks**:
- Trigger installation
- Webhook connectivity
- Column mapping
- Configuration status

**Duration**: ~45 seconds
**Output**: Detailed diagnostic report

---

#### **Test Form Submission**
```javascript
testFormSubmission();
```
**Purpose**: Simulates a real form submission
**Action**:
- Adds test row to spreadsheet
- Triggers full processing pipeline
- Posts to Discord and Slack

**Data Used**: Sample suggestion about study groups
**Cleanup**: Manually delete test row and posts after verification

---

#### **Test Discord Webhook**
```javascript
testWebhookDirect();
```
**Purpose**: Direct webhook connectivity test
**Action**: Sends simple test embed to Discord
**Duration**: ~5 seconds
**Expected**: Green test message in Discord

---

#### **Test Slack API**
```javascript
testSlackWebhook();
```
**Purpose**: Slack Web API connectivity test
**Action**: Sends test message using Block Kit
**Duration**: ~5 seconds
**Expected**: Test message in Slack channel

---

#### **Verify Column Mapping**
```javascript
verifyColumnMapping();
```
**Purpose**: Checks form-to-column alignment
**Output**:
- Current spreadsheet headers
- Expected COLUMN_INDEX mappings
- Last row data sample

**Use Case**: After form structure changes

---

#### **Check Triggers**
```javascript
checkTriggers();
```
**Purpose**: Verifies trigger installation
**Output**: List of all installed triggers with IDs
**Expected**: onFormSubmit trigger with type ON_FORM_SUBMIT

---

#### **Process Last Submission Manually**
```javascript
processLastSubmission();
```
**Purpose**: Reprocesses the most recent form entry
**Use Case**:
- Missed submission due to script error
- Testing changes without submitting new form
- Recovering from failed execution

---

### 10.2 Diagnostic Workflows

#### **New Installation Verification**

```javascript
// Run in sequence:
1. quickSetup()           // Initial setup
2. testWebhookDirect()    // Test Discord
3. testSlackWebhook()     // Test Slack
4. testFormSubmission()   // End-to-end test
5. checkTriggers()        // Confirm automation
```

---

#### **Troubleshooting Failed Posts**

```javascript
// Run in sequence:
1. checkTriggers()           // Ensure trigger exists
2. verifyColumnMapping()     // Check data structure
3. runFullDiagnostics()      // Comprehensive check
4. testWebhookDirect()       // Test each platform
5. testSlackWebhook()
6. processLastSubmission()   // Retry last entry
```

---

#### **After Form Modification**

```javascript
// Run in sequence:
1. verifyColumnMapping()     // Check new column order
2. Update COLUMN_INDEX if needed
3. testFormSubmission()      // Test with new structure
```

---

### 10.3 Manual Testing Checklist

Test each privacy preference:

- [ ] **Public with Name**
  - Submit form with "Post with my name"
  - Verify name appears in Discord
  - Verify name appears in Slack
  - Check tracking ID in spreadsheet

- [ ] **Anonymous**
  - Submit form with "Post anonymously"
  - Verify "Anonymous Member" in Discord
  - Verify "Anonymous Member" in Slack
  - Check tracking ID in spreadsheet

- [ ] **Leadership Only**
  - Submit form with "Contact Leadership"
  - Verify post in Discord leadership channel
  - Verify NO post in public channels
  - Verify email confirmation received
  - Check tracking ID in spreadsheet

---

### 10.4 Performance Monitoring

#### **Check Execution Time**

```
Apps Script → Executions
Look for onFormSubmit executions
Normal time: 2-5 seconds
```

**Optimization if slow:**
1. Disable unused platform (Discord or Slack)
2. Reduce Debug sheet logging
3. Check if spreadsheet is too large (>10,000 rows)

#### **Monitor API Response Times**

Add timing logs:

```javascript
const startTime = Date.now();
DiscordIntegration.post(suggestionID, data);
const discordTime = Date.now() - startTime;
console.log(`Discord API: ${discordTime}ms`);
```

**Benchmarks:**
- Discord webhook: <1 second
- Slack API: <2 seconds
- Gmail send: <1 second
- Total execution: <5 seconds

---

## 11. Advanced Features

### 11.1 Slack Slash Commands

The system supports Slack slash commands for suggestion management.

#### **Setup Web App Deployment**

1. In Apps Script editor: Deploy → New deployment
2. Type: Web app
3. Description: "Suggesty Slash Commands"
4. Execute as: Me
5. Who has access: Anyone (Slack will authenticate)
6. Click Deploy → Copy Web App URL

#### **Configure Slack Slash Commands**

1. Go to [api.slack.com/apps](https://api.slack.com/apps) → Your App
2. Navigate to "Slash Commands" → Create New Command

**Command 1: Delete Suggestion**
```
Command: /suggesty-delete
Request URL: [YOUR_WEB_APP_URL]
Short Description: Delete a suggestion by tracking ID
Usage Hint: HKN-YYYY-XXXX
```

**Command 2: List Suggestions**
```
Command: /suggesty-list
Request URL: [YOUR_WEB_APP_URL]
Short Description: List all stored suggestions
```

#### **Usage Examples**

```
/suggesty-delete HKN-2025-1234
  → ✅ Successfully deleted suggestion HKN-2025-1234

/suggesty-list
  → 📋 Stored Suggestions (15):
     • HKN-2025-1234
     • HKN-2025-1235
     • HKN-2025-1236
     ...
```

#### **Code Reference**

Slash command handlers are in [Suggestions.js:1674-1803](Suggestions.js#L1674-L1803):
- `doPost(e)` - Main webhook receiver
- `handleDeleteCommand()` - Delete logic
- `handleListCommand()` - List logic
- `createSlackResponse()` - Response formatter

### 11.2 Custom Priority Colors

Modify color thresholds and values:

```javascript
// In configuration section:
const PRIORITY_HIGH_THRESHOLD = 9;      // Change from 8
const PRIORITY_MEDIUM_THRESHOLD = 6;    // Change from 5

// In DiscordIntegration class:
static getPriorityColor(priority) {
  if (priority >= 9) return 0xFF0000;   // Bright red
  if (priority >= 6) return 0xFFFF00;   // Yellow
  if (priority >= 3) return 0x00FF00;   // Green
  return 0x808080;                       // Gray for low
}
```

### 11.3 Multi-Language Support

Add language detection and translation:

```javascript
class LanguageHandler {
  static detectLanguage(text) {
    // Use Google's LanguageApp (Apps Script built-in)
    return LanguageApp.translate(text, '', 'en');
  }

  static addTranslation(embed, originalText) {
    const detected = LanguageApp.detectLanguage(originalText);

    if (detected !== 'en') {
      const translated = LanguageApp.translate(originalText, detected, 'en');
      embed.fields.push({
        name: "🌐 Translation (Auto)",
        value: translated,
        inline: false
      });
    }

    return embed;
  }
}
```

### 11.4 Automated Follow-Up Reminders

Create time-based triggers for leadership:

```javascript
/**
 * Sends weekly digest of unreviewed suggestions
 */
function sendWeeklyDigest() {
  const ss = SpreadsheetApp.openById(SPREADSHEET_ID);
  const sheet = ss.getSheets()[0];
  const data = sheet.getDataRange().getValues();

  const oneWeekAgo = new Date();
  oneWeekAgo.setDate(oneWeekAgo.getDate() - 7);

  const pending = data.filter((row, index) => {
    if (index === 0) return false;  // Skip headers
    return row[0] > oneWeekAgo;     // Recent submissions
  });

  if (pending.length > 0) {
    const digestEmbed = {
      title: "📊 Weekly Suggestion Digest",
      description: `${pending.length} suggestions submitted this week`,
      color: 0x8C1D40,
      fields: pending.slice(0, 10).map(row => ({
        name: row[10],  // Tracking ID
        value: `**Category:** ${row[3]}\n**Priority:** ${row[6]}/10`,
        inline: true
      }))
    };

    const payload = {
      username: 'Suggesty',
      embeds: [digestEmbed]
    };

    UrlFetchApp.fetch(SecureConfig.getDiscordLeadershipWebhook(), {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      payload: JSON.stringify(payload)
    });
  }
}

// Install trigger: Edit → Current project's triggers → Add trigger
// Function: sendWeeklyDigest
// Event: Time-driven, Week timer, Every Monday, 9 AM
```

### 11.5 Analytics Dashboard

Track suggestion metrics:

```javascript
/**
 * Generates monthly analytics report
 */
function generateMonthlyAnalytics() {
  const ss = SpreadsheetApp.openById(SPREADSHEET_ID);
  const sheet = ss.getSheets()[0];
  const data = sheet.getDataRange().getValues();

  const thisMonth = new Date();
  thisMonth.setDate(1);  // First day of month

  const monthData = data.filter((row, index) => {
    if (index === 0) return false;
    return row[0] >= thisMonth;
  });

  // Calculate metrics
  const totalSubmissions = monthData.length;
  const avgPriority = monthData.reduce((sum, row) =>
    sum + parseInt(row[6]), 0) / totalSubmissions;

  // Category breakdown
  const categories = {};
  monthData.forEach(row => {
    categories[row[3]] = (categories[row[3]] || 0) + 1;
  });

  // Privacy preference breakdown
  const privacy = {};
  monthData.forEach(row => {
    const pref = row[9] || 'Unknown';
    privacy[pref] = (privacy[pref] || 0) + 1;
  });

  // Create report
  const reportEmbed = {
    title: `📈 Monthly Analytics - ${thisMonth.toLocaleString('default', { month: 'long', year: 'numeric' })}`,
    color: 0x8C1D40,
    fields: [
      {
        name: "Total Submissions",
        value: totalSubmissions.toString(),
        inline: true
      },
      {
        name: "Average Priority",
        value: avgPriority.toFixed(1) + "/10",
        inline: true
      },
      {
        name: "Category Breakdown",
        value: Object.entries(categories)
          .map(([cat, count]) => `${cat}: ${count}`)
          .join('\n'),
        inline: false
      },
      {
        name: "Privacy Preferences",
        value: Object.entries(privacy)
          .map(([pref, count]) => `${pref}: ${count}`)
          .join('\n'),
        inline: false
      }
    ]
  };

  const payload = {
    username: 'Suggesty Analytics',
    embeds: [reportEmbed]
  };

  UrlFetchApp.fetch(SecureConfig.getDiscordLeadershipWebhook(), {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    payload: JSON.stringify(payload)
  });

  console.log('Monthly analytics sent to leadership channel');
}
```

---

## 12. FAQ

### 12.1 General Questions

**Q: What happens if the script stops working?**
A: The system includes automatic error notifications to Discord/Slack leadership channels. Check the Debug sheet for details and run `runFullDiagnostics()`.

**Q: Can members edit their submissions after submitting?**
A: No. Google Forms doesn't support post-submission editing. Members must submit a new form or contact leadership with their tracking ID.

**Q: How long are submissions stored?**
A: Indefinitely in Google Sheets. Archive old data annually using `archiveOldSuggestions()`.

**Q: Can we use this for multiple forms?**
A: Not directly. Each form needs its own script instance. Copy the script and change the SPREADSHEET_ID.

**Q: What's the maximum form submission rate?**
A: Google Apps Script can handle ~20 submissions per minute. Beyond that, some may queue.

---

### 12.2 Technical Questions

**Q: Why use Script Properties instead of variables?**
A: Script Properties persist between executions and are encrypted by Google, making them secure for API credentials.

**Q: Can we use Discord bots instead of webhooks?**
A: Webhooks are simpler and don't require bot hosting. However, you can modify the `DiscordIntegration` class to use Discord.js if needed.

**Q: Why does Slack use Web API but Discord uses webhooks?**
A: Discord webhooks are sufficient for posting. Slack requires Web API for advanced features like message deletion and editing.

**Q: How do we handle form spam?**
A: Add Google Forms' built-in captcha: Form Settings → "Limit to 1 response" and require ASU login.

**Q: Can we add file attachments to suggestions?**
A: Yes. Add a file upload question to your form. Access uploaded files via Google Drive API and include links in Discord/Slack embeds.

---

### 12.3 Customization Questions

**Q: Can we change the tracking ID format?**
A: Yes. Edit `TrackingIdGenerator.generate()` at [Suggestions.js:313-318](Suggestions.js#L313-L318).

**Q: Can we add more fields to Discord embeds?**
A: Yes. Edit `DiscordIntegration.post()` at [Suggestions.js:390-494](Suggestions.js#L390-L494) and add new embed fields.

**Q: Can we customize the email template?**
A: Yes. Edit `EmailNotifier.sendTrackingIdConfirmation()` at [Suggestions.js:674-720](Suggestions.js#L674-L720).

**Q: Can we post to multiple Discord servers?**
A: Yes. Create additional webhooks and modify the posting logic to loop through an array of webhook URLs.

**Q: Can we use this with Microsoft Teams?**
A: Yes. Replace Slack integration with Teams incoming webhooks. Teams uses a similar JSON format to Discord.

---

### 12.4 Troubleshooting Questions

**Q: Why isn't the trigger firing?**
A: Run `checkTriggers()` to verify installation. Triggers can be deleted if the script is disconnected from the spreadsheet.

**Q: Why are Discord messages not formatted correctly?**
A: Check if your form structure changed. Run `verifyColumnMapping()` and update `COLUMN_INDEX` if needed.

**Q: Why do some submissions not generate tracking IDs?**
A: The script might not have edit permissions. Reauthorize: Run any function → Accept all permissions.

**Q: Why are leadership submissions going to the public channel?**
A: Leadership webhook may not be configured. Run `setDiscordLeadershipWebhook()` and verify the URL.

---

## Appendix A: Permission Reference

### Required Google Permissions

When first running the script, you'll be asked to authorize:

| Permission | Reason | Risk Level |
|------------|--------|------------|
| View and manage spreadsheets | Read form data, write tracking IDs | Medium |
| Send email as you | Confirmation emails for leadership submissions | Low |
| Connect to external service | Discord/Slack API calls | Low |
| Manage script properties | Store credentials securely | Low |

All permissions are standard for Google Apps Script automation.

---

## Appendix B: File Structure

```
IEEE-HKN-Suggestion-System/
│
├── Suggestions.js          # Main script file (all code)
│
├── [Google Spreadsheet]
│   ├── Form Responses      # Main sheet with submissions
│   └── Debug               # Auto-generated log sheet
│
├── README.md               # Quick start guide
└── DOCUMENTATION.md        # This file (complete docs)
```

---

## Appendix C: Contact & Support

### For IEEE-HKN Members

- **Submit Suggestions**: [Google Form Link]
- **Track Your Suggestion**: Contact leadership with your tracking ID
- **Privacy Concerns**: Email leadership directly

### For Webmasters & Developers

- **Primary Maintainer**: Current IEEE-HKN Webmaster
- **Code Repository**: [GitHub Link - if applicable]
- **Apps Script Project**: Accessible via Google Sheets → Extensions → Apps Script
- **Emergency Contact**: IEEE-HKN Leadership Email

### Resources

- [Google Apps Script Documentation](https://developers.google.com/apps-script)
- [Discord Webhook Guide](https://discord.com/developers/docs/resources/webhook)
- [Slack API Reference](https://api.slack.com/docs)
- [Apps Script Best Practices](https://developers.google.com/apps-script/guides/support/best-practices)

---

## Document Version History

| Version | Date | Changes | Author |
|---------|------|---------|--------|
| 2.0.0 | Nov 2025 | Complete system documentation | IEEE-HKN Webmaster |
| 1.0.0 | [Prior Date] | Initial implementation | [Prior Webmaster] |

---

**END OF DOCUMENTATION**

*Last reviewed: November 2025*
*Next review: August 2026*
