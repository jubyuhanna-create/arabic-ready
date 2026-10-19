---
name: arabic-ready
description: 'Makes Arabic output correct the first time. Covers right-to-left layout, connected letter shaping, mixed Arabic/English/number ordering, Arabic fonts and typography, numerals, dates, currency, and natural Arabic wording. Use it whenever the output will contain Arabic text or is meant for Arabic speakers. That includes websites and apps (HTML, CSS, Tailwind, React, Next.js, Vue, React Native, Flutter), Word/PowerPoint/PDF files, charts and images (matplotlib, Pillow, SVG, canvas), emails, SMS/WhatsApp/social posts, markdown, and plain text. Use it when the request is written in Arabic (e.g. "صمم صفحة", "اكتب رسالة", "تقرير", "بالعربي"). Use it when an English request mentions Arabic, RTL, right-to-left, bilingual English/Arabic, i18n for Arabic, or users in Saudi Arabia, the UAE, Egypt, the Gulf, the Levant, North Africa or the MENA region. Also use it to fix broken Arabic: reversed or disconnected letters, boxes instead of glyphs, wrong alignment, or punctuation and numbers on the wrong side.'
---

# Arabic-ready output

Goal: Arabic that renders, reads and feels native on the first attempt, in any output format.

## 1. Decide three things before writing anything

| Decision | Default | Change when |
|---|---|---|
| **Audience locale** | `ar` + region if known (`ar-SA`, `ar-EG`, `ar-AE`, `ar-MA`...) | User names a country/city/market |
| **Digits** | Western `0-9` for UI, prices, phones, code, charts. Arabic-Indic `٠-٩` for formal/editorial text aimed at Egypt, Saudi Arabia, Iraq, the Levant, Sudan, Yemen | User asks, or the brand/region clearly uses `٠-٩`. Maghreb (MA/DZ/TN) always uses `0-9` |
| **Register** | Modern Standard Arabic (فصحى), clear and modern, not classical | User asks for a dialect (Egyptian, Gulf, Levantine...) or the channel is casual (WhatsApp promos, social posts): see `references/writing.md` |

Never mix digit systems in one piece of output.

## 2. Core rules (apply to every output type)

1. **Declare direction at the root, once.** Then let everything inherit.
   ❌ `<html>` + `style="direction:rtl"` sprinkled on elements
   ✅ `<html lang="ar" dir="rtl">` · docx `<w:bidi/>` + `<w:rtl/>` · pptx `rtl="1"` · Flutter `Directionality(textDirection: TextDirection.rtl)` / `locale: Locale('ar')`
2. **Use logical directions, never left/right.** "Start" is the right side in Arabic.
   ❌ `margin-left`, `ml-4`, `text-left`, `left: 0`, `EdgeInsets.only(left:)`
   ✅ `margin-inline-start`, `ms-4`, `text-start`, `inset-inline-start: 0`, `EdgeInsetsDirectional.only(start:)`
3. **Isolate LTR islands** (phone numbers, emails, URLs, code, English names next to numbers, negative numbers).
   ❌ `اتصل على +966 50 123 4567.` → renders as `4567 123 50 966+`
   ✅ `اتصل على <bdi dir="ltr">+966 50 123 4567</bdi>.` · plain text: wrap in LRI…PDI (`⁦…⁩`) or put LRM `‎` before `-5`
4. **Inputs for phone, email, URL, code, IBAN, passwords stay `dir="ltr"`** even inside an RTL form (align them with `text-end`).
5. **Mirror directional icons only.** Back/forward arrows, chevrons, send, reply, undo/redo, progress direction flip. Play/pause, media controls, checkmarks, clocks, search, logos, brand marks and numbers do not.
6. **Typography:** use an Arabic-designed font (list in `references/typography.md`), line-height ≥ 1.6 for body text, and leave room above and below for tashkeel. **Never** apply letter-spacing/`tracking-*`, `uppercase`, `italic` or tight `overflow:hidden` heights to Arabic.
7. **Shaping: raw text to shaping renderers, shaped text to the rest. Never both.**
   Browsers, Word, LibreOffice, PowerPoint, WeasyPrint, Chromium PDF, **matplotlib ≥ 3.11**, **Pillow with libraqm**, **fpdf2 with `set_text_shaping()`** shape Arabic themselves: pass the raw string.
   reportlab, PyMuPDF `insert_text`, OpenCV, matplotlib ≤ 3.10, Pillow without raqm draw glyph by glyph: pass `prepare(text)` from `scripts/shape_text.py`.
   Reshaping text for a renderer that already shapes reverses it again (verified in matplotlib 3.11 and Pillow+raqm).
8. **Arabic punctuation:** `،` comma, `؛` semicolon, `؟` question mark, `«»` quotes. Keep `.` and `!`. Use no space before punctuation and one space after.
9. **Write Arabic natively, don't translate word-for-word.** Imperative verbs for buttons (`أرسل`, `احفظ`, `ابدأ الآن`), and no English word order. See `references/writing.md`.
10. **Format numbers/dates/currency with the locale, not string concatenation.** Use `Intl.NumberFormat('ar-SA', ...)` / `Intl.DateTimeFormat('ar-EG-u-nu-latn', ...)`, and always set `nu` (digits) and `ca` (calendar) explicitly.

## 3. Load the reference for your output type

Read only what the task needs:

| Output | Read |
|---|---|
| HTML, CSS, Tailwind, React, Next.js, Vue, React Native, Flutter, email HTML | `references/web.md` + `references/bidi.md` |
| Mixed Arabic + English/numbers anywhere (incl. markdown, plain text, SMS, JSON strings) | `references/bidi.md` |
| Fonts, line-height, tashkeel, weights, justification | `references/typography.md` |
| Word (.docx), PowerPoint (.pptx), PDF | `references/documents.md` |
| matplotlib, Pillow, SVG, canvas, OpenCV, Plotly, any image with text | `references/charts-images.md` |
| Copywriting, translation, dialects, numerals, dates, currency, names, Hijri | `references/writing.md` |

## 4. What flips in RTL and what doesn't

| Flips (mirror the layout) | Stays the same |
|---|---|
| Reading order, text alignment, nav order, sidebar side | Phone numbers, credit cards, IBANs (always LTR) |
| Breadcrumbs, pagination, steppers: "next" points left. Text `›` `>` `(` mirror automatically, but `→` and SVG icons do not | Code blocks, terminal output, URLs, emails |
| Back/next arrows, chevrons, carousels/slider swipe direction | Time-series chart x-axis (time runs left→right). Category axes may flip |
| Table column order (first column on the right) | Media controls (play ▶, fast-forward), clocks, checkmarks |
| Progress bars and range sliders fill from the right | Logos, brand marks, photos, maps |
| Form labels on the right, close button `×` on the left of a dialog | Math expressions and numbers themselves (`1,234.5` stays `1,234.5`) |
| Toast/drawer slide-in side, badges on avatars, list bullets | Keyboard shortcuts (`Ctrl+S`) |

## 5. Before you finish

1. **Run the linter** on any code you produced (it needs only the Python standard library):
   `python <skill-dir>/scripts/rtl_check.py <file-or-folder>`
   Fix every `error`. Fix `warning`s unless they are intentional, and mark intentional ones with an `rtl-ok` comment.
2. **Render and look** when you can, then check that the letters are connected, the first word is on the right, punctuation sits at the left end, and numbers read in the right order:
   - HTML: `python -m playwright screenshot --full-page file://$PWD/index.html shot.png` (or `npx playwright screenshot …`). If no browser is available, `weasyprint index.html out.pdf` gives a static render.
   - docx/pptx: `soffice --headless --convert-to pdf file.docx`, then rasterize the PDF.
   - PDF → PNG: `python -c "import pymupdf; pymupdf.open('f.pdf')[0].get_pixmap(dpi=110).save('f.png')"`
   - Images/charts: open the PNG you wrote.
3. **If scripts can't run** (no Python, no shell), go through this checklist manually:
   - [ ] root `dir="rtl"` + `lang="ar…"` (or the format's equivalent)
   - [ ] no `left`/`right` in spacing, positioning or alignment
   - [ ] phones/emails/URLs/code isolated and LTR
   - [ ] directional icons mirrored, others untouched
   - [ ] Arabic font loaded, line-height ≥ 1.6, no letter-spacing, italic or uppercase
   - [ ] image/PDF text: shaping path matches the renderer (rule 7)
   - [ ] `،` `؛` `؟` used, one digit system, natural MSA (or the requested dialect)

## Scripts

- `scripts/rtl_check.py <path>`: lints HTML/CSS/Tailwind/JSX/TSX/Vue/Svelte/React Native/Flutter for RTL bugs. Prints `file:line`, the problem and the exact fix. Exit code 1 on errors. Stdlib only.
- `scripts/shape_text.py`: `prepare(text, target)` for renderers that don't shape Arabic. It detects matplotlib/Pillow native support and returns the text unchanged when shaping is not needed. It uses `arabic-reshaper` + `python-bidi` (pip-installed on demand) or falls back to a built-in stdlib shaper. `python shape_text.py --check` reports what this machine needs.
