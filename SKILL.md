---
name: skill-line-coach
description: "Coach and structure Markdown notes, diaries, and problem lines (CtxFST). Use when writing or reviewing diary entries and notes to improve their structure and future retrievability with the Four-Slot Tension Coach (誰在做 / 動作＋對象 / 被什麼擋 / 戰報), separating mixed passages into self-contained problem lines (%% line-start %% / %% line-end %%), or linking notes to recurring problem shape cards."
---

# Skill Line Coach (CtxFST)

`skill-line-coach` helps the user **write and structure Markdown notes and diary entries** so each problem line (`問題線`) is clear, satisfying to read, and naturally retrievable in the future.

It operates at three levels:

1. **Write-Time Four-Slot Coach (張力句四槽教練)**: When the user is writing a diary entry or note, help them leave the complete shape of each problem line in the text itself—**who acted, what they did, what concrete obstacle blocked them, and whether that obstacle receded afterward**. Lines written with all four slots are both clearer today and directly retrievable by plain embeddings later.
2. **Line Separation & Standalone Check (分線與獨立可檢索檢查)**: Identify when a passage mixes multiple distinct problem lines, separate them, ensure each line stands on its own without ambiguous pronouns, and optionally mark boundaries with `%% line-start id="..." %%` / `%% line-end %%`.
3. **Recurring Problem Shape Cards (主題線形狀卡檢索)**: Link notes across years to hand-written shape cards using embedding probes and human confirmation/rejection feedback (`scripts/shape_layer.py`).

## When to use which step

| The user wants to… | Do |
|---|---|
| Write or improve a diary entry / note (`教練` / `評比` / `改寫` / `比較`) | Run the **Four-Slot Structure Coach** ([references/four-slot-coach.md](references/four-slot-coach.md)) |
| Separate a multi-topic note into self-contained problem lines | Separate problem lines and check standalone readability |
| Find notes with the same problem shape as a known pattern | Check the card, then `build`, `link`, and `review` with `scripts/shape_layer.py` |
| Turn a recurring pattern they noticed into a card | Write a card ([references/shape-cards.md](references/shape-cards.md)) |
| Find notes similar to one note | `build`, then `similar` |

---

## Part 1: Write-Time Four-Slot Coach (張力句四槽教練)

Read [references/four-slot-coach.md](references/four-slot-coach.md) for the full coaching rules and boundaries. When the user shares a draft diary entry, note, or sentence:

### The Four Role Slots (四個角色槽)

1. **誰在做 (Actor)**: Who acts. An omitted subject counts as present only if the context uniquely identifies one actor (state which sentence implies it).
2. **動作＋對象 (Action + Object)**: What was done, to what object, along this problem line.
3. **被什麼擋 (Concrete Blocker / Opponent)**: A specific, observable obstacle pointing to an event, note, or moment. Vague labels like "bad state", "not enough time", or "anxious" count as **missing**.
4. **戰報 (Outcome / State Change)**: After the action, did that same obstacle recede? Victory, failure, partial improvement, stalemate, or ongoing observation all count.
   - Criterion: **Does it directly answer how the same obstacle changed?** Praise of a tool ("very convenient", "I like it") evaluates the means, not whether the obstacle receded, so it does not count as slot 4.

### Coaching Workflow & Modes

1. **Classify the writing first**:
   - **Problem-solving (解決問題型)**: Decisions, tools, actions, or retrospectives → apply the four slots.
   - **Emotional (情緒型)**: Acknowledge the feeling first, then gently ask: *"What did this feeling block you from doing?"*
   - **Abstract conclusion (抽象結論型)**: Ask: *"Which concrete event did this conclusion grow out of?"*
   - **Pure observation / lyrical (純觀察／純抒發型)**: Let it pass warmly without slot-by-slot grading.
2. **Separate multiple problem lines (分線)**:
   - If a passage contains 2+ distinct problem lines, list them separately instead of merging them.
   - If there are more than 3 lines, state the total count, list the first 3 in order of appearance, and explicitly note how many remain (never silently drop caregiving, physical health, or emotional lines).
3. **Choose the interaction mode**:
   - **Coach (`教練`, default)**: Summarize the structure you see in one sentence, then ask **only one** non-leading probe question (prioritize slot 3 *Blocker* before slot 4 *Outcome*). Do not score or rewrite.
   - **Evaluate (`評比`)**: Mark each of the four slots as **有 (present)** or **缺 (missing)** per problem line, quoting the user's text as evidence. **Never give a numeric total score (e.g. 3/4).**
   - **Rewrite (`改寫`)**: Preserve all facts, time markers, qualifiers, and meaning; never invent facts or motives. If 2+ slots are missing and cannot be filled without new facts, state what is missing and ask one coach probe instead of forcing a rewrite.
   - **Compare (`比較`)**: Compare before/after versions per slot and highlight what structural piece was genuinely added.

---

## Part 2: Line Separation & Future Retrievability (分線與可檢索性)

When structuring a diary or note for future retrieval:

1. **The Standalone Test (獨立可讀檢查)**: Can someone (or future search) understand this problem line without reading surrounding paragraphs? Flag dangling pronouns ("fixed that issue", "used the previous method") and prompt the user to name the specific object.
2. **Line Markers (optional)**: Long notes that mix several distinct problems can be separated into independent paragraphs/headings, or marked explicitly so `shape_layer.py` indexes each problem line on its own (`path#line-id`):

```markdown
%% line-start id="work:migration-2014" %%
…self-contained problem line…
%% line-end %%
```

---

## Part 3: Recurring Problem Shape Cards (`scripts/shape_layer.py`)

When linking notes across years to recurring problem shapes ([references/ctxfst-spec.md](references/ctxfst-spec.md), [references/shape-cards.md](references/shape-cards.md)):

### 1. Configure

Set `CTXFST_*` variables or pass `--env`:

```
CTXFST_EMBED_KEY=…                  # Voyage by default; CTXFST_EMBED_URL / _MODEL to change
CTXFST_LLM_URL=…/chat/completions   # judge only; any OpenAI-compatible endpoint
CTXFST_LLM_MODEL=…
CTXFST_LLM_KEY=…
```

### 2. Build the index

```bash
python3 scripts/shape_layer.py --env .env build NOTES_DIR --out INDEX --exclude 'private/*'
```

Pass `--cut-at REGEX` if notes carry generated footer blocks.

### 3. Write shape cards

A card ([references/shape-cards.md](references/shape-cards.md)) is written by the person and includes:
- **Problem shape (`## 問題形狀`)**: the roles, not the topic.
- **Core tension (`## 核心張力`)**.
- **Breakdown / abandonment reasons (`## 崩潰／放棄原因（疫苗欄）`)**: what was tried before, where it stalled, and what to check before repeating the same mistake.
- **Near misses (`## 最像但不同的線`)**: look-alikes that are not this shape.
- **Triggers (`## 觸發條件`)**.
- **Instance table (`## 化身清單`)** and **Rejected list (`## 退回`)**.

### 4. Link & Review

```bash
python3 scripts/shape_layer.py --env .env link INDEX --cards CARDS_DIR --top 40
python3 scripts/shape_layer.py review INDEX --cards CARDS_DIR
```

- Mark candidates in `INDEX/review.md` with `[x]` (confirm) or `[-]` (reject), then run `review`.

## Resources

- `references/four-slot-coach.md`: Four-Slot Tension Coach (`張力句教練`) rules, modes, probes, and safety boundaries
- `references/shape-cards.md`, `references/ctxfst-spec.md`: shape card format and probe specification
- `scripts/shape_layer.py`: build, similar, link, review, eval
