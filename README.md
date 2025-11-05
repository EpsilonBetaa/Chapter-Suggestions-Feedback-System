# IEEE-HKN Chapter Suggestion System

**Automated feedback processing and routing for IEEE-HKN at Arizona State University**

[![Google Apps Script](https://img.shields.io/badge/Google%20Apps%20Script-V8-blue.svg)](https://developers.google.com/apps-script)
[![Version](https://img.shields.io/badge/version-2.0.0-green.svg)](WEBMASTER_GUIDE.md)
[![Status](https://img.shields.io/badge/status-production-success.svg)](WEBMASTER_GUIDE.md)

---

## What It Does

Automatically processes Google Form submissions and posts them to Discord and Slack channels with:
- Unique tracking IDs (`HKN-YYYY-XXXX`)
- Privacy controls (public, anonymous, or leadership-only)
- Priority-based color coding
- Email confirmations for confidential submissions

---

## Quick Start

### 1. Configure Spreadsheet ID
```javascript
// Line 13 in Suggestions.js
const SPREADSHEET_ID = 'YOUR_SPREADSHEET_ID_HERE';
```

### 2. Add Tracking Column
Add "Tracking ID" as Column K header in your spreadsheet.

### 3. Run Setup
```javascript
// In Apps Script editor:
quickSetup()
```

### 4. Configure Webhooks
```javascript
// Run these functions:
function setupDiscordWebhooks() {
  const props = PropertiesService.getScriptProperties();
  props.setProperty('DISCORD_WEBHOOK_URL', 'YOUR_WEBHOOK_URL');
  props.setProperty('DISCORD_LEADERSHIP_WEBHOOK_URL', 'YOUR_LEADERSHIP_URL');
}

setupSlackCredentials();  // Built-in function
```

### 5. Test
```javascript
testFormSubmission();
```

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Not posting automatically | `installTrigger()` |
| Discord failing | `testWebhookDirect()` |
| Slack failing | `checkSlackConfig()` |
| Wrong data | `verifyColumnMapping()` |
| Full diagnostic | `runFullDiagnostics()` |

---

## Documentation

📘 **[WEBMASTER_GUIDE.md](WEBMASTER_GUIDE.md)** - Complete technical documentation with:
- Architecture & component details
- Configuration reference
- Platform integration guides
- Maintenance schedules
- Advanced features
- FAQ

---

## File Structure

```
Suggestion/
├── Suggestions.js        # Main script
├── README.md            # This file
└── WEBMASTER_GUIDE.md   # Complete documentation
```

---

## Support

**For Issues**: Refer to the troubleshooting section in [WEBMASTER_GUIDE.md](WEBMASTER_GUIDE.md)

**External Resources**:
- [Google Apps Script Documentation](https://developers.google.com/apps-script)
- [Discord Webhooks Guide](https://discord.com/developers/docs/resources/webhook)

## Maintenance

**For Current Webmasters**: Please review the [WEBMASTER_GUIDE.md](WEBMASTER_GUIDE.md) thoroughly before making any modifications to the system.

**For Incoming Webmasters**: Schedule a walkthrough with the outgoing webmaster and reference the guide for complete system documentation.

## License

Maintained by IEEE-HKN Epsilon Beta Chapter Webmasters at Arizona State University.

---

**Last Updated**: November 2025
**Current Webmaster**: Vishesh Singh Rajput (2024-2025, 2025-2026)
