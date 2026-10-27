# skill-line-coach

> **Coach every problem line as you write it, and find where you hit the same problem years ago.**

`skill-line-coach` is an agent skill (for Claude Code, Codex, Gemini CLI, and other `SKILL.md`-compatible agents) that helps you write structured, self-contained **problem lines (`問題線`)** in Markdown diaries and notes, and connects recurring problems across years using **Shape Cards**.

---

## Why Four Slots Make Vector Search Work Across Years

People often blame embedding models when vector search fails to connect the same recurring problem across a decade:

- **2013 note**: *"Set up a site wiki engine"*
- **2014 note**: *"Installed Evernote on my phone, voice notes are super convenient"*
- **2023 note**: *"Prompting ChatGPT to build a Zettelkasten card bot"*

Across 15 years of personal notes, plain cosine similarity between notes on this same recurring problem averaged **`0.623`** (barely above the **`0.534`** random baseline)—not because the embedding model is dumb, but because **the invariant half of the problem was never written into the text**:

1. **Un-coached notes only record Slot 2 (the tool or object of the day)**. Tools expire every three years (`Wiki` → `Evernote` → `ChatGPT`), so the embedding model only sees unrelated surface nouns.
2. **Coaching the Four Slots forces you to write down the part that stays invariant across a decade**: **what concrete phenomenon blocked you (Slot 3: Blocker)** and **how that obstacle changed after your action (Slot 4: Outcome)**.
3. Modern embedding models (`voyage-3.5`, etc.) are highly sensitive to relational tension (*who tried what, blocked by what, with what result*)—once that skeleton is actually present in the prose. In testing, writing the role relationships directly into the note raised cross-era cosine similarity from **`0.623` to `0.858`** without changing the retrieval algorithm at all.
4. By contrast, trying to extract four slots or entity graphs *retroactively* from old, half-written notes failed (`0.51`–`0.629` AUC), because nearly half of un-coached notes never recorded the blocker (`46%` missing) or outcome (`39%` missing) in the first place.

**Fixing the note at write time beats fixing the database after the fact.**

---

## 10-Second Example: Four-Slot Line Coach (`張力句四槽教練`)

**Your draft diary line:**
> "Installed the new local search plugin in Obsidian today and moved my project notes over. Really fast and convenient."

**`skill-line-coach` (Default `教練` Coach Mode):**
> - **Structure seen**: You recorded **who** (`I`), **action + object** (`installed the local search plugin and moved project notes`), and praised the tool ("fast and convenient"), but **Slot 3 (Concrete Blocker)** and **Slot 4 (Outcome on that blocker)** are missing.
> - **One probe question**: Before installing this plugin, which specific note or task were you unable to find or complete—and after moving your notes over, did that specific problem go away?

**Your revised problem line (all 4 slots complete & self-contained):**
> "When quoting the client's PCB revision yesterday, I couldn't find last month's MOQ pricing constraints buried across three daily logs (**Slot 3: Blocker**). Today I installed the local search plugin and tagged pricing lines separately (**Slot 1 & 2: Actor + Action**); pulling up past MOQ terms now takes one query, though unindexed PDF attachments still don't show up (**Slot 4: Outcome / State Change**)."

### Four Role Slots

| Slot | Role | What counts |
|---|---|---|
| **1. 誰在做 (Actor)** | Who acts | Explicit subject, or omitted subject when context uniquely identifies one actor |
| **2. 動作＋對象 (Action + Object)** | What was done | Action and target along this problem line |
| **3. 被什麼擋 (Concrete Blocker)** | What stood in the way | A specific, observable scene or obstacle (vague labels like *"bad state"* or *"not enough time"* count as missing) |
| **4. 戰報 (Outcome / State Change)** | Did the obstacle recede? | Direct update on that same obstacle—victory, failure, partial improvement, or stalemate |

### Four Interactive Modes
- **`教練` (Coach, default)**: Summarizes the structure you have in one sentence and asks **one** non-leading probe question. Never invents facts or gives numeric scores.
- **`評比` (Evaluate)**: Marks each slot as `有` (present) or `缺` (missing) per problem line with quotes from your text, plus separates multi-line passages (`分線`).
- **`改寫` (Rewrite)**: Reorganizes your draft while preserving all facts, dates, and qualifiers.
- **`比較` (Compare)**: Compares before/after versions to highlight what structural piece was added.

See [`references/four-slot-coach.md`](references/four-slot-coach.md) for the complete coaching rules and safety boundaries (such as gently bypassing pure lyrical/observational notes without forcing a template).

---

## Installation

Clone into your agent's skills directory:

```bash
# Claude Code
git clone https://github.com/ctxfst/skill-line-coach.git ~/.claude/skills/skill-line-coach

# Shared ~/.agents/skills (for Claude Code, Codex, Gemini CLI, etc.)
git clone https://github.com/ctxfst/skill-line-coach.git ~/.agents/skills/skill-line-coach
ln -s ~/.agents/skills/skill-line-coach ~/.claude/skills/skill-line-coach
ln -s ~/.agents/skills/skill-line-coach ~/.codex/skills/skill-line-coach
```

---

## Recurring Problem Shape Cards (`scripts/shape_layer.py`)

Once notes capture clear problem lines, `scripts/shape_layer.py` links notes across years to hand-written **Shape Cards**—problems that keep returning with different objects and tools.

In leave-one-out evaluations on 72 personal notes across 15 years:

| Method | Mean card AUC |
|---|---:|
| Nearest confirmed note (embedding) | 0.682 |
| LLM judge against the card + examples | 0.780 |
| Card probe with your rejections (`Rocchio`) | 0.834 |
| **Card probe with your rejections + LLM judge** | **0.866** |

```bash
pip install numpy pyyaml

python3 scripts/shape_layer.py --env .env build ~/notes --out index --exclude 'private/*'
python3 scripts/shape_layer.py --env .env link index --cards ~/notes/shapes
# mark index/review.md with [x] (confirm) or [-] (reject)
python3 scripts/shape_layer.py review index --cards ~/notes/shapes
```

Optional line markers inside a multi-topic note:

```markdown
%% line-start id="work:migration-2014" %%
…self-contained problem line…
%% line-end %%
```

## Files

| Path | Content |
|---|---|
| [`SKILL.md`](SKILL.md) | Agent skill instructions (`skill-line-coach`) |
| [`references/four-slot-coach.md`](references/four-slot-coach.md) | Four-Slot Tension Coach (`張力句教練`) rules, modes, probes, and boundaries |
| [`references/shape-cards.md`](references/shape-cards.md) | Shape card format (including `疫苗欄` breakdown vaccine section) |
| [`references/ctxfst-spec.md`](references/ctxfst-spec.md) | Probe & shape card specification |
| [`scripts/shape_layer.py`](scripts/shape_layer.py) | `build`, `similar`, `link`, `review`, `eval` CLI |
| [`tests/test_shape_layer.py`](tests/test_shape_layer.py) | Offline unit tests (`python3 -m unittest discover -s tests`) |

## License

[MIT](LICENSE). Maintained by [ctxfst](https://github.com/ctxfst).
