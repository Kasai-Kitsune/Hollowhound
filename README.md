# Hollowhound

A local, browser-based tool that scans a folder you pick, finds every empty file and empty folder inside it, and lets you review and delete them. Nothing is uploaded anywhere, it all runs in your browser against the folder you grant access to.

## Files

- `index.html` - landing page, links to the scanner
- `scanner.html` - the actual tool (picker, scan, review list, delete)
- `logo.png` - app icon

## Why it can't just scan your whole device at once

This is a browser limitation, not a missing feature. Due to the huge minefield of security restrictions, the scanner uses the File System Access API (`showDirectoryPicker`), and by design that API only ever grants access to **one folder tree, chosen manually, per session**. There is no API call that means "give me the device root" or "give me every user folder." Browsers deliberately block that, because a webpage silently getting read/write access to an entire filesystem is a massive security hole. Every browser that implements this API works the same way.

Practical result:

- You have to pick a folder yourself each time.
- If you want to sweep both your DCIM folder AND Downloads, that's two separate picks and two separate scans, the tool can't hold access to two unrelated trees at once.
- On some mobile setups (including wrapped APKs like Median) folder access can be even more restricted than desktop Chrome/Edge, since the OS layer adds its own permission boundary on top of the browser's. I can't access my downloads folder for example. 

So the "inconvenience" is really the tradeoff for the tool not needing (desktop) installation, accounts, or any special permissions beyond what you explicitly hand it folder by folder.

## ⚠️ Use this carefully

This tool permanently deletes files and folders. There is no recycle bin, no undo, no trash folder it moves things to first. `removeEntry` in the File System Access API is a hard delete.

Before you hit **Wipe all** or **Delete selected**:

- **Skim the list yourself.** The "protected placeholders" logic (`.gitkeep`, `.nomedia`, `__init__.py`, `.env*`, `.git`/`node_modules` trees, etc.) only knows about common, well-known conventions. It has no idea what *your* specific projects or any apps rely on. If you've got something that uses an empty file as a sentinel, a lock flag, a build marker, or anything with a name that isn't on that protected list, this tool will not recognize it and will offer it up for deletion just like true junk.
- **You might not remember every placeholder from memory**, and that's exactly the risk here: an empty file that looks like leftover junk today might be something a script, build tool, data backups, or app checks for later. When in doubt, leave it unchecked rather than assume it's safe.
- **Re-scan after deleting** before doing anything else, since deleting files can turn previously non-empty folders into empty ones on the next pass.

##  ‼️ Common-sense "don't delete this" notes 

- Anything inside `.git`, `node_modules`, `.svn`, `.hg` - the tool already skips these trees, don't try to force them in.
- Empty files that are actually feature flags or markers for an app/build system (not just the common ones the tool already protects).
- Empty folders that a program expects to exist even when unused (some apps break if a required-but-empty folder disappears, since the folder itself is the thing being checked for, not its contents).
- Anything you didn't create and don't recognize the purpose of. Empty is not the same as pointless.

## What it actually does well

- Two-pass scan (count, then walk) so the progress bar shows a real percentage, not a fake animation.
- Recurses through folders, flags empty files, and flags folders that end up empty once their contents are accounted for.
- Separates results into empty files, empty folders, and protected placeholders, with protected items excluded from "Wipe all" and unchecked by default.
- Everything stays local. No network calls, no analytics, no data leaving the device.
