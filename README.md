# Recovering a Forgotten PDF Password with John the Ripper (Johnny GUI)

This document walks through the workflow shown in the screenshots: extracting the password hash from a locked PDF and recovering the password using **John the Ripper** (jumbo build) with the **Johnny** graphical front-end.

> ⚠️ **Use responsibly.** Only run this process on files you own or are explicitly authorized to test. Recovering passwords on files you don't have permission to access is illegal in most jurisdictions.

---

## Overview

| Step | Tool | Purpose |
|------|------|---------|
| 1 | openwall.com/john | Download John the Ripper (jumbo) |
| 2 | openwall.info/wiki/john/johnny | Download Johnny (GUI for John) |
| 3 | onlinehashcrack.com – PDF Hash Extractor | Convert the locked PDF into a crackable hash (`pdf2john` format) |
| 4 | Johnny | Point Johnny at the John executable, load the hash, run the attack |
| 5 | Johnny | Review the cracked password and open the PDF |

---

## Step 1 — Download John the Ripper

Go to **openwall.com/john/** and download the jumbo build for your OS. On Windows, grab the **1.9.0-jumbo-1 64-bit Windows binaries (zip)** and extract it, e.g. to:

```
C:\Users\<you>\Downloads\john-1.9.0-jumbo-1-win64\
```

## Step 2 — Download Johnny (GUI)

Johnny is a graphical front-end for John the Ripper, making it easier to manage hashes and attacks without the command line. Download it from **openwall.info/wiki/john/johnny** (`johnny_2.2_win.zip` for Windows) and extract it.

## Step 3 — Identify the locked file

The target file in this example is `My Locked PDF1.pdf`. Opening it prompts:

> *"This file is password protected. Please enter a password to open the file."*

Since the password is unknown, it needs to be recovered rather than guessed manually.

## Step 4 — Extract the password hash from the PDF

John the Ripper can't attack the PDF file directly — it needs a hash extracted in the `pdf2john` format first.

1. Go to **onlinehashcrack.com/tools-pdf-hash-extractor.php**.
2. Under **Upload an encrypted PDF**, choose `My Locked PDF1.pdf` and upload it.
3. The tool extracts a hash string such as:

   ```
   $pdf$4*4*128*-1028*1*16*ca7f72f11459cba469f1005a8765ed51*32*f32d8fa1bfbe...
   ```

4. Copy this hash — it's exactly the format John the Ripper's `pdf2john.pl` script would produce locally.

   > Note: since this uploads the file to a third-party site, prefer running the bundled `pdf2john.py`/`pdf2john.pl` script that ships inside the John the Ripper `run` folder if you'd rather not send the file externally.

## Step 5 — Set up Johnny

1. Launch **Johnny**.
2. Go to **Settings** and click **Browse** next to *"John the Ripper executable"*.
3. Navigate to the extracted John folder, e.g.:

   ```
   john-1.9.0-jumbo-1-win64\run\
   ```

4. Select the `john` executable (or `john.exe`) and click **Open**.
5. Johnny should confirm it detected the binary, e.g.:

   ```
   Detected John the Ripper 1.9.0-jumbo-1 OMP [cygwin 64-bit x86_64 AVX2 AC]
   ```

## Step 6 — Load the hash into Johnny

1. Save the extracted hash string into a text file (e.g. `pdf_hash.txt`).
2. In Johnny, use **Open password file** and select that text file.
3. The **Passwords** tab will show one entry with:
   - **User:** `?`
   - **Hash:** `$pdf$4*4*128*-...`
   - **Formats:** `PDF`
   - **Password:** *(empty — not yet cracked, 0% progress)*

## Step 7 — Run the attack

1. Click **Start new attack**.
2. Johnny calls John the Ripper in the background to try password candidates (dictionary, incremental, or rule-based modes depending on configuration).
3. Progress is shown in the bottom status bar, e.g.:

   ```
   0% (0/1: 0 cracked, 1 left) [format=PDF]
   ```

## Step 8 — Review the cracked password

Once John finds a match, Johnny updates the row:

```
100% (1/1: 1 cracked, 0 left) [format=PDF]
Password: good-luck
```

The password column now shows the recovered password (`good-luck` in this example), highlighted and ready to copy.

## Step 9 — Open the PDF

1. Open `My Locked PDF1.pdf` again.
2. At the **"Enter a password"** prompt, type the recovered password.
3. Click **Open file** — the PDF unlocks and displays its contents.

---

## Summary

```
Locked PDF  →  pdf2john hash extraction  →  Johnny + John the Ripper attack  →  Recovered password  →  Unlocked PDF
```

This same flow (hash extraction → John/Johnny attack → recovered password) applies to many other password-protected formats John the Ripper supports — ZIP, Office documents, SSH keys, and more — by using the matching `*2john` extraction script for that file type.
