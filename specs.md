# Selit: build specs (FOI Putrajaya 2026, Agentic AI Challenge)

Handoff brief for the build session. Written 2026-11-09 18:40 (MYT). Card: `selit.md`. Log: `log.md`.
**Deadline: 2026-11-09 23:59 MYT (about 5 hours from writing). Late entries are not accepted.**

## 1. Competition facts (read from the organiser pages 2026-11-09; re-read live rules before submitting, rule 7.4 lets them amend)
- **Event:** Festival Idea Putrajaya (FOI) 2026, run by Kementerian Pendidikan Tinggi (KPT) via Jabatan Pendidikan Tinggi (JPT). PICC Putrajaya, 17-19 Nov 2026. Site: https://sites.google.com/mohe.gov.my/foi2026ai/home (Google Sites app inside an iframe; scrape with Playwright, plain scrapers return nothing).
- **Our track:** Agentic AI Challenge, "The Solution Arena", pitch Thu 19 Nov, Dewan Putra A. The other track (Innovation Challenge, pitch 18 Nov) is not entered. Rule 1.6: one contest only, one category, one team.
- **Category:** IPT (public or private university student). All team members must be in the same category (1.4). John is solo, UM.
- **Theme (both tracks):** "Edutainment & Innovation". Undefined by the organiser. Scored only on the Innovation track (its criterion 1 includes theme fit); the Agentic criteria do not mention it.
- **Entry:** free. Online form, 5 steps (Kategori, Peserta, Projek, Bahan, Semak). Draft autosaves in the same browser only. One entry per team (2.3). Deadline 9 Nov 2026 23:59 (2.4). Form field details beyond the step names were not seen.
- **Material (rule 4):** a demo video of at most 5 minutes that shows the agent working (4.1). Any HTML code or web app shown or submitted must show a clear "DEMO" or "SAMPLE" watermark (4.2). Video on YouTube, Google Drive or OneDrive with "Anyone with the link"; a link the panel cannot open is not judged (4.3). Video in Malay or English; English is encouraged to carry Malay subtitles (4.4).
- **Definition (rule 3.1):** must be an AI agent that makes decisions and acts autonomously. A plain Q&A chatbot is ineligible.
- **Originality and ethics (rule 5):** original work that has never won another competition (5.1); declare every AI model, framework and tool in the form (5.2); undeclared third-party components or plagiarism can void the entry (5.3); no real personal data in the video or demo (5.4); API and cloud costs are the entrant's (5.5).
- **Judging (verbatim intent):** (1) innovation and creativity of the solution; (2) agent capability and quality: works autonomously, AI at the core, used responsibly; (3) impact and usability: solves a real problem, can be scaled; (4) presentation and Q&A quality.
- **Timeline:** finalists announced 12 Nov. Live pitch 19 Nov, 10 minutes plus 5 minutes Q&A. A backup demo recording is encouraged; the panel may also refer to the submitted video.
- **Prize:** "RM50,000" per challenge. The organiser's Instagram (2026-11-03) says it "includes an R&D matching grant routed through the IPT (merangkumi Geran Padanan R&D melalui IPT)". Split, cash portion and release terms are NOT published. Never call it cash or salary. Prize and conditions are subject to the organiser's decision.
- **Rights:** copyright stays with the entrant; the organiser may use the work for promotion (7.1). Jury decision final, no appeal (6.4).
- **Contact:** Urus Setia, Bahagian Standard Swasta, JPT: sppbss+FOI@mohe.gov.my. Questions worth sending: who receives the R&D grant (team, university, company), is it released in stages, is theme fit mandatory on this track.
- **Known competitors:** Tulen (UTHM; scam-letter agent, 8 tools, 88-message eval) and MyTadbir (multi-agent public-service desk officer). Both are real engineering. Public winner data for FOI challenges is near nil.

## 2. What we build (settled with John, 2026-11-09)
**One engine, two users, one game style, Agentic track.** John decided to do it all; the demo is mock-first (no GitHub link needed, only a video), the real build is the later roadmap.
- **Engine:** weekly quiz results in, weak topics per class and per student out, verified questions generated, spaced schedule planned.
- **User 1, teacher (Cikgu Loop):** gets a re-teach list and a per-student report; approves before anything goes to students.
- **User 2, student (Selit):** the agent pushes short questions in a chat at spaced times (the bus stop), adapts to answers, stops when ignored.
- **Game style (Pak Pandir; game name "Betulkan Pak Pandir"):** a spot-the-mistake round in the browser. Pak Pandir (the Malay folk fool) shows a worked solution line by line; one line has a mistake injected by code; the student taps the wrong line. Not an app-store game, not 3D; about 10 problems, 2-3 minutes.
- **Demo hero:** the student side (it shows autonomy most vividly). The teacher side is the first minute of the demo and the source of the data.
- **Dropped:** students authoring traps (too heavy); heritage/VR layers; photo OCR.

## 3. Agent design (this is what judges score as "agent capability")
**Tools (6 to 8, the model chooses among them):**
1. `analyse_results(csv)`: per-question correctness mapped to DSKP standard codes; ranks weak standards per class and per student.
2. `lookup_standard(code)`: reads a hand-typed JSON of about 40 KSSM Form 4 Maths standards (code format `chapter.contentStandard.learningStandard`, e.g. `2.1.1`; source: DSKP PDFs, born-digital; Maths F4/F5 mirror parsed in `verify-merged.md`).
3. `generate_questions(standard, n)`: LLM drafts questions with a worked solution as a list of algebraic steps.
4. `verify_question(q)`: an independent SymPy solver re-solves the question from the stem only and compares; unverified items are dropped and counted.
5. `inject_mistake(solution)`: deterministic mutation of one step (sign flip, dropped coefficient, wrong index rule, arithmetic slip), then SymPy confirms the mutated step is NOT equivalent to the previous one, so the exact wrong line is known.
6. `plan_schedule(student, now)`: spaced-repetition policy (below) plus coverage check that every weak standard has a slot.
7. `compose_nudge(student, item)`: wording only; short BM/EN.
8. `report_teacher()`: weekly summary; blocks on `await_approval()`.
All steps write to an **audit log** shown on screen. The LLM never executes an action directly: it proposes, code decides and logs (the pattern Gemini and Claude web both flagged as the winning one).

**Visible autonomous behaviours (put each on screen at least once):**
- verifier drops items, the agent regenerates with a simpler template (re-plan on failure);
- coverage check finds an uncovered weak standard, revises, re-runs;
- the agent chooses question style per student (normal vs Pak Pandir) after two wrong answers;
- backoff: after 2 unanswered nudges it stops and flags the teacher; quiet hours respected;
- next week's results arrive and the plan updates without a prompt.

**Spaced-repetition policy (design choice, not a tested protocol):** Leitner-style boxes per student per standard. New weak item due in 1 day; correct answer moves it to intervals of 1, 3, 7, 14 days; wrong answer returns it to box 1 and re-queues the same day later. Max 3 nudges per day; quiet hours 21:00-06:30. The demo uses a simulated clock.

## 4. Data (all synthetic, DEMO/SAMPLE watermark)
- **Class CSV:** 30 invented students ("Pelajar A".."Pelajar AD" or invented first names), 12 questions, each tagged with a standard code, values 1/0. Plant three patterns: one weak topic for most of the class, one weak topic for one student, one student who never answers nudges. Two to three topics only (suggest quadratic functions, indices, simultaneous equations).
- **No real personal data anywhere in the video (rule 5.4).** No real school, no real names, no real phone numbers.

## 5. Architecture and stack
- **Backend:** Python (FastAPI) with a plain loop; SymPy; storage as JSON files; an LLM adapter so the model is swappable.
- **LLM:** DeepSeek V4 via its OpenAI-compatible API (John's usual, cheap). From the official pricing page as read 2026-11-09: `deepseek-flash` supports vision, JSON output and tool calls, about USD 0.15 in / 0.60 out per 1M tokens off-peak, doubled at peak (06:00-10:00 UTC, i.e. 14:00-18:00 MYT). Re-check names and prices before use. Data is stored in the PRC and PDPA 2024 has a cross-border transfer guideline: send only anonymised item-level data (standard codes and correct/wrong), never names; declare the model; keep the adapter so another model can replace it.
- **Frontend:** one web page with three panels: teacher view (results, weak topics, plan, approve), student chat (WhatsApp-style simulated), decisions panel (why this question, why now; audit log). Watermark "DEMO / SAMPLE" on every view.
- **Channel:** simulated chat for the demo. Real channel later: Telegram bots can message first once a user has started them (easiest); WhatsApp needs the paid Business API with approved templates, or an unofficial linked-device route (OpenClaw uses a QR link) with ban and terms risks; minors need consent.

## 6. Real vs mocked
| Real | Mocked |
|---|---|
| results analysis, schedule policy, coverage check | the chat channel (simulated) |
| SymPy verifier and mistake injector | students and class data (synthetic) |
| LLM question drafting (DeepSeek) | the clock (simulated days) |
| decisions panel and audit log | next-week scores (scripted) |
| | teacher approval (a button) |

**Build order and cut line (stop adding at the first red item):**
- P0: CSV analysis + verifier + mutation + a schedule timeline.
- P1: chat UI + Pak Pandir round + decisions panel.
- P2: teacher report + approval + the loop closing with next week's results.
- P3: cached "bunker" run (pre-recorded outputs) so the demo never depends on the network.
- P4: polish, Malay subtitles.
If the engine is not working by 21:30, ship the story with a pre-validated question set and keep the verifier shown on a cached run; say so plainly in the video, never fake it.

## 7. Demo script (5:00 maximum; record to this)
1. 0:00-0:30 Problem and two fake people (a teacher with a spreadsheet, a student at a bus stop). One sentence. No invented statistics.
2. 0:30-1:30 Teacher side: results in; weak topics per class and per student; question generation; the verifier shows "38 generated, 31 verified, 7 dropped" and the agent regenerates.
3. 1:30-2:30 Teacher taps Approve; the agent lays out the week as a timeline.
4. 2:30-3:45 Student side: nudge arrives in the chat; a normal question; a Pak Pandir round (find the wrong line); a wrong answer re-queues sooner; the decisions panel explains why; two unanswered nudges and the agent stops and flags the teacher.
5. 3:45-4:20 Next quiz comes back; the plan updates; the teacher report refreshes (loop closed).
6. 4:20-5:00 Responsible AI (verifier decides, code owns actions, teacher approves, synthetic data, declared models), then the roadmap (Telegram/WhatsApp pilot, one UM partner class, what the grant would pay for).
Language: Malay or English; if English, burn in Malay subtitles (rule 4.4). DEMO/SAMPLE watermark must be visible in the recording (4.2).

## 8. Mini-eval (the "one measured result")
Generate 40 questions across the two or three topics; report verified / dropped counts, any wrong answer key found by manual check, and the Pak Pandir mutation check (all injected errors confirmed non-equivalent). Label everything as synthetic. Do not claim learning gains: spacing evidence is general, no WhatsApp-specific study was found.

## 9. Entry-form declaration (draft; verify wording against the live form)
- **Models and services:** DeepSeek V4 (question drafting and message wording, anonymised item-level data only); SymPy (independent verification and mistake injection, not an AI model).
- **Frameworks and libraries:** FastAPI, SymPy, plain HTML/CSS/JS (confirm the final list from the repo).
- **AI coding assistant:** Claude Code (Anthropic); the entrant reviewed and owns the result.
- **Data:** all students and results are synthetic and marked SAMPLE. DSKP standard codes and descriptors are paraphrased from public KPM documents.
- State plainly what was generated with an assistant vs hand written.

## 10. Q&A bank (finalist pitch)
- Is this just a chatbot? No: it initiates, schedules, chooses tools and question styles, backs off, escalates and re-plans; the chat is only the delivery channel.
- Where does the AI decide, and where is it not trusted? The LLM drafts; an independent solver verifies or drops; code owns every action; a teacher approves.
- What if it is wrong? Unverified items are dropped; the wrong-line in Pak Pandir is code-injected so the key is known; teacher can edit.
- Data and PDPA? Synthetic in the demo; only anonymised item-level data to the model; adapter allows another model; real minors need consent and a data assessment.
- Why DeepSeek? Cost; declared; swappable; no names sent.
- How is it different from Pandai or Gemini? They generate content; this plans per student from a class's own results, verifies each question, and closes the loop weekly.
- Evidence it works? Spacing is well established in general; no WhatsApp-specific test found; we state that and propose a pilot.
- Who pays after the grant? API cost is small; one pilot class; the grant would pay for the pilot and a device, via the university.
- Scale? Standards are a JSON; new subjects add topics where a verifier exists; geometry and proofs are out of scope.

## 11. Tonight's timeline (from 18:40; deadline 23:59)
- 18:40-19:10 repo skeleton, DSKP JSON (about 40 codes), synthetic CSV.
- 19:10-20:30 engine: analysis, generation, verifier, mutation, scheduler.
- 20:30-21:30 UI: teacher view, chat, Pak Pandir round, decisions panel, watermark.
- 21:30 CUT LINE: assess and decide mock vs real for the remainder.
- 21:30-22:00 mini-eval and cached run.
- 22:00-22:45 record the video, subtitle if English.
- 22:45-23:15 upload, test the link in a private window while logged out, fill the form (declaration).
- 23:15-23:59 buffer. Submit before 23:30 if possible.

## 12. Open items
- [ ] John: confirm solo, IPT, UM student; confirm no project here has won another competition (new work, so expected fine).
- [ ] Pick the two or three topics for the verifier.
- [ ] Choose Malay or English narration.
- [ ] Email the secretariat the three questions (grant recipient, release terms, theme fit).
- [ ] Re-read the live rules and form before submitting.
