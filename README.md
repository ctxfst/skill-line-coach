# skill-line-coach

> **Coach every problem line as you write it, and find where you hit the same problem before.**

`skill-line-coach` is an agent skill for structuring Markdown notes and diary entries around clear **problem lines (`問題線`)**:

1. **Write-Time Four-Slot Coach (`張力句四槽教練`)**: While writing a diary or note, check that each problem line captures the four role slots—**誰在做 (Actor)**, **動作＋對象 (Action + Object)**, **被什麼擋 (Concrete Blocker)**, and **戰報 (Outcome / State Change)**—and separate mixed passages into standalone lines (`教練` / `評比` / `改寫` / `比較`).
2. **Shape Card Retrieval (`scripts/shape_layer.py`)**: Link notes across years to hand-written **shape cards** (`問題形狀` + `疫苗欄` + `化身清單` + `退回`) using embedding probes and human feedback.

## Files

| Path | Content |
|---|---|
| `SKILL.md` | Agent skill instructions (`skill-line-coach`) |
| `references/four-slot-coach.md` | Four-Slot Tension Coach (`張力句教練`) rules, modes, probes, and boundaries |
| `references/shape-cards.md` | Shape card format and writing guide |
| `references/ctxfst-spec.md` | Probe & shape card specification |
| `scripts/shape_layer.py` | `build`, `similar`, `link`, `review`, `eval` |
| `tests/test_shape_layer.py` | Offline unit tests |

## Quickstart: Shape Card Indexing

```bash
python3 scripts/shape_layer.py --env .env build ~/notes --out index --exclude 'private/*'
python3 scripts/shape_layer.py --env .env link index --cards ~/notes/shapes
# mark index/review.md with [x] / [-]
python3 scripts/shape_layer.py review index --cards ~/notes/shapes
```

Optional line markers inside a multi-topic note:

```markdown
%% line-start id="work:migration-2014" %%
…self-contained problem line…
%% line-end %%
```

## License

MIT. Maintained by [ctxfst](https://github.com/ctxfst).
