---
name: dele-b1-passage
description: >
  A skill that generates DELE B1-level Spanish short reading passages.
  Designed for a daily morning study routine. Outputs a passage + vocabulary list + comprehension questions in one go.
  Trigger on keywords like "DELE", "spanish passage", "morning spanish",
  "dele practice", "spanish reading", etc.
---

# DELE B1 Daily Passage Generator

## Usage

```
/dele-b1-passage [YYYY-MM-DD]
```

- **With date argument** (e.g. `/dele-b1-passage 2026-04-05`): generate a passage for that specific date.
- **Without argument**: use today's date (from the `currentDate` context or system clock).

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

Use a `history.json` file in the same directory as the skill to track past outputs.

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

1. Review the vocabulary lists from the last 14 days in `history.json`
2. If 5 or more of the key vocabulary items (the 8–12 words in Part 2) overlap with the last 14 days, adjust the passage to prioritize new vocabulary

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

File path: `history.json` inside the skill directory (e.g., `dele-b1-passage/history.json`)

## Output Format

Write everything to a folder at `passages/YYYY-MM-DD/` (using today's date), outputting two files:
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

**Text-to-Speech (TTS) button — required in every HTML output:**

The Spanish passage container (`<div class="passage-box">`) must have `id="spanish-text"`. Immediately before the `<div class="passage-columns">` block, insert a TTS button. At the end of the file, just before `</body>`, include the following script:

```html
  <div style="margin-bottom: 16px;">
    <button id="tts-btn" onclick="toggleSpeak()" style="background:#d4a96a;color:#fff;border:none;padding:10px 20px;border-radius:6px;font-size:1rem;cursor:pointer;font-family:sans-serif;">🔊 スペイン語を読み上げ</button>
  </div>
  <div class="passage-columns">
    <div>
      <div class="passage-col-label">🇪🇸 Español</div>
      <div class="passage-box" id="spanish-text">
        ...
```

```html
<script>
function toggleSpeak() {
  if (speechSynthesis.speaking) {
    speechSynthesis.cancel();
    document.getElementById('tts-btn').textContent = '🔊 スペイン語を読み上げ';
    return;
  }
  const text = document.getElementById('spanish-text').textContent;
  const utterance = new SpeechSynthesisUtterance(text);
  utterance.lang = 'es-ES';
  utterance.rate = 0.85;
  utterance.onend = () => document.getElementById('tts-btn').textContent = '🔊 スペイン語を読み上げ';
  document.getElementById('tts-btn').textContent = '⏹ 停止';
  speechSynthesis.speak(utterance);
}
</script>
</body>
```

The button toggles between `🔊 スペイン語を読み上げ` and `⏹ 停止` while speaking. Language is `es-ES`, rate is `0.85`.

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

1. **Determine the target date**: Use the date argument if provided (YYYY-MM-DD format). Otherwise, use today's date from the `currentDate` context variable or system clock. All subsequent file paths, commit messages, and HTML titles must use this date.
2. **Check for existing output**: If `passages/YYYY-MM-DD/YYYY-MM-DD.md` already exists, stop immediately and output:
   `⚠️ passages/YYYY-MM-DD/ already exists. To regenerate, delete the folder first.`
   Do NOT proceed further.
3. Read `history.json` (treat as empty if it does not exist)
2. Refer to the history and select a non-overlapping theme category and subtopic
3. Generate a B1-level passage on that theme
4. Write a natural English translation of the passage
5. Extract key vocabulary from the passage and verify that fewer than 5 words overlap with the last 14 days. Adjust if needed
6. Create 3 comprehension questions
7. Create answers and explanations
8. Append the current date / category / subtopic / title / vocab to `history.json` and save
9. Create the `passages/YYYY-MM-DD/` directory
10. Write the full output (Parts 1–5) to `passages/YYYY-MM-DD/YYYY-MM-DD.md`
11. Write the same content as a styled, self-contained HTML file to `passages/YYYY-MM-DD/YYYY-MM-DD.html`
12. Update `index.html` — prepend a new `<li>` entry at the top of the `<ul class="list">` block, using the same format as the existing entries, with the correct date, title, and category tag
13. Copy the generated HTML to overwrite `today.html` at the repo root:
    `cp passages/YYYY-MM-DD/YYYY-MM-DD.html today.html`
14. Run the following git commands (using Bash) to publish to GitHub Pages:
    ```
    git add passages/YYYY-MM-DD/ history.json index.html today.html
    git commit -m "passage: YYYY-MM-DD — [Title]"
    git push origin main
    ```
15. Output only a short confirmation to the CLI:
    `✅ Saved to passages/YYYY-MM-DD/ and pushed to GitHub Pages`
    `🌐 https://benjamin-taro.github.io/dele-b1-passage/today.html`

## Quality Checks

Verify the following before output:

- The passage is within 150–250 words
- No vocabulary or grammar above B1 has crept in
- Comprehension question options are clearly distinguishable (not overly tricky)
- The theme differs from recent entries (confirmed via `history.json`)
- The `history.json` append is complete
