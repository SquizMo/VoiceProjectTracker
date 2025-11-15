# VOICE PROJECT TRACKER — README

A script-powered dashboard that scans your Google Drive voice-over folders, lists every file in a Google Sheet, lets you add “Final Production” links, track Current/Past status, and sync statuses with your separate Auditions tab. It also includes a polished HTML UI (buttons, tabs, KPIs, Settings) that you can open in the Sheet sidebar.

---

## 1. WHAT THIS TOOL DOES

---

* Recursively scans two Google Drive folders:

  * **Voice Projects** (finished or in-progress recordings)
  * **Auditions** (raw/alt takes, submissions)

* Writes rows into your Google Sheet with:

  * Folder path
  * File name
  * Character (auto-parsed from file name using your configured **Performer / Stage Name**, e.g. `John Doe`)
  * Date added
  * MIME type
  * File link
  * Final Production link
  * Status

* Syncs Voice status → Auditions status:

  * Voice **"Current"** → Auditions **"Booked"**
  * Voice **"Past"**    → Auditions **"Submitted"**
  * Never overwrites **"Passed"**

* Adds a sidebar UI with:

  * Dashboard, Updates, Search, Insights, Auditions, Settings, Actions
  * Quick filters & search
  * Bulk update tools
  * CSV/PDF export
  * A **Settings** tab for folder IDs **and** performer name
  * A **Donate** button/link to support the project

---

## 2. PREREQUISITES

---

* A Google account
* Google Sheets
* Access to Google Drive (web)
* (Optional, recommended) **Google Drive for desktop** installed and signed in

---

## 3. INSTALL GOOGLE DRIVE FOR DESKTOP (TO KEEP LOCAL FILES SYNCED)

---

### Windows

1. Search the web for **“Google Drive for desktop download”** (official Google page).
2. Download and run the installer.
3. Sign in with your Google account.
4. In Drive settings:

   * Choose **Stream files** (recommended) or **Mirror files**.
   * Confirm your local Drive letter (Windows often maps it to `G:`).
5. You will see Google Drive in File Explorer. Anything in **My Drive** or **Shared drives** will sync.

### macOS

1. Download Google Drive for desktop from Google’s site.
2. Open the `.dmg`, drag the app into **Applications**.
3. Sign in with your Google account.
4. In Finder, you will see Google Drive mounted in the sidebar.

> Tip: Work from your desktop DAW while Drive syncs to the cloud. The script reads the *cloud* folders—sync keeps the cloud in step with your desktop.

---

## 4. CREATE THE GOOGLE SHEET + APPS SCRIPT

---

1. Create a new Google Sheet (e.g., **“Voice Project Tracker”**).

2. Click **Extensions → Apps Script**.

3. In the Script Editor:

   * Create a file named **`Code.gs`** and paste the **entire server script**
     (everything from `// ========== MAIN SCRIPT ==========` down to the last function).
   * Create a file named **`Sidebar.html`** and paste the **entire HTML block**
     (everything from `<!DOCTYPE html>` to `</html>`).

4. Click **Save**.

---

## 5. CONFIGURE YOUR FOLDER IDS & PERFORMER NAME (SETTINGS TAB)

---

The script stores your config in **Document Properties** and exposes them through the **Settings** tab in the Tracker Panel UI.

### Under the hood

`getTrackerSettings()` returns an object like:

```js
{
  voiceFolderId: '...',
  auditionFolderId: '...',
  performerName: 'John Doe'   // or whatever name you set in Settings
}
```

Convenience helpers:

* `getVoiceFolderId()`
* `getAuditionFolderId()`
* `getPerformerName_()`

On first run, the script seeds defaults (original hard-coded IDs and a default performer, which you’ve now set to **John Doe**). You can change this any time via Settings.

### How to find a folder ID

1. In Drive (web), open the folder.

2. The URL looks like:

   ```text
   https://drive.google.com/drive/folders/1AbCDeFGhijkLMNOPqR...
   ```

3. Copy the long string after `/folders/` — that’s the **Folder ID**.

### How to set in the Tracker

1. In the Sheet, go to **Voice Tracker → Show Tracker Panel**.

2. In the sidebar, switch to the **Settings** tab.

3. Fill in:

   * **Voice Folder ID** – where your finished / in-progress projects live
   * **Audition Folder ID** – where your audition files live
   * **Performer / Stage Name** – the name that appears at the start of your file names (e.g., `John Doe`)

4. Click **Save Settings**.

From that point on:

* **Refresh Sheet** uses the **Voice Folder ID** from Settings.
* **Refresh Auditions** uses the **Audition Folder ID** from Settings.
* CSV/PDF exports also write into the configured Voice folder.
* Character detection uses the **Performer / Stage Name** you set (e.g., `John Doe`).

If either folder ID is missing, affected functions will throw a clear error telling you to configure it in **Settings**.

---

## 6. FILE NAMING FORMAT (CHARACTER AUTO-DETECTION)

---

The current script detects **Character** based on your **Performer / Stage Name** from Settings, which in your setup is now **John Doe**.

### 6.1. Primary pattern (using performerName from Settings)

In `collectFiles()`, the script does:

```js
var performerName = getPerformerName_();      // from Settings
var escapedName   = escapeRegex_(performerName);
var pattern       = new RegExp('^' + escapedName + '\\s*-\\s*(.+?)\\s*-');
```

With `performerName = "John Doe"`, this matches file names like:

```text
John Doe - Character Name - Anything else.ext
```

Examples:

* `John Doe - Captain Orion - Take 3.wav`    → Character = `Captain Orion`
* `John Doe - Shop Announcer - v2.mp3`      → Character = `Shop Announcer`

As long as the file starts with:

```text
John Doe - <Character> - ...
```

the Character column will be filled correctly.

### 6.2. Fallback generic pattern

If the performer name pattern **doesn’t** match (or performerName is blank), the script falls back to a generic pattern:

```js
var match = fileName.match(/^[^-]+-\s*(.+?)\s*-/);
```

This ignores the first hyphen-separated segment and treats the **second** segment as the Character:

```text
Anything - Character - Rest.ext
```

Examples:

* `Somebody - Witch Queen - final.wav`   → Character = `Witch Queen`
* `Demo - Narrator - v1.mp3`            → Character = `Narrator`

### 6.3. When Character is “Unknown”

If neither pattern matches, Character is set to `"Unknown"`. That usually means:

* The file name doesn’t have the `Name - Character - ...` shape, or
* There are no hyphens in the name, or
* There’s something unusual about the spacing.

### 6.4. Recommended naming template

For consistency, with your current setup:

```text
John Doe - Character Name - Project Name - 2025-11-10 - take01.wav
```

(If someone else uses the tool, they’ll just replace `John Doe` with their own performer name in **Settings**.)

---

## 7. USING THE UI & WHAT EACH BUTTON DOES

---

After saving scripts:

1. Return to the Sheet and refresh.
2. Run any function when prompted and accept permissions.

You will see a **“Voice Tracker”** menu in the Sheet.

### 7.1. Voice Tracker menu (top of the Sheet)

* **Refresh Auditions**
  Scans the Auditions folder (from Settings) and rebuilds the **Auditions** tab.

* **Sync Auditions from Voice**
  Applies the mapping logic:

  * Voice **Current** → Auditions **Booked**
  * Voice **Past** → Auditions **Submitted**
  * Never overwrites **Passed**

* **Show Tracker Panel**
  Opens the sidebar HTML UI (tabs, KPIs, Settings, etc.).

* **Refresh Sheet**
  Scans the Voice Projects folder (from Settings) and rebuilds the main Voice sheet.

* **Export Summary PDF**
  Saves a PDF snapshot of the whole Sheet into your **Voice Projects** folder.

* **Install Auto-Refresh (hourly)**
  Adds a time-based trigger to run `updateVoiceProjects` every hour.

* **Remove Auto-Refresh**
  Deletes that trigger.

* **Export Visible Auditions CSV / Voice CSV**
  Exports **only currently visible (filtered)** rows to CSV and saves into your Voice folder.

### 7.2. Sidebar (HTML UI) Tabs

**Dashboard**

* Quick filter buttons for Voice: **Current / Past / All**.

**Updates**

* **Single update**:

  * Select a **Character**.
  * Optionally paste a specific **File URL** (to target only one file).
  * Add a **Final Production link** and/or set **Status**.
* **Bulk update**:

  * Select one or more **Characters**.
  * Optionally select specific **File(s)** for those characters.
  * Apply a Final link and/or Status to all selected rows at once.

**Search**

* Filter by **Character**, **Folder**, or **File Name**.
* Filter by **Status** (All / Current / Past).
* Uses the same sheet, just hides or shows rows in-place.

**Insights**

* **KPIs** (metrics) such as:

  * Voice — Total, Current, Past, Recent additions (7/30 days)
  * Auditions — Total, Pending, Submitted, Booked, Passed, Recent (7/30 days)
* **Recent Activity** table showing a merged timeline from Voice and Auditions.

**Auditions**

* Status chips: **All / Pending / Submitted / Booked / Passed**.
* Quick search bar for character / file / folder.
* **Sync Status** button to run the Voice → Auditions sync.
* **Refresh** to rescan the Auditions folder.

**Settings**  🔧

* **Voice Folder ID** – where your Voice projects live.
* **Audition Folder ID** – where your Auditions live.
* **Performer / Stage Name** – used to recognize filenames like
  `John Doe - Character - ...` for character detection.

Buttons:

* **Save Settings** — writes these values into Document Properties (`VOICE_TRACKER_SETTINGS`).
* **Reload** — reloads the current saved settings into the form.

**Actions**

* Quick access to:

  * **Refresh Voice Sheet**
  * **Export Summary PDF**

### Donate button

In the top card, there is also a **Donate** button:

* **Coffee is nice but not necessary** * 

---

## 8. FIRST-RUN CHECKLIST

---

1. **Create the Sheet and scripts** (`Code.gs` + `Sidebar.html`).

2. Open the Sheet, go to **Voice Tracker → Show Tracker Panel**.

3. In the sidebar, open **Settings** and set:

   * **Voice Folder ID**
   * **Audition Folder ID**
   * **Performer / Stage Name** – in your case: `John Doe`

   Then click **Save Settings**.

4. Ensure your file names follow the pattern, e.g.:

   ```text
   John Doe - Character - Project - ...
   ```

5. In the **Voice Tracker** menu:

   * Click **Refresh Sheet** to build the Voice list.
   * Click **Refresh Auditions** to build the Auditions list.

6. Open **Show Tracker Panel** and test:

   * **Dashboard**: Current / Past / All.
   * **Updates**: add a Final Production link and set Status for at least one file.
   * **Auditions**: click **Sync Status**, verify Booked / Submitted match Voice statuses.

7. (Optional) Install **Auto-Refresh (hourly)**.

---

## 9. TROUBLESHOOTING

---

**Authorization prompts**

* The first time you run anything, Google will ask you to approve OAuth permissions. Accept them.

**“Voice Folder ID is not configured” or “Audition Folder ID is not configured”**

* Open **Show Tracker Panel → Settings** and confirm both IDs are set.
* Make sure there are no extra spaces when you paste the ID.

**No rows appear after Refresh**

* Double-check the folder IDs in Settings.
* Confirm the Drive folders actually contain files.
* Confirm your account can access those folders (Shared drives, permissions, etc.).

**Character shows “Unknown”**

* File name didn’t match either:

  * `John Doe - Character - ...`
  * or the generic `<anything> - Character - ...` pattern.
* Update the file name to match the recommended format, or verify your **Performer / Stage Name** in Settings is exactly `John Doe` (or whatever you’re using in filenames).

**Donate link does not open**

* Ensure the HTML uses:

  ```html
  <a href="https://..." target="_blank" rel="noopener">Donate</a>
  ```

  instead of a `window.open` call, which can be blocked.

**Auditions status not updating**

* Use **Voice Tracker → Sync Auditions from Voice** or the **Sync Status** button in the Auditions tab.
* Ensure file names in Auditions and Voice match (same **Character** + same **File Name**).

**Performance is slow with huge folders**

* Organize projects into subfolders to keep lists manageable.
* Consider archiving old projects into a separate folder that the Tracker doesn’t scan.

---

## 10. CUSTOMIZATIONS (OPTIONAL)

---

### 10.1. Stage / Performer name

* **Preferred method:**
  Just change it in the **Settings** tab (Performer / Stage Name). Your current example is `John Doe`, but it can be anything. The regex used by `collectFiles()` will automatically adjust and remain safe/escaped.

* **Advanced (code)**:
  If you ever want to change pattern behavior, look at:

  ```js
  function getPerformerName_() { ... }
  function escapeRegex_(s) { ... }
  function collectFiles(...) { ... }
  ```

  But for most use cases, the Settings UI is enough.

### 10.2. Status vocab

* Auditions: `["Pending", "Submitted", "Booked", "Passed"]`
* Voice:     `["Current", "Past"]`

If you add new statuses:

* Update Auditions conditional formatting rules (colors).
* Update `syncAuditionsWithProjects()` so it knows how to map Voice statuses → Auditions statuses.

### 10.3. Auto-refresh frequency

In `installAutoRefresh()` you’ll see:

```js
ScriptApp.newTrigger('updateVoiceProjects')
  .timeBased()
  .everyHours(1)
  .create();
```

Change `.everyHours(1)` to `.everyMinutes(15)` (or other allowed frequencies) if you want more frequent updates.

---

## 11. RECOMMENDED FILE NAME TEMPLATE

---

Using your example performer name:

```text
John Doe - Character Name - Project Name - 2025-11-10 - take01.wav
```

If another user adopts the tool, they’d just swap `John Doe` for their own name and update the **Settings** tab accordingly.

---

## 12. QUICK START (TL;DR)

---

1. Install Google Drive for desktop and sign in; let it sync your **Voice** and **Audition** folders.

2. Create a Google Sheet → **Extensions → Apps Script**.

3. Paste **`Code.gs`** and **`Sidebar.html`**.

4. In the Sheet, open **Voice Tracker → Show Tracker Panel → Settings** and set:

   * Voice Folder ID
   * Audition Folder ID
   * Performer / Stage Name → `John Doe`

5. From the **Voice Tracker** menu:

   * **Refresh Sheet** (build Voice list)
   * **Refresh Auditions** (build Auditions list)
   * **Show Tracker Panel** to use the UI.

6. Use file names like:

   ```text
   John Doe - Character - …
   ```

   so Character auto-detection works cleanly.

7. Use **Updates** to add Final Production links and set statuses.

8. Click **Sync** (menu or sidebar) to propagate Voice statuses into Auditions.

You’re ready to roll. 🎙️
