---
name: dele-b1-passage
description: >
  A skill that generates DELE B1-level Spanish short reading passages.
  Designed for a daily morning study routine. Outputs a passage + vocabulary list + comprehension questions in one go.
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
2. When selecting the 8–12 key vocabulary items for Part 3, **only pick words that do not appear in `vocab-used.json`**.
3. If a word is so fundamental to the passage topic that it is truly unavoidable, allow at most **1 repeat** — and only if no reasonable synonym exists.
4. If the draft vocabulary still has more than 1 overlap, revise the passage wording to surface fresher words.

**After generating:**
5. Append the new vocabulary words to the array in `vocab-used.json`, re-sort alphabetically (case-insensitive), deduplicate, and save.
6. Save `vocab-used.json` (the workflow commits it — do NOT run git).

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
      "vocab": ["equipaje", "reclamar", "vuelo", "retraso", "mostrador", "embarque", "pasaporte", "aduana"]
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

### Part 3: Vocabulary List (8–12 words)

Pick out key B1 vocabulary from the passage.

```
📝 Vocabulario clave

| Palabra | Significado (ES) | English | Japanese | Example from text |
|---------|------------------|---------|----------|-------------------|
| ...     | ...              | ...     | ...      | ...               |
```

### Part 4: Comprehension Questions (3 questions)

Multiple-choice questions modeled after the DELE B1 reading section.

Each option must include its English and Japanese translation on the same line, in parentheses.

```
❓ Comprensión lectora

1. [Question (in Spanish)]
   a) ... (English / 日本語)
   b) ... (English / 日本語)
   c) ... (English / 日本語)

2. ...
3. ...
```

### Part 5: Answers and Explanations

```
✅ Respuestas

1. [Correct answer] — [Brief explanation of why (in Japanese)]
2. ...
3. ...
```

## Execution Steps

1. **Determine the target date**: Use the date given in the prompt (YYYY-MM-DD format). All subsequent file paths and HTML titles must use this date.
2. **Check for existing output**: If `passages/YYYY-MM-DD/YYYY-MM-DD.md` already exists, stop immediately and output:
   `⚠️ passages/YYYY-MM-DD/ already exists. To regenerate, delete the folder first.`
   Do NOT proceed further.
3. Read `history.json` (treat as empty if it does not exist)
3. Refer to the history and select a non-overlapping theme category and subtopic
4. Generate a B1-level passage on that theme
5. Write a natural English translation of the passage
6. Extract key vocabulary from the passage. Cross-reference against the full used-words set from all history entries. Replace any overlapping words until at most 1 repeat remains (ideally zero). Adjust the passage wording if needed to surface fresher vocabulary
7. Create 3 comprehension questions
8. Create answers and explanations
9. Append the current date / category / subtopic / title / vocab to `history.json` and save
10. Create the `passages/YYYY-MM-DD/` directory at the repository root (writing the files below with the Write tool creates it)
11. Write the full output (Parts 1–5) to `passages/YYYY-MM-DD/YYYY-MM-DD.md`
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
- Comprehension question options are clearly distinguishable (not overly tricky)
- The theme differs from recent entries (confirmed via `history.json`)
- The `history.json` append is complete
