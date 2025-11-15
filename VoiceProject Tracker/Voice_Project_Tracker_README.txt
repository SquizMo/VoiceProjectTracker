# VOICE PROJECT TRACKER — README

A simple, script-powered dashboard that scans your Google Drive voice-over folders, lists every file in a Google Sheet, lets you add “Final Production” links, track Current/Past status, and sync statuses with your separate Auditions tab. It also includes a small HTML UI (buttons, tabs, KPIs, Settings) you can open in the Sheet.

---

1. WHAT THIS TOOL DOES

---

* Recursively scans two Google Drive folders:
  • Voice Projects (finished or in-progress recordings)
  • Auditions (raw/alt takes, submissions)

* Writes rows into your Google Sheet with:
  • Folder path
  • File name
  • Character (auto-parsed from file name)
  • Date added
  • MIME type
  • File link
  • Final Production link
  • Status

* Syncs Voice status → Auditions status:
  • Voice "Current" → Auditions "Booked"
  • Voice "Past"    → Auditions "Submitted"
  • Never overwrites "Passed"

* Adds a sidebar UI: filters, quick search, bulk updates, CSV/PDF export, Insights, Settings, and a Donate link/button.

---

2. PREREQUISITES

---

* A Google account
* Google Sheets
* Access to Google Drive (web)
* (Optional, recommended) Google Drive for desktop installed and signed in

---

3. INSTALL GOOGLE DRIVE FOR DESKTOP (TO KEEP LOCAL FILES SYNCED)

---

Windows

1. Search the web for “Google Drive for desktop download” (official Google page).
2. Download and run the installer.
3. Sign in with your Google account.
4. In Drive settings:

   * Choose **Stream files** (recommended) or **Mirror files**.
   * Confirm your local Drive letter (Windows often maps it to G:).
5. You will see Google Drive in File Explorer. Anything in My Drive or Shared drives will sync.

macOS

1. Download Google Drive for desktop from Google’s site.
2. Open the .dmg, drag the app into **Applications**.
3. Sign in with your Google account.
4. In Finder, you will see Google Drive mounted in the sidebar.

Tip: Work from your desktop DAW while Drive syncs to the cloud. The script reads the *cloud* folders—sync keeps the cloud in step with your desktop.

---

4. CREATE THE GOOGLE SHEET + APPS SCRIPT

---

1. Create a new Google Sheet (e.g., **“Voice Project Tracker”**).
2. Click **Extensions → Apps Script**.
3. In the Script Editor:

   * Create a file named **Code.gs** and paste the entire server script
     (everything from `// ========== MAIN SCRIPT ==========` down to the last function).
   * Create a file named **Sidebar.html** and paste the entire HTML block
     (everything from `<!DOCTYPE html>` to `</html>`).
4. Click **Save**.

---

5. CONFIGURE YOUR FOLDER IDS (VIA SETTINGS TAB)

---

The script now stores your folder IDs in **Document Properties** and exposes them through the **Settings** tab in the Tracker Panel UI.

Under the hood, the script uses:

* `getTrackerSettings()` → returns
  `{ voiceFolderId: '...', auditionFolderId: '...' }`
* `getVoiceFolderId()` / `getAuditionFolderId()` → convenience helpers used by all scan/export functions.

On first run, it seeds default IDs (the ones that were originally hard-coded).
You should **override these** with your own folder IDs from the UI:

**How to find a folder ID:**

* In Drive (web), open the folder.
* The URL looks like:
  `https://drive.google.com/drive/folders/1AbCDeFGhijkLMNOPqR...`
* Copy the long string after `/folders/` — that’s the **Folder ID**.

**How to set in the Tracker:**

1. In the Sheet, go to **Voice Tracker → Show Tracker Panel**.
2. In the sidebar, switch to the **Settings** tab.
3. Paste your:

   * **Voice Folder ID** (for finished / in-progress projects)
   * **Audition Folder ID** (for auditions & submissions)
4. Click **Save** (or whatever the Settings button is labeled in your UI).

From that point on:

* `Refresh Sheet` uses the **Voice Folder ID** from Settings.
* `Refresh Auditions` uses the **Audition Folder ID** from Settings.
* CSV/PDF exports also write into the configured Voice folder.

If either ID is missing, affected functions will throw a clear error message telling you to configure it in Settings.

---

6. FILE NAMING FORMAT (CHARACTER AUTO-DETECTION)

---

The script extracts the **Character** from each file’s name using this pattern:

```text
Fred French - <Character Name> - <anything else>.wav
```

In `collectFiles()`, the line is:

```js
var match = fileName.match(/Fred French - (.+?) -/);
```

It looks for `"Fred French - "` followed by the character name, then `" -"`.

Examples that parse correctly:

* `Fred French - Captain Orion - Take 3.wav`    → Character = `Captain Orion`
* `Fred French - Shop Announcer - v2.mp3`       → Character = `Shop Announcer`

If your display name is not **“Fred French”**, choose one of the following:

**Option A — Replace the literal name**
Change the regex to match your name:

```js
var match = fileName.match(/Your Name - (.+?) -/);
```

**Option B — Make it more generic**
If you ensure the file names follow:

```text
<Performer Name> - <Character Name> - <rest>
```

you can generalize the regex and ignore the performer name entirely:

```js
var match = fileName.match(/^[^-]+-\s*(.+?)\s*-/);
```

This captures the text between the first and second hyphen blocks as Character.

**Recommendation:**
Adopt a consistent naming standard, for example:

```text
Your Name - Character - Project - 2025-11-10 - take01.wav
```

If a file does not match the pattern, the script sets **Character** to `"Unknown"`.

---

7. USING THE UI & WHAT EACH BUTTON DOES

---

After saving scripts:

1. Return to the Sheet and refresh.
2. Run any function when prompted and accept permissions.

You will see a **“Voice Tracker”** menu in the Sheet. Menu items:

* **Refresh Auditions**
  Scans the Auditions folder (from Settings) and rewrites the *Auditions* tab.

* **Sync Auditions from Voice**
  Applies the mapping logic:

  * Voice **Current** → Auditions **Booked**
  * Voice **Past** → Auditions **Submitted**
  * Never overwrites **Passed**.

* **Show Tracker Panel**
  Opens the sidebar UI (tabs, KPIs, Settings, etc.).

* **Refresh Sheet**
  Scans the Voice Projects folder (from Settings) and rewrites the main Voice sheet.

* **Export Summary PDF**
  Saves a PDF snapshot of the whole Sheet into your **Voice Projects** folder.

* **Install Auto-Refresh (hourly)**
  Adds a time-based trigger: `updateVoiceProjects` every hour.

* **Remove Auto-Refresh**
  Deletes that trigger.

* **Export Visible Auditions CSV / Voice CSV**
  Exports **only currently visible (filtered)** rows to CSV, and saves into your Voice folder.

### Sidebar (HTML UI) Tabs

* **Dashboard**
  Quick filter buttons for Voice: **Current / Past / All**.

* **Updates**

  * **Single update**:
    Choose a Character, optional specific File URL, then add a Final Production link and/or set Status.
  * **Bulk update**:
    Choose multiple Characters (and optional specific File(s)), then apply one Final link or Status to all.

* **Search**
  Free-text filters for Folder / File / Character plus a Status filter.

* **Insights**
  KPIs and a Recent Activity table compiled from both Voice and Auditions sheets.

* **Auditions**
  Status chips (All / Pending / Submitted / Booked / Passed), quick search box, and a **Sync Status** button.

* **Actions**
  Quick links to refresh the Voice sheet and export a summary PDF.

* **Settings**  🔧

  * **Voice Folder ID**: Google Drive folder ID where your voice project files live.
  * **Audition Folder ID**: Google Drive folder ID where your audition files live.
  * A **Save** button that writes these values into Document Properties so all server functions use them.

---

8. FIRST-RUN CHECKLIST

---

1. **Create the Sheet and scripts** (Code.gs + Sidebar.html).
2. Open the Sheet, go to **Voice Tracker → Show Tracker Panel**.
3. In the sidebar, open **Settings** and paste:

   * Your **Voice Folder ID**
   * Your **Audition Folder ID**
     Then click **Save**.
4. Use file names that match the naming pattern (see section 6).
5. In the **Voice Tracker** menu:

   * Click **Refresh Sheet** (Voice folder).
   * Click **Refresh Auditions** (Audition folder).
6. Open **Show Tracker Panel** and test:

   * **Dashboard**: Current / Past / All
   * **Updates**: add a Final Production link and set Status for at least one file
   * **Auditions**: click **Sync Status**, verify that Booked / Submitted match Voice statuses.
7. (Optional) Install **Auto-Refresh (hourly)**.

---

9. TROUBLESHOOTING

---

**Authorization prompts**

* Run any menu action and approve OAuth permissions.

**“Folder ID is not configured” errors**

* Open **Show Tracker Panel → Settings** and confirm both IDs.
* Make sure there are no extra spaces when you paste the ID.

**No rows appear after Refresh**

* Check the folder IDs in Settings.
* Confirm the Drive folders actually contain files.
* Confirm your account has access (especially for Shared drives).

**Character shows “Unknown”**

* File name did not match the expected pattern.
* Fix the file name or switch to the generic regex in `collectFiles()`.

**Donate link does not open**

* Make sure it is an `<a href="...">` with `target="_blank"` instead of using `window.open()`.

**Auditions status not updating**

* Use **Voice Tracker → Sync Auditions from Voice** (or “Sync Status” in the sidebar).
* Confirm file names in Auditions match those in Voice (same Character + same File Name).

**Performance is slow with very large folders**

* Organize into subfolders to keep lists manageable.
* Consider archiving older projects elsewhere if the list gets huge.

---

10. CUSTOMIZATIONS (OPTIONAL)

---

**Stage name**

* Change the regex to your name or use the generic version (see section 6).

**Status words**

* Auditions: `["Pending", "Submitted", "Booked", "Passed"]`
* Voice:     `["Current", "Past"]`
  If you add new statuses, update:
* Conditional formatting (for colors).
* `syncAuditionsWithProjects()` mapping logic.

**Auto-refresh frequency**

* In `installAutoRefresh()`, change `.everyHours(1)` to `.everyMinutes(15)` if desired.

---

11. RECOMMENDED FILE NAME TEMPLATE

---

```text
Your Name - Character Name - Project Name - 2025-11-10 - take01.wav
```

---

12. QUICK START (TL;DR)

---

1. Install Google Drive for desktop and sign in; let it sync your voice and audition folders.
2. Create a Google Sheet → **Extensions → Apps Script**.
3. Paste **Code.gs** and **Sidebar.html**.
4. In the Sheet, open **Voice Tracker → Show Tracker Panel → Settings** and set:

   * Voice Folder ID
   * Audition Folder ID
5. From the **Voice Tracker** menu:

   * **Refresh Sheet** (build Voice list)
   * **Refresh Auditions** (build Auditions list)
   * **Show Tracker Panel** to use the UI.
6. Use file names like:
   `Your Name - Character - …` so Character auto-detection works.
7. Use **Updates** to add Final Production links and set statuses.
8. Click **Sync** to propagate Voice statuses into Auditions.

You’re ready to roll.
