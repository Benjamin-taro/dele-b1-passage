---
name: dele-b1-passage
description: >
  A skill that generates DELE B1-level Spanish short reading passages.
  Designed for a daily morning study routine. Outputs a passage + translation + vocabulary list in one go (no comprehension questions).
  Trigger on keywords like "DELE", "spanish passage", "morning spanish",
  "dele practice", "spanish reading", etc.
---

# DELE B1 Daily Passage Generator

> **CI environment (GitHub Actions)**: This skill runs non-interactively via `claude -p`, with the current working directory = the repository root. All paths below are relative to the repository root. Only Read / Write / Edit / Glob / Grep are available — Bash is NOT available. Do NOT run git; the workflow commits and pushes.

## Usage

```
/dele-b1-passage [YYYY-MM-DD]
```

- **With date argument** (e.g. `/dele-b1-passage 2026-04-05`): generate a passage for that specific date.
- **In CI**: use the date given in the prompt (the workflow passes today's Europe/London date).

## Purpose

Run once every morning to get a short DELE B1-level reading passage and a complete study set.

## DELE B1 Level Criteria

Strictly follow these standards when generating a passage:

- **Vocabulary**: B1 level (everyday life, work, travel, hobbies — never use C1 vocabulary)
- **Grammar**: All indicative tenses + present subjunctive + conditional + imperative. Keep past subjunctive to a bare minimum
- **Length**: 150–250 words
- **Structure**: 3–5 paragraphs
- **Sentence complexity**: Mix simple and complex sentences. Use relative pronouns (que, donde, quien) naturally

## Regional Variety: Latin American Spanish (Mexico-first)

**Global rule (mandatory):** Use Latin American Spanish (Mexican dialect) — vocabulary, usage and spelling conventions of Mexico.

Prioritize **Mexican and Latin American Spanish** vocabulary, expressions, and usage throughout the passage. Specific guidance:

- Prefer Latin American vocabulary over Castilian Spanish where there is a difference (e.g., *carro* over *coche*, *celular* over *móvil*, *computadora* over *ordenador*, *camión* for bus in Mexico, *departamento* over *piso*, *platicar* over *charlar*, *ahorita*, *padre* (cool), etc.)
- Use Latin American grammar preferences: *ustedes* instead of *vosotros*, *se los digo* instead of *os lo digo*
- Settings, place names, and cultural references should reflect Mexico or Latin America when relevant (e.g., markets, taquerías, CDMX, Guadalajara, Monterrey)
- If a word has a notably different meaning in Mexico vs. Spain, use the Mexican meaning and note the difference in the vocabulary table if it appears there

## Theme Pool

Select randomly from the following categories (use a different theme each time):

1. **Daily life**: Shopping, cooking, moving house, neighbors
2. **Travel & tourism**: Hotel bookings, asking for directions, travel mishaps
3. **Work & school**: Job hunting, workplace relationships, school events
4. **Health & sports**: Doctor visits, exercise habits, diet
5. **Culture & society**: Festivals, traditions, environmental issues, technology
6. **Media & entertainment**: Movies, music, social media, news
7. **Relationships**: Friendships, family, life with a roommate

## History Management (Avoiding Repetition)

Use the `history.json` file at the repository root to track past outputs.

### Reading on Execution

Before generating a passage, check whether `history.json` exists.

- **If the file exists**: Read it and review past theme categories, subtopics, titles, and vocabulary
- **If the file does not exist**: Treat it as an empty state and proceed as a first run

### Theme Selection Logic

1. Check the categories of the most recent 7 entries in `history.json`
2. If all 7 categories appear in the last 7 entries, reset the rotation
3. Prioritize categories that have not appeared yet
4. Even within the same category, avoid repeating subtopics (e.g., under "Travel & tourism", treat "Hotel bookings" and "Asking for directions" as separate)

### Vocabulary Overlap Check

A separate file `vocab-used.json` at the repository root (`vocab-used.json`) maintains a **flat sorted array of every vocabulary word ever used**, across all history entries. This file is the single source of truth for overlap checking — do NOT scan `history.json` entries for this purpose.

**On execution:**
1. Read `vocab-used.json` (treat as an empty array `[]` if the file does not exist).
2. When selecting the NEW vocabulary items for Part 3, **only pick words that do not appear in `vocab-used.json`**.
3. If a word is so fundamental to the passage topic that it is truly unavoidable, allow at most **1 repeat** — and only if no reasonable synonym exists.
4. If the draft vocabulary still has more than 1 overlap, revise the passage wording to surface fresher words.

**After generating:**
5. Append the new vocabulary words to the array in `vocab-used.json`, re-sort alphabetically (case-insensitive), deduplicate, and save.
6. Save `vocab-used.json` (the workflow commits it — do NOT run git).

### Review Words (meet old words again)

New words alone are never met again, so every passage must also **reuse words from earlier passages**. This overrides the "only new words" rule above for the review words (the "at most 1 repeat" limit applies to the NEW words only).

1. Read `read.json` at the repository root (treat as `{"read": {}, "level": {}}` if missing). `read` maps the dates of passages the learner has finished reading.
2. Review candidates = the `vocab` of `history.json` entries whose `date` is a key of `read`. Words from passages not yet read are not candidates.
3. Prefer words first seen **3–30 days before the target date**; if there are too few, use older ones. Do not pick words listed in the `review` array of the 3 most recent history entries.
4. Choose **3–4 review words** that fit today's theme and use them naturally in the passage body. Do not force a word that does not fit — pick another candidate instead.
5. The vocabulary table (Part 3) contains **6–8 new words** (not in `vocab-used.json`) **plus the 3–4 review words** (9–12 rows in total). Put new words first, then review words. Prefix each review word with `🔁 ` in the first column, and add this line under the table: `🔁 = 以前の passage に出た単語（復習）`. In the HTML, give review rows `class="review"` with a light background (`#f3f8f1`).
6. In the new `history.json` entry, keep `vocab` for the new words only and add `"review": ["...", "..."]` for the review words. Do **not** add review words to `vocab-used.json` again.
7. If there are no review candidates yet (nothing read), skip review words and use new words only.

### Difficulty Adjustment (learner feedback)

`read.json` may contain `level`: a map of date → `"easy"` | `"ok"` | `"hard"`, the learner's rating of that passage. Look at the **5 most recently rated** passages:

- **3 or more `"hard"`** → make it easier: length at the lower end of the range, shorter sentences, new words at the minimum count, review words at the maximum count.
- **3 or more `"easy"`** → make it harder, still inside the B1 limits defined above: length at the upper end of the range, more varied sentence structures that the level allows, new words at the maximum count.
- Otherwise → keep the usual difficulty.

Never go outside the B1 level definition. Record the decision in the new `history.json` entry as `"difficulty": "easier" | "same" | "harder"`.

### Writing After Execution

After the passage is generated, append the current run's record to `history.json`.

### history.json Format

```json
{
  "entries": [
    {
      "date": "2026-03-10",
      "category": "Travel & tourism",
      "subtopic": "Travel mishaps",
      "title": "Un problema en el aeropuerto",
      "vocab": ["equipaje", "reclamar", "vuelo", "retraso", "mostrador", "embarque", "pasaporte", "aduana"],
      "review": ["...", "..."],
      "difficulty": "same"
    }
  ]
}
```

File path: `history.json` at the repository root

## Output Format

Write everything to a folder at `passages/YYYY-MM-DD/` at the repository root, outputting two files:
- `passages/YYYY-MM-DD/YYYY-MM-DD.md` — Markdown format
- `passages/YYYY-MM-DD/YYYY-MM-DD.html` — HTML format (styled, self-contained)

Do NOT output the passage content to the CLI — only write to the files.

The file should contain the following sections in order:

### Part 1: Passage

```
📖 Pasaje del día — [Theme category]
Título: [Title]

[Passage body (Spanish only)]
```

### Part 2: English Translation

A natural English translation of the full passage body.

In the HTML output, Part 1 and Part 2 must be rendered **side by side** using a two-column flexbox layout (`.passage-columns`). The Spanish passage goes in the left column and the English translation in the right column. On mobile (max-width: 700px), they stack vertically. Do NOT use a separate `<h2>` heading for the English section — instead use a small label (`<div class="passage-col-label">`) above each column.

Do **NOT** include a text-to-speech (TTS) button or any related JavaScript in the HTML output.

```
🇬🇧 English Translation

[English translation of the passage, paragraph by paragraph]
```

### Part 3: Vocabulary List (6–8 new + 3–4 review words)

Pick out key B1 vocabulary from the passage.

```
📝 Vocabulario clave

| Palabra | Significado (ES) | English | Japanese | Example from text |
|---------|------------------|---------|----------|-------------------|
| ...     | ...              | ...     | ...      | ...               |
```

> **問題と解答は作らない。** 出力は Part 1〜3（本文・訳・語彙）だけにする。Markdown にも HTML にも、読解問題・解答・解説のセクションを入れない。

## Execution Steps

1. **Determine the target date**: Use the date given in the prompt (YYYY-MM-DD format). All subsequent file paths and HTML titles must use this date.
2. **Check for existing output**: If `passages/YYYY-MM-DD/YYYY-MM-DD.md` already exists, stop immediately and output:
   `⚠️ passages/YYYY-MM-DD/ already exists. To regenerate, delete the folder first.`
   Do NOT proceed further.
3. Read `history.json` (treat as empty if it does not exist)
4. Read `read.json` (treat as empty if it does not exist). Decide the difficulty (Difficulty Adjustment) and choose the review words (Review Words) before writing.
5. Refer to the history and select a non-overlapping theme category and subtopic
6. Generate a B1-level passage on that theme
7. Write a natural English translation of the passage
8. Build the vocabulary table: 6–8 new words (cross-reference `vocab-used.json`; replace overlaps until at most 1 remains) plus the 3–4 review words chosen above, each with all required columns.
9. Append the current date / category / subtopic / title / vocab to `history.json` and save
10. Create the `passages/YYYY-MM-DD/` directory at the repository root (writing the files below with the Write tool creates it)
11. Write the full output (Parts 1–3) to `passages/YYYY-MM-DD/YYYY-MM-DD.md`
12. Write the same content as a styled, self-contained HTML file to `passages/YYYY-MM-DD/YYYY-MM-DD.html`
13. Update `index.html` — prepend a new `<li>` entry at the top of the `<ul class="list">` block. The entry **must** include `data-date` and `data-category` attributes so that the month/theme grouping JavaScript picks it up automatically. Use this exact format (replace placeholders):
    ```html
    <li data-date="YYYY-MM-DD" data-category="CATEGORY">
      <a href="passages/YYYY-MM-DD/YYYY-MM-DD.html">
        <span class="title-text">TITLE</span>
        <span class="tag">CATEGORY</span>
        <span class="date">YYYY-MM-DD</span>
      </a>
    </li>
    ```
    `CATEGORY` must be one of the exact strings used elsewhere: `Daily life`, `Travel &amp; tourism`, `Work &amp; school`, `Health &amp; sports`, `Culture &amp; society`, `Media &amp; entertainment`, `Relationships`. Use `&amp;` for `&` in both `data-category` and `<span class="tag">`. New months and themes are grouped automatically by the existing JavaScript — no other changes to `index.html` are needed.
14. Copy the generated HTML to overwrite `today.html` at the repo root: write exactly the same content as `passages/YYYY-MM-DD/YYYY-MM-DD.html` to `today.html` using the Write tool (Bash/`cp` is not available).
15. Do NOT run git. The workflow commits and pushes (and triggers the GitHub Pages build).
16. Output only a short confirmation to the CLI:
    `✅ Saved to passages/YYYY-MM-DD/ — [Title]`
    `🌐 https://benjamin-taro.github.io/dele-b1-passage/today.html`

## Quality Checks

Verify the following before output:

- The passage is within 150–250 words
- No vocabulary or grammar above B1 has crept in
- The theme differs from recent entries (confirmed via `history.json`)
- The `history.json` append is complete
