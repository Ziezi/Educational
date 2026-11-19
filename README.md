# Physics Exam Rooms

Two self-contained practice applications for Cambridge Physics multiple-choice papers. Load question papers and mark schemes, assemble a test, answer questions against a timer, and review or download the results. These independent practice tools are unaffiliated with Cambridge.

| Application | Papers | Standard timing |
| --- | --- | --- |
| [0625 Physics Exam Room](0625%20Physics%20Exam%20Room.html) | Cambridge IGCSE Physics: Paper 1 (Core), Paper 2 (Extended), or both | 40 questions in 45 minutes |
| [9702 Physics Exam Room](9702%20Physics%20Exam%20Room.html) | Cambridge International AS Level Physics: Paper 1 | 40 questions in 75 minutes |

Both apps retain PDF loading and question previews, manual answer keys, question exclusions, ordered or random selection, timers, pause/resume, answer review and flags, scoring, history, whole-page mode, and PDF result export. Standard timing scales proportionally for shorter tests; custom and untimed attempts are also available.

## Files

- **`0625 Physics Exam Room.html`** contains the complete IGCSE Core/Extended application, including HTML, CSS, JavaScript, and bundled PDF libraries.
- **`9702 Physics Exam Room.html`** contains the complete AS Level application with its own bundled PDF libraries.
- **`README.md`** explains usage, implementation, storage, and limitations.

The repository includes a 9702 sample question paper and mark scheme. IGCSE 0625 sample PDFs are not included. Supply additional PDFs that you are entitled to use.

## Run locally

No package installation, build step, account, database, or application backend is required. Open the HTML file in a modern browser with JavaScript enabled. If your browser restricts local-file modules or workers, serve this directory with Python:

```sh
cd /workspace/Educational
python3 -m http.server 8000 --bind 127.0.0.1
```

Then open either application on that machine:

- IGCSE: `http://127.0.0.1:8000/0625%20Physics%20Exam%20Room.html`
- AS Level: `http://127.0.0.1:8000/9702%20Physics%20Exam%20Room.html`

Stop the server with Ctrl+C. The server only serves static files; PDF processing and marking happen in the browser. Use a consistent browser profile and address to retain the same local history.

## Practice workflow

1. Drop PDF question papers and mark schemes into the upload area, or choose them with the file picker. Multiple files can be loaded together.
2. Standard filenames pair papers and keys by session, paper number, and variant. For IGCSE Core, use names such as `0625_s24_qp_11.pdf` and `0625_s24_ms_11.pdf`; for Extended, use `0625_s24_qp_21.pdf` and `0625_s24_ms_21.pdf`. AS Level uses names such as `9702_s24_qp_12.pdf` and `9702_s24_ms_12.pdf`. Assign unmatched keys manually, attach a key to an individual paper, or paste numbered answers such as `1 A 2 C 3 D`.
3. Use **Check questions** to inspect detected question crops and keys, disable sections, or exclude unwanted questions. Verify detection before relying on the test.
4. In the IGCSE app, choose **Paper 1 (Core)**, **Paper 2 (Extended)**, or **Papers 1 & 2 (Core and Extended)**. Then choose 40 questions, all eligible questions, or a custom count. Select paper order or a random mixture, optionally leaving out questions previously answered correctly.
5. Choose the standard timing rate, a custom duration, or an untimed attempt. The default is **45 minutes for 40 questions in IGCSE** and **75 minutes for 40 questions in AS Level**, scaled proportionally for shorter tests.
6. Answer, navigate, flag questions, and use the review screen before ending the test. Countdown expiry submits automatically. Pause stops timing but still permits reading and answering.
7. Review the score report, optionally enter student name and class, inspect answers, and export a PDF. The export follows the selected results filter.

Scanned PDFs without selectable text can be attempted through the paper's whole-page mode, using an answer sheet and manually supplied key as needed. Individual question timings are not measured in this mode.

### IGCSE paper selection

The 0625 app detects Core or Extended from standard filenames, the first page's component code (such as `0625/11` or `0625/21`), or an identifiable paper title. Check the **paper type** selector below each loaded filename and correct it when necessary. For an unidentified paper, choose its type manually; unassigned papers are excluded from tests and cannot start whole-page mode. Papers 3–6 are outside this app's multiple-choice workflow.

The paper-selection filter controls which questions enter an ordinary test. **Use** checkboxes, section choices, exclusions, and the skip-correct option still apply. Selecting both types allows a mixed Core/Extended practice test; it is not an official combined examination. A paper's **Whole pages** action starts that individual paper, using its assigned type and the selected timing mode.

Paper-type choices are saved with that paper's preferences. Standard key matching keeps Core and Extended keys separate, including when both are uploaded together. Unmatched or ambiguously named keys remain available for manual assignment.

### Keyboard shortcuts

These apply in the normal question view, outside text fields and dialogs; whole-page mode does not use these exam shortcuts.

| Key | Action |
| --- | --- |
| A–D | Select an answer |
| Right / Left arrow | Next / previous question |
| M | Toggle the review flag |
| R | Open review |
| P | Pause / resume timing |

## Code organization

Each HTML file has an inline stylesheet, two mount points (`#app` and `#modalRoot`), embedded PDF.js library and worker payloads, bundled jsPDF, and a final application script. Read the final script for application behavior; the large encoded/minified blocks are third-party dependencies.

### Layout and styling

CSS custom properties define the turquoise header, colors, typography, and light/dark palettes. Media queries adapt forms, tables, toolbars, and whole-page layout to smaller screens. The interface uses native controls, visible focus indicators, dialog overlays, and inline SVG icons. Question images retain white backgrounds for legibility.

### PDF loading

`b64Url()` decodes the embedded PDF.js payloads into JavaScript Blob URLs. `loadPdfjs()` dynamically imports the library and configures its worker. `openPdf()` reads a selected file into bytes, opens it with PDF.js (`isEvalSupported: false`), and collects each page's dimensions and text items with coordinates. No remote PDF service or CDN is needed.

### Question detection

`pageLimits()` estimates usable page boundaries while excluding common headers, footers, and page numbers. `pageLines()` groups text fragments into lines. `analysePaper()` searches the left portion of the page for numbered question starts, follows numbering sequences, handles numbering restarts and section headings, and scores candidates partly by detected A–D options.

`buildQuestions()` converts those starts and stop markers into page slices, including continuation pages where possible. It records question numbers, section labels, option letters, and crop boundaries. `questionText()` extracts text from those slices. This is layout-based analysis, not OCR or physics understanding. `classify()` assigns all questions to Physics; it does not infer syllabus topics.

### Answer keys and file matching

`parseKey()` extracts numbered A–D answers and supports segments when numbering restarts. It ignores question numbers above 40 and rejects conflicting duplicate answers within a segment. `handleFiles()` distinguishes papers from keys using filenames and extracted content, loads papers first, then attempts key matching. `ciePaperId()` recognizes the app's standard syllabus/session/paper/variant names (0625 Papers 1 and 2, or 9702 Paper 1); `matchKeyToPaper()` otherwise uses filename-token overlap. Ambiguous keys remain available for manual assignment.

`keyFor()` resolves a question's answer from its paper and numbering segment. Missing keys remain unmarked rather than counting as incorrect.

### Rendering questions

`renderPage()` rasterizes PDF pages to canvas at scale 2.2 and maintains a six-page cache. `inkBounds()` scans pixels to locate visible content. `questionImage()` crops question slices, trims whitespace, joins continuation slices, and creates a PNG Blob URL. `pageImage()` provides full-page images for the fallback view. Crops and page images are generated locally and cached in memory.

### State, test selection, and navigation

The central `state` object contains loaded papers, unmatched keys, test configuration, view, zoom, and the current attempt. Paper objects hold PDF handles, extracted text, detected questions, keys, exclusions, and section preferences. Questions receive identifiers based on a paper fingerprint and question label.

`poolFor()` filters eligible questions using paper inclusion, section choices, exclusions, and prior correctness. In the 0625 app it also requires an assigned paper type and applies the Core/Extended/both selection. `shuffle()` implements random ordering. `startModule()` selects questions; `startExam()` initializes answers, flags, timing, and the first screen. Rendering functions replace the relevant interface HTML and attach event handlers. `esc()` escapes text inserted into HTML templates.

### Timing and marking

`timeFor()` calculates duration. `elapsed()` subtracts accumulated pauses from wall-clock time; `tick()` updates the timer every 500 milliseconds and ends timed tests when time expires. `noteTime()` tracks time spent in individual question views, excluding review-screen time and pauses.

`scoreExam()` awards one mark for each correct keyed answer, with no negative marking. An unanswered question with a key earns zero; a question without a key is unmarked and excluded from the marked total. `endExam()` records a summary and updates prior-answer status. Reports distinguish correct, incorrect, unanswered, and unmarked questions and support filters for wrong/unanswered, flagged, and unusually slow responses.

### PDF reports

`requestStudentDetails()` collects a name and class for the current attempt. `exportResultsPdf()` uses bundled jsPDF to generate an A4 report with student details, score, percentage, timing, pause information, and a paginated question-results table. It exports the currently filtered rows, not a copy of the original question paper. The download filename includes the attempt end date.

## Storage and privacy

The application code contains no remote upload or analytics endpoint. Uploaded PDFs, extracted text, answer keys, active answers, and student details are held in memory and must be reloaded or re-entered after refresh. The app warns before leaving an active test but does not implement attempt recovery.

Browser `localStorage` keys use **`cie0625room:`** for IGCSE and **`cie9702room:`** for AS Level. The apps keep their preferences, prior answers, history, and zoom separate:

| Data | Purpose |
| --- | --- |
| `papers` | Per-paper exclusions, disabled parts, subject overrides, and IGCSE paper-type assignments |
| `seen` | Prior correctness status used by the skip-correct option |
| `history` | Up to 60 attempt summaries; setup displays the latest 15 |
| `zoom` | Question display scale |

Student names and classes are not written to the attempt-history store, but they appear in downloaded reports. Browser storage can be cleared or unavailable; storage errors are caught, so persistence may silently stop. Treat reports as personal files when they contain identifying details. Paper fingerprints are convenience hashes, not cryptographic identities.

## Dependencies and maintenance

- **PDF.js 6.2.108**, including its worker, is embedded as Base64 and carries a Mozilla Foundation Apache 2.0 notice.
- **jsPDF 3.0.4** is embedded as JavaScript and carries an MIT license notice.

Keep bundled notices intact. These dependency notices do not establish a license for the application's original code or for imported exam material. No project-wide license is declared here.

There is no package manifest, lockfile, build pipeline, or checked-in automated test suite. For ordinary interface changes, edit the HTML directly. `window.__physics0625` and `window.__physics9702` expose selected analysis functions, state, and PDF helpers for developer self-checks. The 0625 helper also exposes paper detection, matching identifiers, question pooling, and timing. Avoid editing encoded/minified dependencies manually; library replacement requires keeping the library and worker compatible and preserving their notices.

## Limitations and useful checks

Automatic cropping and key extraction depend on PDF text structure and layout. Scans need the whole-page fallback; unusual columns, numbering, headers, or mark-scheme tables may need manual intervention. Detected answers are not guaranteed correct. Large documents and pre-rendered question images can consume substantial browser memory. This is a practice tool with a freely pausable timer, not a secure examination or proctoring system.

For a functional check, load a known text-based supported multiple-choice PDF and matching mark scheme, preview crops, answer at least one question correctly and one incorrectly, flag a question, pause/resume, finish, and compare the report against the key. Check whole-page mode separately with a scanned paper. Verify PDF export and local history in the browser you intend to use. For IGCSE, load both Core and Extended papers with their corresponding keys and confirm each individual filter and the combined selection. Verify that 40 questions allow 45 minutes and that unrecognized filenames can be assigned manually. Representative IGCSE PDFs are not distributed with this repository.

The IGCSE app was checked in Chromium using generated two-question Core and Extended PDFs: batch key matching, separate and combined question pools, proportional timing, question images, answer entry, flags, pause/resume, scoring, PDF report download, separate history, manual paper-type assignment, and whole-page mode passed. This is a synthetic smoke check, not validation of cropping across real exam layouts or a Safari compatibility guarantee.
