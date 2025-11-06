# Facebook Messenger Attendance Bot

## Overview
This project is a Facebook Messenger bot designed to automate group chat attendance tracking. Its primary purpose is to simplify daily attendance recording, manage group members, and enforce group rules through an automated system. Key capabilities include authenticating with Facebook via appstate, automatically tracking group members, managing daily check-ins, customizing greetings, and implementing a robust moderation system with advanced vulgar word detection, spam protection, and a progressive ban system. The bot ensures persistent data storage and per-group chat administrative control, aiming to provide a comprehensive solution for managing active Facebook Messenger groups.

## User Preferences
I prefer iterative development, where features are implemented and tested in small, manageable steps. I value clear and concise explanations, avoiding overly technical jargon unless absolutely necessary. I want the agent to prioritize robust error handling and security in its implementations. When making significant changes or architectural decisions, please ask for my approval first. I prefer the use of modern JavaScript features and clean, readable code. I also expect the agent to be proactive in identifying potential issues or improvements.

## System Architecture
The bot is built on **Node.js 20** and utilizes the `fca-unofficial` library for Facebook Messenger API interaction. Data persistence is managed entirely through **JSON files**, with separate files for different data types, ensuring data survives bot restarts.

**UI/UX Decisions (Implicit):**
- **Command-line Interface:** All interactions are command-based within the Messenger chat.
- **Paginated Help:** The help menu is paginated for readability, displaying 5 commands per page.
- **Role-based Command Access:** Commands are filtered based on user roles (all users, admins, super admin).
- **Clear Status Indicators:** Uses emojis (✅, 💔) for attendance and missed attendance.
- **Varied Bot Responses:** Employs multiple responses for invalid commands to enhance user experience.

**Technical Implementations & Feature Specifications:**
- **Authentication:** Uses `appstate.json` for Facebook session cookies.
- **Attendance System:**
    - Daily check-in via `.present` command.
    - Automatic member detection and live synchronization with group membership changes.
    - Tracks consecutive absences (Philippines timezone UTC+8).
    - **Admins and Super Admins are completely exempt from attendance tracking** - they never appear in attendance lists or missed attendance lists.
    - Auto-kick after 3 consecutive days of absence.
    - Manual and targeted absence resets for admins.
    - Temporary exclusion mechanism for members from attendance.
    - **Nickname Display:** Uses group chat nicknames in attendance lists; shows "(Please apply Gamer Tag/Nick Name)" for members without nicknames.
- **Moderation System:**
    - **Advanced Vulgar Word Detection:** Maximum-strength obfuscation detection with comprehensive Unicode font normalization (circled, fraktur, sans-serif, old italic, fullwidth, letterlike symbols), triple-pass normalization, aggressive character replacement, enhanced de-duplication, and comprehensive special character handling to detect highly obfuscated vulgar words. Handles accents, leetspeak, special characters, repeated letters, fancy Unicode fonts, **hyphenated variations (fuck-you, tang-ina-mo, potangina-mo)**, and underscores.
    - **Split Message Detection:** Tracks user's last 5 messages within 30-second windows to detect vulgar words spread across multiple messages (e.g., "tang" + "ina mo" = "tanginamo").
    - **Bot Mention Protection:** Automatically KICKS users who mention the bot user ID (61572200383571 - Tanginamo Pabayo) inappropriately.
    - **Startup Scan:** On bot startup, scans last 50 messages in each group for missed vulgar words while bot was offline, avoiding duplicate warnings.
    - **Abbreviation Expansion:** Automatically expands common slang.
    - **Warning System:** Auto-warning on vulgar words (applies to all users, including admins).
    - **War Extreme Mode:** Temporarily disables all warning detection.
    - **Spam Detection:** Auto-kick for 7 same messages or 7 invalid commands within 10 seconds.
    - **Unsent Message Spam Protection:** Kicks users who unsend 5+ messages within 60 seconds; after 3 consecutive kicks, applies progressive ban system (1st ban: 3 days, 2nd ban: 7 days, 3rd ban: permanent).
    - **Progressive Ban System:** 1st ban (3 days), 2nd ban (7 days), 3rd+ ban (permanent). **Ban records are retained even after unbanning** to ensure proper escalation.
    - **Auto-lift Expired Bans:** Checks every minute to lift temporary bans.
    - **Manual Ban/Unban:** Admins can ban/unban users with unique Ban IDs.
    - **Separate Ban Messages:** Ban announcement and "Uy may lumipad HAHAHA\n\nGoodboy ka next time ha HAHA 😂😂" are sent as separate messages for better UX.
    - **Admin Protection:** Admins cannot be banned directly; privileges must be removed first.
    - **Super Admin Nuclear Option:** `.banall` command for mass banning (Super Admin only).
- **Message Management:**
    - **Message Caching:** Caches messages and attachments for 1 minute for unsent message recovery.
    - **Instant Unsent Message Recovery:** Reveals deleted messages immediately.
- **Admin System:**
    - **Per-Group-Chat Admin System:** Admin privileges are managed per group and persist.
    - Dynamic management allowing admins to add/remove other admins within their group.
- **Notifications:**
    - **New User Alerts:** When a user joins a group, the super admin (100092567839096) receives a private message with the user's name, UID, account creation date, gender, and profile URL.
- **Customization:**
    - Customizable group greetings.
    - Configurable auto-warning keywords.
    - Storage and display of server IP:port information.

**System Design Choices:**
- **Persistent Storage:** All dynamic data (greetings, attendance, bans, warnings, admins, etc.) is stored in JSON files to maintain state across bot restarts.
- **Modular Command Handling:** Commands are structured for easy expansion and role-based access control.
- **Timezone Awareness:** Attendance tracking and daily resets are aligned with the Philippines timezone (UTC+8).
- **No Command Cooldown:** Commands execute instantly for a fluid user experience.

## Recent Changes (November 2025)
### Enhanced Vulgarity Detection System
- **Hyphenated Word Detection:** Now detects vulgar words with hyphens (fuck-you, tang-ina-mo, potangina-mo) by removing hyphens and underscores during normalization.
- **Cross-Message Detection:** Tracks last 5 messages per user within 30-second windows to catch vulgar words split across multiple messages (e.g., "tang" followed by "inamo").
- **Bot Mention Protection:** Automatically KICKS users who mention bot ID 61572200383571 (Tanginamo Pabayo) inappropriately (no ban, just kick).
- **Startup Scan:** Bot now scans recent message history on startup to catch vulgar words missed while offline, avoiding duplicate warnings.

### Improved Ban System
- **Persistent Ban Records:** Ban counts are retained even after unbanning, ensuring proper ban duration escalation (3 days → 7 days → permanent).
- **Enhanced Messaging:** Ban messages now split into two parts: official ban notification followed by "Uy may lumipad HAHAHA\n\nGoodboy ka next time ha HAHA 😂😂".

### Attendance System Improvements
- **Admin Exclusion:** Admins and super admins are now completely excluded from attendance tracking and lists.
- **Nickname Display:** Attendance lists show group chat nicknames; users without nicknames show "(Please apply Gamer Tag/Nick Name)" suffix.

### Unsent Message Spam Protection
- **Kick on Spam:** Users who unsend 5+ messages within 60 seconds are automatically kicked for spamming.
- **Progressive Ban After Repeated Kicks:** Users kicked 3 times consecutively for spamming are banned using the progressive ban system (1st ban: 3 days, 2nd ban: 7 days, 3rd ban: permanent).
- **Automatic Tracking:** System maintains kick history without requiring admin commands.

### Super Admin Notifications
- **New User Alerts:** When anyone joins a group, super admin receives detailed notification including name, UID, account creation date, gender, and profile URL.

## External Dependencies
- **Facebook Messenger API:** `fca-unofficial` library for programmatic interaction with Facebook Messenger.
- **JSON Files:** Used as the primary data storage mechanism for all persistent bot data.