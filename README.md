# Augur Engine

A match-time analyst tool for Marvel Snap. Given the cards the opponent has already played, it estimates their deck, likely next moves, and the distribution of card and location strength.

This is a **decision-support tool, not a bot**: it does not play for the user and does not control the game.

## Hard constraints

- Input is **manual only**: the user enters the cards and locations themselves.
- The program does **not** read game memory, client files, the screen (screenshots, OCR), and does **not** intercept network traffic.
- Data is collected **only legally**: the official Reddit API within its rate limits, open datasets, and websites whose terms allow such use. No scrapers that violate site rules.
- **Statistics and card ability texts are not invented.** If data is missing, use explicitly labeled synthetic data and say so. For card effects, ask the user.
- **The LLM does not compute probabilities.** Classical models and the engine produce all numbers.

## Game rules (context)

- 6 turns, 12-card deck.
- 3 locations, revealed on turns 1–3.
- Energy equals the turn number.
- Cards are revealed simultaneously at the end of a turn.
- To win the match, win at least 2 of the 3 locations.

## Architecture

Stack: Java 21 (Maven, built with JDK 25 using `release` 21), JavaFX for the UI, ONNX Runtime for Java for loading models. Python is used for training in `training/`.

| Module | Language | Purpose |
|---|---|---|
| `engine/` | Java | Deterministic engine: board state, reveal order, ongoing recalculation, location effects. No ML |
| `opponent-model/` | Java, Python | Stage A: archetypes (Naive Bayes in Java, then logistic regression in Python). Stage B: LightGBM, exported to ONNX |
| `simulation/` | Java | Monte Carlo: sampling of opponent deck and hand, time budget, intermediate results |
| `ui/` | Java (JavaFX) | Fast input during a match and single-screen output |
| `training/` | Python | Data prep, training, calibration, ONNX export |
| `config/` | YAML | Configuration, including the archetype list |
| `data/` | — | Matches, tables, models. Git-ignored |
| `schemas/` | JSON Schema | Card, location, archetype, match |
| `tests/` | — | Card scenarios and integration tests |

## Decisions

- **Archetypes** live in `config/archetypes.yaml`. The list is extensible; the `Other` class is mandatory.
- **Cards and locations** are described as data (YAML/JSON), not code.
- **An unimplemented effect** is marked `unknown_effect`. Such a card widens the strength interval, and the UI marks the location as "imprecise".
- **Every card with an implemented effect** has a scenario test: "state before → turn → expected strength after".
- **Data is split by match or by time**, not by row.
- **Synthetic data** is marked `synthetic = true`, used only for pipeline tests, and **excluded from quality metrics**.
- **Metrics:** Brier score, log loss, isotonic calibration. A table by turns 2, 3, 4, 5.
- **Randomness:** fixed seeds.
- **Load:** the simulation uses half the CPU cores (8 by default), because the game runs on the same laptop. The number is configurable.
- **Window:** a separate window on the laptop, with an option to stay on top of other windows.

## Stages

1. **Data schema and engine** on the 30 most common cards and 5 locations, with tests for each card.
2. **Monte Carlo** on a stub (opponent picks moves uniformly) and speed measurement.
3. **Minimal fast-input UI** connected to stage 2.
4. **Archetype classifier** (Naive Bayes in Java).
5. **Python pipeline:** LightGBM, calibration, ONNX export, integration into Java.
6. **Card database expansion and validation:** interval coverage, calibration curves.

After each stage: tests, metrics, timing, UI description, and confirmation from the project owner.

**Current stage:** 1, steps 1–4 (folder structure, `.gitignore`, Maven scaffold, schemas). Next step: the engine's card and board model, checked against the schemas.

## Journal

- Folder structure created.
- `.gitignore` and Maven scaffold (parent `pom.xml` and four modules) created; `mvn validate` passes.
- JSON Schemas for card, location, archetype, and match written in `schemas/`; all four are valid JSON. Not yet tested against sample data.

## How we work on this project

- At the start of each conversation, reference this file: "read README.md".
- After each completed step, update the journal and "Current stage".
- Record changes to decisions here, not only in chat.
