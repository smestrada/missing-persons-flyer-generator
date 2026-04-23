# Known Limitations

This document details five known limitations of the Missing Persons Flyer Generator and workarounds for each.

## 1. Re-saved PDFs Lose Embedded Case Data

### What Happens

When you export a flyer as PDF, the app embeds a JSON backup of all case data in the PDF's metadata (Keywords field). This allows another deputy to drag the PDF back into the tool and reload all fields to edit or update the flyer.

**However**, if a deputy opens the exported PDF in Adobe Acrobat, Preview, or another PDF editor and re-saves the file, the metadata is stripped. The next time someone tries to load that PDF back into the tool, the case data will be lost.

### Why This Happens

PDF metadata is fragile. Most PDF editors either don't preserve metadata or allow users to strip it (intentionally or accidentally). It's a limitation of the PDF format itself.

### Workaround

**The app always downloads two files alongside the PDF:**

- `[name]_[date].pdf` — the flyer (for printing and sharing)
- `[name]_[date]_backup.json` — a JSON backup of all case data

**Always keep the `.json` backup file.** If the PDF gets re-saved, drag the JSON file into the tool instead. The case data will load perfectly, and the deputy can update and re-export.

### Best Practice

- Deputies should share the PDF for viewing/printing.
- If editing is needed, share the `.json` backup alongside the PDF or save it to the shared drive.
- Older PDFs without metadata can't be reloaded, but new exports always include the backup.

---

## 2. Large Photos Can Hit localStorage Limits

### What Happens

The app auto-saves all form state (including photos) to browser localStorage. Browsers typically allow 5–10MB per domain.

On older browsers or storage-restricted kiosks (e.g., government machines with tight disk quotas), localStorage limits can be hit. On modern browsers, this is rarely a concern.

### Why This Happens

The app downscales photos on upload to max 1200px at JPEG 85% quality, typically reducing phone photos from multi-megabytes to ~200-400KB each. Four photos total roughly 1-1.5MB, well under the 5–10MB limit on modern browsers. 

However, older browser versions may have tighter limits, and some organization-managed devices artificially restrict localStorage further. In those cases, the quota can be exceeded.

### Workaround

**The app detects quota exceeded and falls back gracefully:**

- Photos are stripped from the auto-save, but all other form data (name, age, etc.) is preserved.
- A warning toast appears: "⚠ Photos too large to auto-save — export before closing."
- **Export immediately to PDF/PNG** — this bypasses localStorage and writes directly to disk.
- If the browser crashes, form text is safe; photos might be lost. But exported PDFs are permanent.

### Best Practice

- On modern browsers with typical configurations, this is not a concern.
- If you're on an older device or see the warning, export to PDF/PNG before closing the browser.
- The `.json` backup file includes all photos, so if you download it, you have a complete backup.
- If your organization restricts storage, talk to your IT team about adjusting localStorage limits for law enforcement tools.

---

## 3. Shared MDCs Show Last User's Draft

### What Happens

Deputies use mobile data computers (MDCs) on patrol, sometimes shared between shifts. When a deputy opens the flyer tool, it auto-loads the previous deputy's draft from localStorage.

If the last deputy didn't click "Clear Form," the next deputy sees a partially-filled flyer from someone else's case.

### Why This Happens

Auto-save persists to browser localStorage, which is shared at the machine level, not the user level. MDCs are shared devices, so each machine has one localStorage per browser.

### Workaround

**Deputies should click "Clear Form" when finished.**

This:

- Clears all fields
- Deletes all photos
- Resets photo positioning
- Removes the browser's auto-save cache

Encourage this as a habit: "Start a new flyer → fill it out → export → **Clear Form** → next shift begins clean."

### Best Practice

- Post a reminder near MDCs: "Clear the form when done with your flyer."
- If a flyer loads with unexpected data, it's from the previous user — clear and start fresh.
- There's no per-user login in this tool (by design, to keep it simple), so procedural discipline is key.

---

## 4. Single-Flyer Session

### What Happens

The app holds only one flyer in memory at a time. You can't:

- Have multiple cases open at once
- Compare two flyers side-by-side
- Build a case library or archive
- View past flyers without reloading them

If you want to switch to a different case, you must export the current one (to save it), clear the form, and start the new case.

### Why This Happens

The tool is designed for one-off flyer generation during an active shift. It's not a case management system. A deputy fills out one flyer, exports it, and moves on. Adding multi-case memory would require a database backend, which breaks the "single file, no backend" design goal.

### Workaround

- Export every flyer before starting a new one (PDF or JSON backup).
- Store PDFs in a shared folder for agency records.
- If you need to edit an old case later, load the PDF or `.json` file back in.

### Best Practice

- Save all exported flyers to a shared drive organized by date or case number.
- Use the PDF as the "official" record; keep the `.json` backup as a restore point.
- If you're running many cases in one shift, batch them: create all the flyers, export each, clear, repeat.

---

## 5. Agency Colors Aren't Auto-Validated for WCAG Contrast

### What Happens

The app relies on the agency IT staff to set appropriate brand colors in the CSS variables:

```css
:root {
  --agency-primary: #1e3a5f;   /* You set this */
  --agency-accent:  #94a3b8;   /* You set this */
}
```

If you set a very light primary color (e.g., `#ffff99` yellow) and white text, contrast could fail WCAG AA standards. The flyer would still generate and export, but it might be hard to read in print or fail accessibility audits.

### Why This Happens

Validating contrast at runtime is expensive (color-to-contrast is a complex algorithm). The app assumes the agency has set sensible colors. Default colors in the code are tested and compliant, but custom colors aren't validated.

### Workaround

**Use the provided default colors or test your colors before deploying:**

1. **Use the defaults** — `#1e3a5f` (agency primary) and `#94a3b8` (accent) are tested and WCAG AA compliant.
2. **If you customize**, test contrast with a tool like:
   - [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/)
   - [WCAG Color Contrast Analyzer](https://www.tpgi.com/color-contrast-checker/)
   - **Chrome/Edge DevTools** — right-click on any element → Inspect → Styles panel shows contrast ratio automatically
3. **Test both directions:**
   - Text on primary background (e.g., white text on your primary)
   - Text on white/accent backgrounds

### Best Practice

- Stick with the default color scheme unless you have a brand standard.
- If you must change colors, pick high-contrast pairs (dark on light, bright on dark).
- Print a test flyer before rolling out to deputies. If it's hard to read, adjust.
- Remember: a missing person flyer needs to be legible. Branding is secondary.

---

## Summary

| Limitation              | Impact                       | Workaround                                                 |
| ----------------------- | ---------------------------- | ---------------------------------------------------------- |
| Re-saved PDFs lose data | Can't reload old PDFs        | Use `.json` backup file; best practice: don't re-save PDFs |
| Photos hit localStorage | Auto-save might drop photos  | Export to PDF immediately if uploading multiple photos     |
| Shared MDCs             | Last user's draft persists   | Click "Clear Form" when done                               |
| Single-flyer session    | Can't multi-task cases       | Export each flyer before starting the next one             |
| Colors not validated    | Potential readability issues | Test colors before deploying; use defaults if unsure       |

None of these are bugs; they're intentional trade-offs to keep the tool simple, fast, and deployable with zero infrastructure. If any of these are deal-breakers for your agency, please [open an issue](https://github.com/smestrada/missing-persons-flyer-generator/issues).
