# Engagement–Performance Gap Screener

A single-file, fully offline web app that helps instructors spot exam scores that are unusually high relative to a student's logged engagement with the course (LMS hours) and the textbook (reading time).

> **This is a screening aid, not evidence of academic misconduct.** A high exam score with low logged engagement is also exactly what a well-prepared student who studies offline, uses a print textbook, or has prior knowledge of the material looks like. Treat a "Review Priority" flag as a reason to look more closely, never as a conclusion.

**Live version:** https://YOUR-USERNAME.github.io/engagement-gap-screener/
**Or download:** save `index.html` and open it in any modern browser. No installation needed.

---

## Privacy and FERPA

The app is designed so that student data never leaves your computer.

- **100% client-side.** All parsing and calculation happen in your browser's memory.
- **No network activity.** There are no external scripts, fonts, CDNs, analytics, or API calls. A Content-Security-Policy meta tag blocks outbound connections.
- **No persistence.** The app uses no cookies, localStorage, sessionStorage, IndexedDB, or service workers. Closing the tab erases everything, and a **Clear all data** button wipes it immediately.
- **Pseudonymization.** An **Anonymize view** toggle replaces names with stable IDs (e.g., `S-014`) for screen sharing. Exports are pseudonymized by default; names are included only if you opt in.

Using the hosted GitHub Pages version is equivalent to opening the file locally. GitHub serves the HTML, but your CSV files are read by the browser and are never uploaded.

**Verify it yourself:** open your browser's developer tools (F12), go to the **Network** tab, load your files, and confirm that no requests are made.

The architecture supports FERPA compliance, but compliance also depends on your own handling of the files and any exported reports. Follow your institution's data policies.

---

## How to use

1. Export three files from your systems (formats below).
2. Open the app and load each file by clicking or drag-and-drop.
3. Review the **Match review** panel. Confirm fuzzy name matches and manually assign any unmatched students.
4. If the exam file contains more than one exam, select the exam to analyze.
5. Adjust thresholds and weights if needed, then review the results table and scatter plot.
6. Optionally export a CSV report or print a summary.

Click **Load demo data** to try the app with synthetic records first.

---

## Input files

Headers are matched case-insensitively. Comma- and tab-delimited files are both detected automatically.

### 1. Course engagement (Blackboard Performance Dashboard)

`Last Name, First Name, Username, Student ID, Overall Grade, Overall Grade Alert, Missed Due Dates, Missed Due Dates Alert, Hours in Course, Last Access Date, Days Since Last Access, Days Since Last Access Alert`

### 2. Textbook engagement (publisher export)

`Student Name, Total Time(seconds), Avg. Session Length (seconds), Sessions, Pages, Annotations, Last Activity, Practices Completed, First Attempt Accuracy`

`Student Name` may be either "First Last" or "Last, First"; the app detects the format for each row.

### 3. Exam scores (Blackboard Grade Center)

`Last Name, First Name, Username, Student ID, Last Access, Availability`, plus exam columns such as:

```
Testing Center - Exam 1 [Total Pts: 200 Score] |6430020
Respondus Monitor - Exam 1 [Total Pts: 200 Score] |6430021
```

- Exam columns are detected by pattern, so Exam 2, Exam 3, and different point totals work without changes.
- Each student's score is taken from whichever modality column is populated. If both columns are populated, the student is flagged as a data conflict.
- Non-numeric entries (`Needs Grading`, `In Progress`, `-`, blank, `Exempt`) are excluded from scoring and listed separately.

---

## Record matching

- Course and exam files are joined on **Username**, falling back to **Student ID**.
- The textbook file has only a name field, so it is joined by **normalized name matching**: case, accents, punctuation, and suffixes are ignored, both name orders are tried, and a fuzzy match is used as a fallback. Fuzzy matches are shown for confirmation.
- **An unmatched student's textbook time is treated as *unknown*, not zero.** These students are scored on course hours only and marked "partial data".

---

## Scoring method

The method is rank-based so it stays robust with small class sizes (roughly 20–60 students).

1. Course hours and textbook hours are converted to within-class **percentile ranks**.
2. **Engagement percentile** is the weighted mean of the two (default 50/50, adjustable).
3. **Exam percentile** is the within-class percentile of the exam score. You can optionally compute it within each testing modality instead.
4. **Gap** = Exam percentile − Engagement percentile.
5. A student receives a **Review Priority** flag only if *all* of these hold (defaults shown; all are adjustable):
   - exam score ≥ 85%
   - exam percentile ≥ 75
   - gap ≥ 50

   | Tier | Gap |
   |---|---|
   | High | ≥ 70 |
   | Moderate | 50–69 |

The app also reports the **Spearman correlation** between exam scores and each engagement measure. If engagement barely predicts scores in your class (|ρ| < 0.2), a warning appears: the gap metric then carries little information.

Context columns (Overall Grade, Missed Due Dates, Practices Completed, First Attempt Accuracy) are displayed but **not** used in the score. Strong practice accuracy alongside a high exam score, for example, weakens the basis for a flag.

---

## Known limitations

- **LMS hours are noisy.** Idle open tabs inflate them; offline study isn't captured at all.
- **Textbook time misses print, library, and shared copies,** as well as study from notes.
- **Prior knowledge** (repeating the course, related coursework, work experience) produces legitimate high scores with low engagement.
- **Small classes amplify noise.** A single student's ranking can shift substantially.
- **Name matching can fail.** Always review the Match review panel before interpreting results.

Validate the tool on a past exam with known outcomes before relying on it. Check your institution's policies on the use of analytics in academic integrity processes.

---

## Technical notes

- A single `index.html` with inline CSS and JavaScript and no dependencies.
- Includes a hand-written CSV parser that handles quoted fields, UTF-8 BOM, CRLF line endings, and tab delimiters.
- Charts are rendered natively, with no charting library.
- Keyboard navigable; flag tiers are shown in text as well as color.

## Contributing

Issues and pull requests are welcome. **Never attach real student data to an issue or pull request.** Use the demo data or synthetic records to reproduce problems.

## License

MIT. See [LICENSE](LICENSE).

Built with assistance from Claude Code.
