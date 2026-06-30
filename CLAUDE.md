# Vienna Piano Summer School — Project Context

## What this is
A single static HTML file (`index.html`) — parent-facing schedule page for Eleny Piano Studio's
Vienna summer school. No build step, no backend, no dependencies beyond a Google Fonts CDN link
(with system-font fallback). Self-hosted on GitHub Pages.

## Hosting
- Public GitHub repo (free Pages tier requires public), Settings → Pages → deploy from branch → /root
- Custom domain (in progress): `vienna26.elenypiano.studio`
  - CNAME DNS record → `<username>.github.io`
  - CNAME *file* in repo root — auto-created by GitHub when the domain is typed into Settings → Pages
- Marijus is handling DNS/repo creation himself
- GitHub MCP connector not available in claude.ai yet — file changes go via manual upload/download
  or Claude Code on Marijus's machine

## Data source (do not re-guess any of this — it was extracted, not assumed)
- Original schedule: Google Sheet "Vienos kursų klasės - pradėti scheduling" (.xlsx uploaded once)
- **Teacher-to-room mapping is encoded by cell background color**, not by column position — the
  visible header row and the color-legend row are offset by one column in the source sheet. Extracted
  programmatically via `openpyxl`, reading `cell.fill.fgColor.rgb` and matching against the legend
  in row 1 (columns D–H: Theory Lesson / Elena / Leon Tomic / Michela Sara / Marija).
- 70 sessions total: 69 lessons (July 6–10, 2026) + 1 Grand Finale concert (July 11, 2026, 16:00–17:00)
- **Lesson duration: 50 min for every lesson** (Theory included), even though the spreadsheet shows
  hourly blocks (10 min is changeover buffer). Concert is the one exception at 60 min — implemented
  as a single `DURATION_BY_CATEGORY` constant in the JS, with a per-session `duration` override used
  only for the concert.
- **Year confirmed: 2026.**

## People
- **8 students:** Kotryna, Leonas, Diana, Mark, Elizabet, Sophie, Amy, Amelie
  - Kotryna and Leonas are siblings → the Student filter chip row is **multi-select** so one family
    can see both kids at once. Teacher and Category filters are single-select.
- **4 piano teachers:** Elena, Leon Tomic, Michela Sara, Marija
  - Theory (daily group session, 09:00, Raum 25) is taught by **Marija** — confirmed verbally by
    Marijus, not visible in the spreadsheet color-coding.
  - Teacher badge colors on the page reuse the **exact hex values from the studio's own spreadsheet
    legend** (deliberate — matches what teachers/parents may already associate with each name):
    - Elena `#D8AAF4` · Leon Tomic `#A9D08E` · Michela Sara `#FFD966` · Marija `#9FC5E8`
    - Theory (category accent, not a teacher color): `#EA9999`
- **Ieva** — studio administrator, the main parent-facing logistics contact in Vienna. Not a teacher.
- Elena and Marija are sisters, from a family of musicians (mother Dalia Šešelgienė is a violinist).
  Elena founded the studio in 2018; trained at the Royal Conservatory of Brussels and Vytautas Magnus
  University under Prof. Alexander Paley. Marija trained in violin (Graz, under Prof. Ida Bieler) and
  viola (North Carolina School of Arts; VDU under Raimondas Butvilas) — teaches piano + theory here.
- **Leon Tomic and Michela Sara are local Vienna hires** — Marijus had not met them before this trip.
  Both turned out to be teachers at the **Vienna Piano School** itself (en.viennapiano.at), which is
  located at Mühlgasse 28-30 and Mariahilfer Straße 51 — the same two venues this summer school uses.
  Worth confirming with Marijus whether Vienna Piano School is the actual host/venue partner, since
  that's almost certainly not a coincidence.

## Venues — 3 distinct addresses, looked up by ROOM not by teacher
- **Main:** Mühlgasse 30, 1040 Wien — Theory room (Raum 25) + Raum 23, 22, 01, 06 (most sessions)
- **Alternate:** Mariahilfer Straße 51, 1060 Wien — Raum 13. Used for **some** July 8th sessions only
  — July 8th is genuinely mixed between both buildings, not entirely at the alternate one (this was a
  real bug caught and fixed — don't reintroduce the "whole day is elsewhere" assumption).
- **Concert:** Feurich Performing Stage, Mariahilfer Straße 51, 1060 Wien — same building as Raum 13,
  different room (MusikQuartier venue network; resolved from a Google Maps short link).

## Contacts — footer + every .ics event description
- Main trip email: `vienna26@elenypiano.studio` (a `vienna2026@` alias also exists, but the short one
  is canonical — use it everywhere)
- Ieva (main contact in Vienna): +370 605 62 425
- Elena: +370 686 02 274
- **Deliberately not listed:** the other 3 teachers' personal contacts — keeps parents routing
  questions through Ieva rather than whichever teacher's name they recognize.
- Every `.ics` event (single download and bundle) includes in its DESCRIPTION:
  `Questions on the day? Ieva +370 605 62 425 · vienna26@elenypiano.studio`
- ICS LOCATION field format: `address (room label)` — address first so calendar apps that truncate
  at the first comma still show the useful part, not just the room code.

## Architecture / behavior notes
- All editable data lives in JS consts at the top of the `<script>` block: `SESSIONS`, `ROOMS`,
  `TEACHER_COLORS`, `DURATION_BY_CATEGORY`.
- Filtering = 3 facets: Student (multi-select), Teacher (single-select), Category (single-select:
  Theory / Practice / Concert), ANDed together, mirrored in URL params for shareable pre-filtered
  links: `?student=Kotryna,Leonas&teacher=Elena&category=Practice`
- **"Piano" was renamed to "Practice"** throughout — 65 occurrences, all replaced.
- Category chip values sourced from ALL sessions (so Concert appears as a chip despite being a group
  entry). Student and teacher chips sourced from individual (non-group) sessions only.
- **Filter logic — critical, do not regress:**
  - `studentMatch`: group sessions bypass this (always visible); individual sessions filtered normally
  - `teacherMatch`: applies to ALL sessions including Theory — Theory only shows when teacher=All or
    teacher=Marija. Concert has teacher="All teachers" which is special-cased to match any selection.
  - `categoryMatch`: applies to ALL sessions including Theory and Concert — no bypass for group entries.
  - All three conditions are ANDed. No `if(s.group) return true` blanket bypass.
- Directions: per-card "Map" button resolves `room → ROOMS[room].address` → Google Maps search URL.
- Theory/Concert card text: uses `var(--paper)` (ivory) not `var(--ink)` (dark) — these cards sit on
  a semi-transparent dark background, so inheriting the base `.session` ink color kills contrast.
- Design language: Viennese Secession–inspired (deep green / gold / ivory / garnet), Fraunces
  (display) + Inter (body) + IBM Plex Mono (data/times), a five-line staff motif as the recurring
  divider. Hardcoded dark theme — does not respond to `prefers-color-scheme`.

## Outstanding / pending
- [x] Photos received: Elena (2_10_.jpeg), Leon Tomic (IMG_2177.jpeg), Michela Sara (IMG_2178.jpeg)
- [ ] Marija photo — still pending
- [ ] Ieva photo — still pending
- [ ] "Meet your teachers" section below the schedule, 5 people: Ieva + 4 teachers.
      No fabricated photos of real people — only real uploaded photos, or fallback to colored
      initial-avatars using each person's existing badge color.
- [ ] FAQ section — Elena to provide real questions. If added: collapsed `<details>/<summary>`,
      no JS needed, below schedule above footer.
- [ ] Draft bios written for all 5:
  - **Elena Šešelgytė** — Studio founder and concert pianist, trained at the Royal Conservatory of
    Brussels and Vytautas Magnus University under Prof. Alexander Paley. Performs internationally
    and created the studio's signature mindfulness-based "Eleny Piano" method.
  - **Marija Šešelgytė** — Violin and viola performer, trained at the Graz University of Arts
    (Austria) and the North Carolina School of Arts (USA). Winner of national and international
    competitions, with concerts and festival appearances across Europe and the USA.
  - **Ieva Šešelgytė** — Administrative coordinator of the studio; handles communication,
    organization, and practical support for students and families. During the Vienna Summer School
    she oversees daily logistics, the main contact for any on-the-day questions.
  - **Leon Tomic** — Vienna-based pianist and piano teacher, trained in jazz piano (IGP, Vienna Music
    Institute) and songwriting (MA, Institute of Contemporary Music Performance, London). Started in
    classical piano, performing Schumann and Rachmaninoff concertos as a teen, before moving into
    jazz and releasing three solo albums.
  - **Michela Sara De Nuccio** — Italian pianist, trained at the Giuseppe Verdi Conservatory in Turin
    and the Koninklijk Conservatory in Brussels (cum laude, under Aleksandar Madžar). Teaches piano
    in Vienna since 2018, regularly performing at venues including the Musikverein Wien.
  - Note: Leon Tomic and Michela Sara bios were sourced via web search (Vienna Piano School's own
    teacher pages), not provided directly by Marijus/Elena — worth a quick factual check before
    publishing, in case anything's changed since those pages were last updated.

## File location
The live file is `index.html`. If resuming work elsewhere, get the latest copy from Marijus rather
than assuming an older version is current. This CLAUDE.md travels with it in the repo root.
