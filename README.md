# 9702 Physics Exam Room

A self-contained practice application for Cambridge International AS Level Physics (9702), Paper 1. Load your own multiple-choice question papers and mark schemes, assemble a test, answer questions against a timer, and review or download the results. This is an independent practice tool, unaffiliated with Cambridge.

## Files

- **`9702 Physics Exam Room.html`** contains the entire application: HTML, CSS, application JavaScript, and bundled PDF libraries.
- **`README.md`** explains usage, implementation, storage, and limitations.

Question papers and mark schemes are not included. Supply PDFs that you are entitled to use.

## Run locally

No package installation, build step, account, database, or application backend is required. Open the HTML file in a modern browser with JavaScript enabled. If your browser restricts local-file modules or workers, serve this directory with Python:

```sh
cd /workspace/Educational
python3 -m http.server 8000 --bind 127.0.0.1
```

Then open `http://127.0.0.1:8000/9702%20Physics%20Exam%20Room.html` on that machine. Stop the server with Ctrl+C. The server only serves static files; PDF processing and marking happen in the browser. Use a consistent browser profile and address to retain the same local history.

## Practice workflow

1. Drop PDF question papers and mark schemes into the upload area, or choose them with the file picker. Multiple files can be loaded together.
2. Standard filenames such as `9702_s24_qp_12.pdf` and `9702_s24_ms_12.pdf` are paired by session and Paper 1 variant. Assign unmatched keys manually, attach a key to an individual paper, or paste numbered answers such as `1 A 2 C 3 D`.
3. Use **Check questions** to inspect detected question crops and keys, disable sections, or exclude unwanted questions. Verify detection before relying on the test.
4. Choose 40 questions, all eligible questions, or a custom count. Select paper order or a random mixture, optionally leaving out questions previously answered correctly.
5. Choose the Paper 1 timing rate, a custom duration, or an untimed attempt. The default is 75 minutes for 40 questions, scaled proportionally for shorter tests.
6. Answer, navigate, flag questions, and use the review screen before ending the test. Countdown expiry submits automatically. Pause stops timing but still permits reading and answering.
7. Review the score report, optionally enter student name and class, inspect answers, and export a PDF. The export follows the selected results filter.

Scanned PDFs without selectable text can be attempted through the paper's whole-page mode, using an answer sheet and manually supplied key as needed. Individual question timings are not measured in this mode.

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

The HTML has an inline stylesheet, two mount points (`#app` and `#modalRoot`), embedded PDF.js library and worker payloads, bundled jsPDF, and a final application script. Read the final script for application behavior; the large encoded/minified blocks are third-party dependencies.

### Layout and styling

CSS custom properties define the turquoise header, colors, typography, and light/dark palettes. Media queries adapt forms, tables, toolbars, and whole-page layout to smaller screens. The interface uses native controls, visible focus indicators, dialog overlays, and inline SVG icons. Question images retain white backgrounds for legibility.

### PDF loading

`b64Url()` decodes the embedded PDF.js payloads into JavaScript Blob URLs. `loadPdfjs()` dynamically imports the library and configures its worker. `openPdf()` reads a selected file into bytes, opens it with PDF.js (`isEvalSupported: false`), and collects each page's dimensions and text items with coordinates. No remote PDF service or CDN is needed.

### Question detection

`pageLimits()` estimates usable page boundaries while excluding common headers, footers, and page numbers. `pageLines()` groups text fragments into lines. `analysePaper()` searches the left portion of the page for numbered question starts, follows numbering sequences, handles numbering restarts and section headings, and scores candidates partly by detected A–D options.

`buildQuestions()` converts those starts and stop markers into page slices, including continuation pages where possible. It records question numbers, section labels, option letters, and crop boundaries. `questionText()` extracts text from those slices. This is layout-based analysis, not OCR or physics understanding. `classify()` assigns all questions to Physics; it does not infer syllabus topics.

### Answer keys and file matching

`parseKey()` extracts numbered A–D answers and supports segments when numbering restarts. It ignores question numbers above 40 and rejects conflicting duplicate answers within a segment. `handleFiles()` distinguishes papers from keys using filenames and extracted content, loads papers first, then attempts key matching. `ciePaperId()` recognizes standard 9702 session/variant names; `matchKeyToPaper()` otherwise uses filename-token overlap. Ambiguous keys remain available for manual assignment.

`keyFor()` resolves a question's answer from its paper and numbering segment. Missing keys remain unmarked rather than counting as incorrect.

### Rendering questions

`renderPage()` rasterizes PDF pages to canvas at scale 2.2 and maintains a six-page cache. `inkBounds()` scans pixels to locate visible content. `questionImage()` crops question slices, trims whitespace, joins continuation slices, and creates a PNG Blob URL. `pageImage()` provides full-page images for the fallback view. Crops and page images are generated locally and cached in memory.

### State, test selection, and navigation

The central `state` object contains loaded papers, unmatched keys, test configuration, view, zoom, and the current attempt. Paper objects hold PDF handles, extracted text, detected questions, keys, exclusions, and section preferences. Questions receive identifiers based on a paper fingerprint and question label.

`poolFor()` filters eligible questions using paper inclusion, section choices, exclusions, and prior correctness. `shuffle()` implements random ordering. `startModule()` selects questions; `startExam()` initializes answers, flags, timing, and the first screen. Rendering functions replace the relevant interface HTML and attach event handlers. `esc()` escapes text inserted into HTML templates.

### Timing and marking

`timeFor()` calculates duration. `elapsed()` subtracts accumulated pauses from wall-clock time; `tick()` updates the timer every 500 milliseconds and ends timed tests when time expires. `noteTime()` tracks time spent in individual question views, excluding review-screen time and pauses.

`scoreExam()` awards one mark for each correct keyed answer, with no negative marking. An unanswered question with a key earns zero; a question without a key is unmarked and excluded from the marked total. `endExam()` records a summary and updates prior-answer status. Reports distinguish correct, incorrect, unanswered, and unmarked questions and support filters for wrong/unanswered, flagged, and unusually slow responses.

### PDF reports

`requestStudentDetails()` collects a name and class for the current attempt. `exportResultsPdf()` uses bundled jsPDF to generate an A4 report with student details, score, percentage, timing, pause information, and a paginated question-results table. It exports the currently filtered rows, not a copy of the original question paper. The download filename includes the attempt end date.

## Storage and privacy

The application code contains no remote upload or analytics endpoint. Uploaded PDFs, extracted text, answer keys, active answers, and student details are held in memory and must be reloaded or re-entered after refresh. The app warns before leaving an active test but does not implement attempt recovery.

Browser `localStorage` keys use the prefix `cie9702room:`:

| Data | Purpose |
| --- | --- |
| `papers` | Per-paper exclusions, disabled parts, and subject overrides |
| `seen` | Prior correctness status used by the skip-correct option |
| `history` | Up to 60 attempt summaries; setup displays the latest 15 |
| `zoom` | Question display scale |

Student names and classes are not written to the attempt-history store, but they appear in downloaded reports. Browser storage can be cleared or unavailable; storage errors are caught, so persistence may silently stop. Treat reports as personal files when they contain identifying details. Paper fingerprints are convenience hashes, not cryptographic identities.

## Dependencies and maintenance

- **PDF.js 6.2.108**, including its worker, is embedded as Base64 and carries a Mozilla Foundation Apache 2.0 notice.
- **jsPDF 3.0.4** is embedded as JavaScript and carries an MIT license notice.

Keep bundled notices intact. These dependency notices do not establish a license for the application's original code or for imported exam material. No project-wide license is declared here.

There is no package manifest, lockfile, build pipeline, or checked-in automated test suite. For ordinary interface changes, edit the HTML directly. `window.__physics9702` exposes selected analysis functions, state, and PDF helpers for developer self-checks. Avoid editing encoded/minified dependencies manually; library replacement requires keeping the library and worker compatible and preserving their notices.

## Limitations and useful checks

Automatic cropping and key extraction depend on PDF text structure and layout. Scans need the whole-page fallback; unusual columns, numbering, headers, or mark-scheme tables may need manual intervention. Detected answers are not guaranteed correct. Large documents and pre-rendered question images can consume substantial browser memory. This is a practice tool with a freely pausable timer, not a secure examination or proctoring system.

For a functional check, load a known text-based Paper 1 PDF and matching mark scheme, preview crops, answer at least one question correctly and one incorrectly, flag a question, pause/resume, finish, and compare the report against the key. Check whole-page mode separately with a scanned paper. Verify PDF export and local history in the browser you intend to use. Such checks require representative PDFs; none are distributed with this repository.
