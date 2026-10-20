# Functional Specifications: Project Mayflower

| | |
| --- | --- |
| **Project Name** | Project Mayflower |
| **Status** | Draft |
| **Team** | Mayflower |
| **Authors** | Noah Levine, Caleb Weisgerber, Ryan Massenburg, Emmett Oliver |
| **Last Updated** | 10/19/2026 |

> **Reading this draft:** Items marked **[OPEN]** still need a team decision and are tracked in [Appendix B](#appendix-b-issues-list). Items marked **[PROPOSED]** are suggested defaults that need confirmation. Remove both markers once they are agreed.

## Version History

| Version | Approved By | Revision Date | Description of Change | Author |
| --- | --- | --- | --- | --- |
| 0.1 | | 10/9/2026 | Added first draft of Section 1 | Emmett Oliver |
| 0.2 | | 10/13/2026 | Revised Section 1.2 and started draft of 2.1 / 2.2 | Emmett Oliver |
| 0.3 | | 10/14/2026 | Drafted Sections 2.7 / 2.8 | Emmett Oliver |
| 0.4 | | 10/15/2026 | Revised Sections 1.1 / 1.2 based on vision proposal | Emmett Oliver |
| 0.5 | | 10/16/2026 | Updated Sections 1.2, 1.4, 2.7, 2.8 | Emmett Oliver |
| 0.6 | | 10/18/2026 | Converted to Markdown. Fixed 1.4 definitions and engine naming. Updated 1.2 scope (4-player only, player trading, bot comparison, Colonist testing). Drafted 2.2–2.6, Section 3 and both appendices from team answers | Caleb Weisgerber |
| 0.7 | | 10/19/2026 | Edited Items Brought Emett | Caleb Weisgerber |


[OPEN] I-11: Does the instructor need to sign the "Approved By" column?

## Table of Contents

1. [Introduction](#1-introduction)
2. [Product Overview](#2-product-overview)
3. [Requirements](#3-requirements)
- [Appendix A. Analysis Models](#appendix-a-analysis-models)
- [Appendix B. Issues List](#appendix-b-issues-list)

---

## 1. Introduction

### 1.1 Purpose

This document specifies the functional, interface, and behavioral requirements for Project Mayflower, a web-based game reviewer and practice platform for Settlers of Catan. Project Mayflower will draw similarties to Chess.com's Game Review/ board anylasis. Players upload recorded matches to determmine detrmine how to improve gameplay. Our tool will allow for the diagnose critical blunders, explore branching lines, and practice against AI opponents.

This specification governs:
- architectural decomposition;
- API contracts;
- background simulation job interfaces;
- data capture schemas;
- automated validation gates across the three-quarter engineering lifecycle.

It sets acceptance criteria for deciding when deliverables are complete, and ensures the system meets the course requirements.

### 1.2 Scope

The platform is a web-based teaching and analysis tool. It helps players analyze starting board placements and strategic choices throughout the game. Our platform will let players practice against our diverse AI opponents.

Project Mayflower works as a post-game review suite and a sparring engine:
1. A client-side extension records match data from the player's own games on Colonist.io.
2. The move stream is fed into our Game Engine.
3. The system computes changes in win probability to give explainable feedback on actions taking in game.

The platform targets **base-game, 4-player Catan only**, and includes **player-to-player trading** while respecting the hidden card of other players.

The project has a research component as well as the product. It compares four bot approaches under identical rules and time limits:
- AlphaBeta search;
- Monte Carlo Tree Search (MCTS);
- AlphaZero-style search (MCTS guided by a neural network);
- Proximal Policy Optimization (PPO) reinforcement learning.

#### In scope

**Phase 1 (Quarter 1): Walking Skeleton / Core Prototype**

- **The Collector (Chrome Extension):** A read-only extension for Chromium-based browsers. It records game events and turn actions from the user's own Colonist.io session and exports serialized match files. *(Built.)*
- **Game Engine (Core):** A custom, headless implementation of base 4-player Catan rules, including player-to-player trading. It handles board topology, legal action validation and turn progression. It can replay recordings exported by the Collector (see FR-03 for what "replay" can guarantee under hidden information).
- **Initial Bot Implementations:** First working versions of the bot approaches under comparison, behind a common bot interface. They are benchmarked against a random baseline. **[OPEN] I-5:** which of the four approaches must be running by the end of Q1.
- **Opening Placement Reviewer:** User can practice initial settlement bot will simulate game and determine if opening was good/bad.
- **Initial Review UI:** A web client for uploading game files, rendering the board, and auditing setup placements.
- **Team Data Logging:** The team records every Colonist game it plays, to seed the human game dataset.

[OPEN] I-5: Phase 1 may be overloaded. The Game Engine and the bots are the Q1 priority. The Opening Reviewer, Review UI and job queue may move to Phase 2.

**Phase 2: Minimum Viable Product**

- **Asynchronous Job Dispatch:** A task queue that spreads simulation rollouts across background worker processes without slowing the UI.
- **Post-Placement Diagnostic & Blunder Review:** Automated comparison of candidate placements, showing changes in win probability and the risk of being starved of resources.
- **Full Move-by-Move Review:** Automated grading of mid-game decisions: builds, robber moves, knight and development card plays, and trades.
- **First Learning & Self-Play Bot:** Self-play training that produces synthetic game datasets to supplement human recordings.
- **4-Player Sparring / Practice Mode:** The user plays complete 4-player games against three bots.
- **Basic Trade Utility & Leader-Threat Model:** Scores a trade offer by its value to the player, weighed against how much it advances the table leader.
- **Real-Player Testing on Colonist:** Team members play Colonist games, making the moves the bot suggests, to measure how it performs against real people. The bot never plays on Colonist by itself.
- **Persistence & Account Layer:** A PostgreSQL database storing user accounts, uploaded recordings, review reports, and practice session histories. *(No accounts or user data in Phase 1.)*
- **Statistical Calibration Audit:** Checks whether predicted win probabilities match observed outcomes across real and simulated matches.

**Phase 3: Extensions / Advanced Capabilities**

- **What-If Branching:** Freezes the game at any turn so the user can try different moves and compare simulated outcomes branch by branch (like Git branches for a Catan game).
- **Structured Table-Talk & Persuasion Engine:** Game-state-grounded message intents that let a bot propose trades, point out the table leader, or stay quiet.
- **Configurable Bot Archetypes:** Parameterized play styles that simulate distinct opponent personalities.
- **The Arena:** Automated bot-vs-bot tournaments with Elo ratings, including benchmarks against external bots.
- **Bot Comparison Write-Up:** A report comparing the four bot approaches, based on Arena results.
- **In Game Habit Diagnostics:** Cross-game analytics that surface a user's recurring bad habit(s).
- **Cloud Worker Deployment & Scaled Storage:** Cloud worker nodes and object storage for historical game records and model checkpoints.

#### Out of scope

- **Automated platform scraping or active injection:** The system never injects gameplay scripts, makes moves on third-party servers, scrapes lobbies, or captures data from other people's accounts. The Collector runs only client-side and read-only, in the user's own browser session.
- **Autonomous bot play on Colonist and live in-game advice:** Bots never play on Colonist. Real-player testing (Phase 2) is done by a human team member entering the bot's suggested moves, for evaluation only. The platform does not advise users during their normal Colonist games.
- **Human-vs-human online multiplayer:** No matchmaking lobbies, peer-to-peer networks, or unmoderated chat rooms. All play inside the platform is between the user and AI agents.
- **1v1 / 2-player games:** 2-player Catan plays differently. The platform supports 4-player games only.
- **Unconstrained free-form chat:** Table talk is limited to structured, game-state-grounded dialogue. General-purpose chatbot conversation is excluded. 
- **Complete game-theoretic solution:** Catan has dice, hidden information and open negotiation, so the bot does not claim to play optimally. It relies on search and learned evaluation.
- **Native mobile apps:** No iOS or Android apps. The interface is a desktop web application.
- **Catan expansions:** Cities & Knights, Seafarers, and 5–6 player boards are excluded (possible future stretch goals).

### 1.3 Intended Audience

| Audience | Why They Need This Document |
| --- | --- |
| Development Team | Defines component boundaries, API contracts, job queue schemas, and who implements what across the three-quarter cycle. |
| Course Instructor | Checks that the system shows software engineering rigor, follows course policies, and gives a defensible basis for evaluation. |
| QA & Verification Testers | Defines expected inputs, processing states, outputs, and edge cases for automated unit, integration, and end-to-end tests. |
| End Users (Competitive & Casual/Intermediate Players) | Explains supported features, depth of analysis, and system limits. |

### 1.4 Definitions, Acronyms, and Terms

#### 1.4.1 Game-Specific Terms

| Term | Definition |
| --- | --- |
| Catan / Settlers of Catan | The board game itself: resource management, trading, and settlement expansion on a hexagonal board. This project supports base-game, 4-player matches only. |
| Colonist.io | The online platform where people play Catan in the browser. The Collector records the user's own games played there. |
| Maritime (Bank) Trade | Trading resource cards with the bank instead of another player: 4 of one resource for 1 of any other, or 3:1 / 2:1 when the player has a settlement or city on a matching port. |
| VP (Victory Points) | Points earned through settlements, cities, certain development cards, and special achievements. A player wins by reaching at least 10 points on their turn. |
| Table Leader | The player estimated to be closest to winning, considering visible points and other signs of progress. |
| Hidden Information | Information a player cannot directly observe, such as opponents’ resource cards and unplayed development cards. |

#### 1.4.2 Statistics and Evaluation Terms

| Term | Definition |
| --- | --- |
| Win Probability | An estimate of a player’s chance of winning from a given position, using only information available to that player. |
| EV (Expected Value) | The average outcome of a decision, weighted by the probabilities of possible outcomes. For example, expected resource production accounts for how likely each dice result is. |
| Win Rate | The proportion of completed games won by a player or bot. |
| Elo | A rating system that uses results and rating differences to estimate relative playing strength. Its use for 4-player games requires a defined adaptation. |
| Calibration | How well predicted win probabilities match reality. If the bot says a player has a 30% chance to win across many games, that player should win about 30% of those games. |
| Brier Score | A single number measuring how far off win-probability predictions were from actual results. 0 is perfect; lower is better. Used to track calibration. |

#### 1.4.3 Computer Science and AI Terms

| Term | Definition |
| --- | --- |
| Information Set | The set of possible game states consistent with everything a player has observed. Used to reason about hidden cards. |
| Policy | A strategy that determines which action a bot takes from the information available to it. |
| Rollout | A simulated continuation of a game, using bot policies to sample possible outcomes. |
| AlphaBeta (Alpha-Beta Pruning) | A technique used with minimax search to skip branches that cannot change the selected move. Applying it to Catan requires handling chance, multiple opponents, and hidden information. |
| MCTS (Monte Carlo Tree Search) | A search algorithm that builds a tree of possible actions and uses repeated simulations to estimate their strength. |
| ISMCTS (Information Set Monte Carlo Tree Search) | A family of MCTS methods designed for games with hidden information. |
| AlphaZero-Style | An approach combining MCTS with a neural network trained through self-play to guide move selection and position evaluation. |
| PPO (Proximal Policy Optimization) | A reinforcement learning algorithm that improves a policy using experience while limiting how much it changes during each update. |

#### 1.4.4 Project Components and System Terms

| Term | Definition |
| --- | --- |
| The Collector | A client-side, read-only Chrome extension that records events from the user’s own Colonist.io session and exports JSON game files using the `colonist-session-collector/v1` format. |
| Game Engine | The project’s custom, headless implementation of 4-player Catan rules. It validates actions, advances game states, replays recordings, and runs simulations. |
| The Arena | An internal test environment for automated bot-vs-bot tournaments, win-rate comparisons, and ratings across bot versions. |
| What-If Branching | Creating an alternative continuation from a recorded position by substituting a different move and comparing simulated outcomes. |
| Job Queue | A to-do list on the server. The website adds slow tasks (e.g. "review this game") to the queue so pages stay responsive while the work runs in the background. |
| Worker Node | A background program that takes tasks off the job queue, runs them (rollouts, reviews, self-play games), and saves the results. |

### 1.5 Reference Documents

| Document | Version / Date | Notes |
| --- | --- | --- |
| CS 481 Course Syllabus | Fall 2026 | Project scope, engineering practices, team requirements, and presentation milestones. |
| Official Settlers of Catan Rules | Catan Studio | Base rule set: board layouts, dice, development cards, victory conditions. |
| Project Mayflower Vision (`docs/VISION.md`) | Oct 2026 | Product direction and research questions. |
| Project Mayflower Design Document (`docs/DESIGN.md`) | Draft | Architecture and component designs. |
| Catanatron (github.com/bcollazo/catanatron) | GPL-3.0 | Reference for ideas and an external Arena opponent. |

### 1.6 Document Conventions

- **Must:** A mandatory requirement for compliance and delivery.
- **Should:** A recommended capability, to be implemented unless technical constraints prevent it.
- **May:** An optional feature reserved for later quarters.
- **FR-XX:** Functional Requirement.
- **NFR-XX:** Non-Functional / Performance Requirement.
- **UI-XX:** User Interface Requirement.
- **[OPEN] / [PROPOSED]:** Pending team decision (see Appendix B).

---

## 2. Product Overview

### 2.1 Product Context

Project Mayflower is a web-based platform for learning Settlers of Catan strategy, practicing against bots, and auditing decisions. Board game platforms have traditionally focused on player-vs-player matchmaking, graphics, and recreation. Project Mayflower instead treats competitive board games as a matter of decision support, algorithmic coaching, and analysis under uncertainty. It is functionally analogous to the chess analysis platforms on Chess.com's Game Review or the Stockfish engine on Lichess. Unlike chess, Catan has high variance and imperfect information, so this tool is built specifically for that kind of game.

### 2.2 User Classes and Characteristics

Project Mayflower has four classes of users and technical stakeholders, each with different technical skill, frequency of use, and expectations.

| User Class | Main Features Used | Frequency | Technical Skill | Expectations |
| --- | --- | --- | --- | --- |
| **Competitive Players** (ranked Colonist players) | Move-by-move review, what-if branching, practice against hard bots | After most games | Can install an unpacked Chrome extension and read win-probability graphs | Precise, credible grades; strongest bots; depth over simplicity |
| **Casual / Intermediate Players** | Opening placement review, plain-language reviews, practice against easier bots | Occasional | Basic web use: install the extension, press Record | Clear explanations without statistics jargon; encouraging feedback |
| **Researchers / Project Team** | Arena tournaments, bot comparison, self-play datasets, calibration audits | Continuous during development | High: Python, ML tooling, Docker | Reproducible experiments, exportable data, consistent bot interfaces |
| **Administrators / Maintainers** [PROPOSED] | Worker and queue health, model deployment, data retention | As needed | High | Logs, monitoring, safe redeploys, data deletion tools |

[OPEN] I-8: confirm the fourth class (Administrators / Maintainers).

### 2.3 Overview of Functional Requirements

| Area | Capability | Phase |
| --- | --- | --- |
| Capture | Record your own Colonist games and export them as JSON | 1 (built) |
| Engine | Base 4-player rules, player trading, hidden-information views, replay of recordings | 1 |
| Bots | Common bot interface; AlphaBeta, MCTS, AlphaZero-style and PPO implementations | 1–2 |
| Opening Review | Rank initial placements and explain why | 1 |
| Move Review | Grade every decision by change in win probability compared with the bot's best move | 2 |
| Practice | Play a full 4-player game against three bots at selectable difficulty | 2 |
| Trade Valuation | Score a trade against the table leader's progress | 2 |
| Accounts & History | Save recordings, reviews, and sessions per user | 2 |
| Calibration | Check that predicted win chances match outcomes | 2 |
| What-If | Branch from any turn and compare outcomes | 3 |
| Table Talk | Structured bot messages (propose trade, name the leader, stay quiet) | 3 |
| Arena | Bot-vs-bot tournaments with Elo | 3 (internal use earlier) |
| Habits | Cross-game patterns | 3 |

Detailed requirements are in Section 3.2.

### 2.4 Overview of Data Requirements

| Data | Source | Stored Where | Notes |
| --- | --- | --- | --- |
| Raw Colonist recordings | Collector export (JSON) | On Server after user upload | Contains the recorder's own hand, other players' card **counts** only, public game logs, chat, and usernames |
| Parsed games and moves | Game Engine replay of recordings | Database | Includes game settings (friendly robber, dice mode, discard limit) |
| Synthetic game datasets | Self-play and Arena | File/object storage | Large; versioned with the bot model that generated them |
| Practice sessions | Practice mode | Database | Full move history, bot versions, and difficulty |
| Reviews | Review workers | Database | Per-move grades, best alternatives, win-probability changes |
| Bot models and checkpoints | Training | File/object storage | Versioned; one or more per difficulty level |
| Arena results and Elo | Arena | Database | Feeds the bot comparison write-up |
| User accounts | Phase 2 only | Database | None in Phase 1 |

**Privacy:** A recording reveals only the recording player's own cards. However, it does contain **other players' usernames and chat messages**.
- [OPEN] I-13: Should we anonymize usernames on upload?
- [OPEN] I-13: Should we keep chat? The negotiation research needs it.
- [OPEN] I-13: How long are uploads retained?

[OPEN] I-2: Target number of real recorded games per phase.

### 2.5 Operating Environment

- **Collector:** Chromium-based desktop browsers (Chrome 118+, Brave). The extension uses Manifest V3 and the `chrome.debugger` API, so Firefox and Safari are not supported. It was developed mainly in Brave. [OPEN] I-10: verify it works in Google Chrome and Edge.
- **Web client:** Current desktop Chromium browsers, served by **nginx**.
- **Backend:**
  - Python **3.11** (Game Engine, bots, training, API).
  - **PyTorch with CUDA** for learning bots.
  - Speed-critical engine code may later be rewritten in Rust or C (see I-6).
- **Containers:** Docker Desktop with Docker Compose on every developer machine.
- **Developer hardware:** Consumer laptops/desktops (4–8 CPU cores, 8–16 GB RAM).
- **Training hardware:**
  - Team GPUs: RTX 50-series cards need PyTorch builds for CUDA 12.8 or newer.
  - The university compute cluster, if available.
  - Fallback: Google Colab or another low-cost cloud provider.
  - [OPEN] I-14: inventory of team GPUs and university cluster access.

### 2.6 General Constraints, Assumptions, Dependencies, Guidelines

**Constraints**
- **Budget:** Perferable low cost training on personal hardware. If speed becomes an issue we can look into Google Colab or another cloud option.
- **Timeline:** Three quarters. Quarter one team reflections on Oct 23, Nov 13, and Dec 4.
- **Team:** Four developers. [OPEN] I-15: ownership of Engine, Bots/Training, API/Web, Collector, and Docs.

**Assumptions**
- Colonist.io's message format stays stable enough for the Collector's decoder. If it changes, the decoder must be updated.
- Recording only your own games and read-only
- Bots of the planned sizes run within the per-turn time limit on consumer hardware. Doesnt make sense to practice against bots that take forever to play.

**Dependencies**
- Colonist.io (main online Game platform)
- Chrome Extension APIs (`debugger`, `storage`)
- Python libraries (PyTorch and others)
- Docker / Docker Compose
- GitHub and GitHub Actions (CI/CD)
- nginx
- PostgreSQL (Phase 2)
- A job queue library (I-9)
- An LLM provider for table talk, likely via OpenRouter (Phase 3; I-4)

### 2.7 Design and Implementation Constraints

The design and implementation of Project Mayflower are bounded by technical, architectural, operational, and academic constraints. They guide the trade-offs between simulation performance, client responsiveness, model training overhead, and course compliance.

**Game Engine & State Representation**

- **Game Engine:** The team will build its own headless Game Engine instead of using a third-party library. This gives us full control over simulation speed, state-copying overhead, and hidden-information modeling (unrevealed development cards and concealed hands) and licencing. 
- **Rule fidelity and settings:** The engine must implement base 4-player Catan rules, board layouts, turn sequences, player-to-player and maritime (bank) trades, and victory conditions. Rule variants that differ between Colonist games (e.g. friendly robber, dice mode, discard limit) must be configurable per game, so that recorded games replay under their own settings.
- **Deterministic execution:**
  - Simulated games must be reproducible from an explicit random seed, for regression testing in CI/CD.
  - Replays of Collector recordings must use the **dice results recorded in the game**, not a seed, because Colonist's random number generator is not ours.
- **Information set support:** The engine must separate public board data from hidden opponent hands. Opponent hands appear in recordings only as card counts, and robberies between two opponents are hidden. Bots and replays must therefore reason over possible hands rather than assume they know them.
- **Performance path:** The engine is written in Python first. Profiled hot paths may later be rewritten in Rust or C, and batched simulation or neural-network work may move to CUDA (I-6).


**Asynchronous Compute & Task Architecture**

- **Non-blocking analysis:** Heavy work (move evaluations, rollouts, and self-play training matches) must never run inside a web API request. Any work over 100 ms must go through the task queue to background workers.
- **Stateless workers:** Workers must be stateless. Each job ticket is self-contained, specifying the board, turn actions, candidate moves, and seed values.
- **Persistence isolation:** Intermediate rollout steps stay in memory. Workers write aggregated metrics and reports to storage only when a batch or turn review finishes, to avoid database lock contention.

**Hardware, Compute Budget & Scaling**

- **$0 local development:** All core development, testing, and prototypes for Phases 1 and 2 must run locally under Docker Compose with no cloud cost.
- **Hybrid training compute:** Training and large self-play runs may use local GPUs or free/low-cost services (e.g. Google Colab) instead of permanent paid servers. Small test runs must come before full-scale runs.
- **Bot time limit:** A bot must return a legal move within **30 seconds per turn** on consumer hardware. Faster is preferred; see NFR-01.

**Academic, Security & Software Engineering Hygiene**

- **Course policy compliance:** The platform is a decision-support, game review, and analytical sparring tool. Commercial multiplayer lobbies, unmoderated human chat rooms, and cosmetic gaming incentives are excluded.
- **Table talk and language models:** Bot communication is limited to structured game-state intents (e.g. trade offers, identifying the table leader, staying quiet). Open-ended conversational bots unrelated to the current game are out of scope.
- **Branch protection and CI/CD:** No direct pushes to `main`. All code merges through Pull Requests that need at least one peer review and passing build, lint, and test checks.
- **Secret management:** Database passwords, authentication keys, and external API tokens (including LLM provider keys) must never be committed to source control. They live in `.env` files listed in `.gitignore`.

### 2.8 User Documentation

Project Mayflower will provide documentation for competitive players, casual learners, course evaluators, and contributing developers. It comes in four forms: guidance inside the web app, installation runbooks, developer guides kept in version control, and automatically generated API docs.

**In-App Guidance & Tooltips**

- **Collector installation and usage guide:** Step-by-step instructions in the web app for loading the unpacked Collector extension, recording a Colonist.io match, and exporting and uploading the file.
- **Game review walkthrough:** An onboarding tour of the post-match review timeline and turn-by-turn evaluations. *(The branching portion arrives in Phase 3.)*
- **Placement audit walkthrough:** A guided, dismissible tour showing how to set up a custom board, choose candidate placements, and run rollout audits.
- **Metric definition tooltips:** Plain-language hover explanations of analytical terms, for players without a statistics background.
- **Review label tooltips:** Hover definitions of review labels and analytical metrics.
- **Statistical confidence notices:** Badges telling the user when two candidate moves are statistically too close to separate because of dice variance.

**User Guide & Strategy Manual**

- **Online quickstart guide:** A web guide to the core workflows: setting up an opening board, running simulations, and choosing sparring bots.
- **Blunder interpretation guide:** Explains how the system flags blunders, how dice luck is separated from decisions, and how trade value is weighed against the table leader.
- **Sample case studies:** Pre-loaded board scenarios showing an analysis from start to finish.
- **Habit guide (Phase 3):** How to read cross-game trends, such as holding development cards too long or undervaluing ports.

**Technical & Operational Documentation**

- **Repository `README.md` and setup runbooks:** Local installation and deployment: prerequisites, Docker Compose, and `.env` configuration.
- **API documentation:** Generated OpenAPI (Swagger) documentation of all REST endpoints, request/response schemas, and job-status contracts.
- **Architecture and contribution guidelines (`CONTRIBUTING.md`):** Team engineering standards, branch protection rules, local pre-commit testing, worker task interfaces, and the definition of done.
- **Arena and bot benchmarking guide (Phase 2/3):** Runbooks for running bot-vs-bot tournaments, computing Elo ratings, and benchmarking bots against external baselines.

---

## 3. Requirements

### 3.1 External Interface Requirements

#### 3.1.1 User Interfaces

| ID | Interface | Phase |
| --- | --- | --- |
| UI-01 | **Collector popup and viewer:** start/stop recording, status badge (REC / PAUSE / ERR / OFF), recordings list, JSON export | 1 (built) |
| UI-02 | **Recording upload page:** select and upload a Collector export; shows validation errors | 1 |
| UI-03 | **Board viewer:** renders the board, buildings, robber, and the recorder's hand at any turn; step forward and back through the game | 1 |
| UI-04 | **Opening placement review / practice:** the board with candidate placements ranked, plus an explanation of each | 1 |
| UI-05 | **Move-by-move review:** win-probability graph over the game, per-move grade, the bot's best alternative, keyboard navigation | 2 |
| UI-06 | **Practice table:** the user against three bots in a 4-player game; difficulty selection; build, trade, robber and development card actions | 2 |
| UI-07 | **Account pages:** sign up, log in, history of recordings, reviews and sessions | 2 |
| UI-08 | **What-if explorer:** branch from any turn, try a different move, compare branches side by side | 3 |
| UI-09 | **Habits dashboard and Arena leaderboard** | 3 |

#### 3.1.2 Hardware Interfaces

No special hardware. Users need a desktop or laptop that runs a Chromium-based browser. Training uses NVIDIA GPUs through CUDA (local, a university cluster, or cloud). No other devices, sensors, or peripherals are involved.

#### 3.1.3 Software Interfaces

| Interface | Description |
| --- | --- |
| Collector → Web App | The JSON export file (`schema: colonist-session-collector/v1`), uploaded by the user. |
| Web Client → API | REST/JSON endpoints served behind nginx: upload a recording, get the board at a turn, request a review, get a review, start practice, submit a move, check job status. Documented with OpenAPI. |
| API → Workers | A job queue with self-contained tickets (board, actions, candidate moves, seeds, bot version). Workers run locally (Phases 1–2) or in the cloud (Phase 3). |
| Game Engine API | `legal_moves(state)`, `apply(state, move)`, `view_for(state, player)`, `winner(state)`, plus a replay loader for Collector recordings. |
| Bot Interface | Every bot implements the same `choose_move(view, time_limit)` and `win_probability(view)`, so the Arena, review, and practice can use any bot. |
| Database | PostgreSQL (Phase 2) for accounts, recordings, reviews, sessions, Arena results. |
| LLM Provider | OpenRouter (or similar) over HTTPS for Phase 3 table talk. [OPEN] I-4. |

#### 3.1.4 Communications Interfaces

- **HTTPS everywhere.** Locally, nginx uses a locally trusted certificate (e.g. `mkcert`). Any deployed instance uses a free Let's Encrypt certificate.
- **Practice games use a WebSocket**, so bot moves are pushed to the browser as they happen.
- **Review job status.** Phase 1 uses **polling**: the client asks about the job's status every few seconds. Later, workers may publish a "job finished" message on a pub/sub channel (e.g. Redis pub/sub), which the API forwards to the browser over the WebSocket. [PROPOSED]
- **Collector:** never sends anything to Colonist. It talks only to the browser's debugger API and local storage.

### 3.2 Functional Requirements

Each requirement follows this template: **Purpose / Inputs / Processing / Outputs**. "Done" criteria are listed for testing.

#### FR-01 Record a Colonist Game (Collector) *(Phase 1, built)*

- **Purpose:** Capture the user's own Colonist game for later review.
- **Inputs:** The user's active Colonist.io tab; Start/Stop actions in the popup.
- **Processing:**
  - Attach the browser debugger to that tab only (HTTPS `colonist.io` and its subdomains).
  - Record WebSocket and HTTP game messages.
  - Redact credentials found in URLs.
  - Mark any gap caused by an unexpected stop.
- **Outputs:** A JSON export file and a status badge.
- **Done when:** A full game recorded from setup to result matches the board on screen (already checked against screenshots in `PROTOCOL.md`).

#### FR-02 Validate an Uploaded Recording *(Phase 1)*

- **Purpose:** Reject bad files before they reach the engine.
- **Inputs:** An uploaded JSON file.
- **Processing:** Check the size limit and the schema version. Check that the recording contains the game start and a game state. Detect gap markers.
- **Outputs:** Either an accepted recording or an error message ("This file can't be read", "Recording is missing the start of the game", "Recording has gaps; only complete sections will be reviewed").

#### FR-03 Replay a Recorded Game in the Engine *(Phase 1)*

- **Purpose:** Turn a recording into a sequence of engine states for viewing and review.
- **Inputs:** A validated recording.
- **Processing:**
  - Build the board and game settings from the initial game state.
  - Apply each recorded action, using the **recorded dice results**.
  - Track opponent hands as **sets of possible hands**, since recordings show only card counts and robberies between opponents are hidden.
- **Outputs:**
  - A per-turn state for the recorder, with exact information.
  - Each opponent's possible hands.
  - Any mismatch between the engine and the recording is flagged.
- **Done when:** Public state (buildings, roads, robber, VP, card counts) matches the recording at every turn for the team's recorded games.

#### FR-04 Enforce Base 4-Player Rules *(Phase 1)*

- **Purpose:** Be the single source of truth for legal play.
- **Inputs:** A game state and a proposed move.
- **Processing:**
  - Generate all legal moves: setup, roll, build, development cards, robber, discard, maritime trades, and player-to-player trade offers, responses and confirmations.
  - Apply a move and check for a winner.
  - Apply per-game settings (friendly robber, discard limit, dice mode).
- **Outputs:** A list of legal moves, the new state, or a rejection with a reason.
- **Done when:** A rule test suite derived from the rulebook passes. Thousands of random games finish without errors. Illegal moves are always rejected.

#### FR-05 Hidden-Information Views *(Phase 1)*

- **Purpose:** Make sure bots and reviews use only what a player could know.
- **Inputs:** A game state and a player seat.
- **Processing:** Mask opponents' resource identities and unplayed development cards. Expose public information (card counts, public logs, the board).
- **Outputs:** A player view.
- **Done when:** No bot or review code can read hidden data through the view (enforced by tests).

#### FR-06 Common Bot Interface and Initial Bots *(Phase 1–2)*

- **Purpose:** Compare the four approaches fairly.
- **Inputs:** A player view and a time limit.
- **Processing:** Each bot (AlphaBeta, MCTS, AlphaZero-style, PPO, plus a random baseline) picks a legal move within the time limit and estimates its win probability.
- **Outputs:** A move and a win probability.
- **Done when:** Each implemented bot beats the random baseline over a fixed number of seeded games, and never returns an illegal move or exceeds the limit. [OPEN] I-5: which bots are due in Q1.

#### FR-07 Opening Placement Review *(Phase 1)*

- **Purpose:** Teach good opening placements.
- **Inputs:** A board (from a recording or set up by hand) and the user's chosen placements.
- **Processing:** Evaluate candidate settlement and road placements with rollouts. Compute win probability and the factors behind it: pips, resource variety, ports, and expansion room.
- **Outputs:** Ranked candidates, the user's placement compared with the best one, plain-language reasons, and an "about equal" notice when results overlap statistically.

#### FR-08 Move-by-Move Review *(Phase 2)*

- **Purpose:** Separate decision quality from luck.
- **Inputs:** A replayed game (FR-03).
- **Processing:** For each of the recorder's decisions, compare the win probability after **their** move with the win probability after the **bot's best** move, using only the recorder's information at that time.
- **Outputs:** For each move: a grade, the change in win probability, the best alternative, and the confidence.
- **Label names and thresholds:** chess.com-style labels (best / good / inaccuracy / mistake / blunder) or our own. [OPEN] I-3.

#### FR-09 Practice Mode *(Phase 2)*

- **Purpose:** Let the user spar against bots.
- **Inputs:** Difficulty selection and the user's moves.
- **Processing:** Run a 4-player game of the user against three bots, with bot turns streamed over the WebSocket. Includes player-to-player trades between the user and the bots.
- **Outputs:** A complete game, saved as a practice session (with accounts, Phase 2) and available for review.

#### FR-10 Trade Valuation *(Phase 2)*

- **Purpose:** Judge whether a trade helps the player.
- **Inputs:** A proposed trade and a player view.
- **Processing:** Estimate the change in win probability for both sides. Weigh in how much the trade helps the table leader, and the likely hands of the other party.
- **Outputs:** A trade score with an explanation. This is used by the review, the practice bots, and table talk.

#### FR-11 Real-Player Testing on Colonist *(Phase 2)*

- **Purpose:** Measure bot strength against real people.
- **Inputs:** Live game state, viewed by a team member.
- **Processing:** A team member manually plays the moves the bot suggests. The game is recorded with the Collector.
- **Outputs:** Win rate and placement results against human opponents.
- **Constraint:** No automated input to Colonist.

#### FR-12 Accounts and History *(Phase 2)*

- **Purpose:** Persist the user's recordings, reviews, and sessions.
- **Inputs:** Sign-up / log-in credentials.
- **Processing:** Authenticate the user and link their data to the account.
- **Outputs:** History pages.
- **Login method:** email/password or OAuth. [OPEN] I-16.

#### FR-13 Calibration Audit *(Phase 2)*

- **Purpose:** Check that "30% to win" really means about 30%.
- **Inputs:** Predicted win probabilities and actual outcomes from real and simulated games.
- **Processing:** Group predictions into bins and compare them with observed win rates. Compute the Brier score.
- **Outputs:** A calibration report.

#### FR-14 What-If Branching *(Phase 3)*

- **Purpose:** Explore alternative lines.
- **Inputs:** A turn in a game and an alternative move.
- **Processing:** Create a branch from that turn, simulate play-outs, and keep each branch linked to its parent (like Git branches).
- **Outputs:** A comparison of the branches' win probabilities and typical play-outs.

#### FR-15 Table Talk *(Phase 3)*

- **Purpose:** Study and use negotiation.
- **Inputs:** A player view.
- **Processing:** Choose an intent (propose trade, name the leader, "I'm not a threat", team up against the leader, stay quiet) and turn it into text, using templates and/or an LLM.
- **Outputs:** A chat message, or silence. [OPEN] I-4.

#### FR-16 Arena *(internal from Phase 2, full in Phase 3)*

- **Purpose:** Benchmark bots.
- **Inputs:** Bot versions, number of games, seeds.
- **Processing:** Run 4-player tournaments, rotating seats to cancel out turn-order advantage. Update Elo ratings. External bots (e.g. Catanatron) run as separate processes behind the bot interface.
- **Outputs:** A leaderboard and per-matchup statistics.

#### FR-17 Habit Diagnostics *(Phase 3)*

- **Purpose:** Surface a user's recurring mistakes.
- **Inputs:** All of a user's reviewed games.
- **Processing:** Combine grades by decision type.
- **Outputs:** The top recurring habits (e.g. "you hold development cards too long"), shown only after a minimum number of games. [OPEN] I-17: the minimum number.

### 3.3 Performance Requirements

[PROPOSED] Targets to confirm after the Phase 1 engine prototype is measured.

| ID | Requirement |
| --- | --- |
| NFR-01 | A bot must return a move within **30 s** per turn (hard limit). Targets: easy ≤ 1 s, medium ≤ 3 s, hard ≤ 10 s. |
| NFR-02 | Opening placement review completes within **30 s** on a developer laptop. |
| NFR-03 | A full game review completes in the background within **10 min**. |
| NFR-04 | Engine throughput is measured in Phase 1 (random 4-player games per second on one core). A target is then set to support self-play. Hot paths move to Rust/C if the target is missed (I-6). |
| NFR-05 | API requests that don't start a job respond within **500 ms**. Anything over **100 ms** of computation is queued (Section 2.7). |
| NFR-06 | The system supports about **10 concurrent users** (team and testers). |
| NFR-07 | Upload limit: [OPEN] set from the size of real exports. |

### 3.4 Security

- **Collector:**
  - Attaches only to HTTPS Colonist domains (lookalike domains are rejected).
  - Read-only, with no content scripts.
  - Redacts credentials in URLs.
  - Keeps recordings local until the user exports them.
  - Exported payloads may contain private data; the user is warned before sharing.
- **Uploads:** Size-limited, validated against the schema, stored privately, never executed.
- **Rendering:** Chat and usernames are displayed as plain text, so they can't inject anything into the page.
- **Transport:** HTTPS only (Section 3.1.4).
- **Accounts (Phase 2):**
  - Passwords hashed with bcrypt or argon2.
  - Login attempts rate-limited.
  - Sessions expire.
  - Users can delete their account and their data.
- **Secrets:** Kept in `.env` files and never committed (Section 2.7).
- **Personal data:** Other players' usernames and chat are handled according to I-13.

### 3.5 Usability

- Every grade and metric has a plain-language explanation and a tooltip.
- "About equal" is shown instead of a grade when two moves are statistically close.
- Player colors are distinguishable for color-blind users.
- Reviews can be navigated with the keyboard (arrow keys step through moves).
- First-time users are guided through installing the Collector (Section 2.8).

### 3.6 Other Requirements

- **Colonist.io terms of service:** Colonist's ToS prohibits bots and scraping. The team acknowledges that the Collector may conflict with these terms.
  - **What the Collector does:** It reads the game messages that Colonist already sends to the user's own browser tab during a game the user is playing: the same information shown on screen, saved automatically instead of written down by hand. It records only the user's own games, is read-only, sends nothing to Colonist, clicks nothing, and automates no gameplay.
  - **What the project never does:** Crawl or scrape other players' games, lobbies, or profiles; access other accounts; or let a bot play on Colonist.
  - **Risk:** Colonist could suspend accounts used for recording. Status and instructor guidance are tracked as I-18.
  - **Fallback:** If the Collector cannot be used, game data comes from practice-mode games on our own platform and from self-play. Neither involves Colonist.
- **Open-source licenses:** Third-party code is used only under compatible licenses, with attribution.
  - Catanatron (GPL-3.0): ideas, plus an external Arena opponent run as a separate process.
  - canopy and ai-catan: no license, so ideas only.
  - MIT-licensed projects: may be reused with their notice kept.
- **Research with human participants:** The review user study (Phase 3) may require IRB approval or an exemption. [OPEN] I-7: confirm with the instructor.
- **Third-party LLM use:** Game chat sent to an LLM provider must not include personal data beyond what is needed. Costs stay within the training budget (I-4).
- **Trademark:** "Catan" is used only descriptively. The product name is Project Mayflower.

---

## Appendix A. Analysis Models

| Document Name | Description | Location |
| --- | --- | --- |
| System Overview Diagram | Major components and their interactions | `docs/DESIGN.md` §2 |
| Collector Class & Sequence Diagrams | Recording pipeline | `docs/DESIGN.md` §5.1 |
| Game Engine Class & Sequence Diagrams | State, board, moves, and views | `docs/DESIGN.md` §5.2 |
| Review Sequence Diagram | Grading a recorded game | `docs/DESIGN.md` §5.5 |
| Entity-Relationship Diagram | Recordings, games, moves, reviews, models | `docs/DESIGN.md` §6 |
| Vision Data Flow | Collector → Web → Review/Practice → Bots → Engine | `docs/VISION.md` §4 |
| Colonist Protocol Notes | Observed message types, coordinates, hidden information | `collector/PROTOCOL.md` |

[OPEN] `DESIGN.md` must be updated to match this spec (4-player only, multiple bots, player trading, accounts in Phase 2) before these diagrams are final.

## Appendix B. Issues List

| Issue ID | Issue Description | Status |
| --- | --- | --- |
| I-1 | Engine approach: clean-room engine (this spec) vs. modifying Catanatron (GPL-3.0) | Resolved in v0.5/0.6: clean-room; confirm at scrum |
| I-2 | Real-game data volume: target recorded games per phase | Open |
| I-3 | Review label names and thresholds | Open |
| I-4 | Table talk: templates, LLM (OpenRouter), or a mix; LLM cost | Open |
| I-5 | Phase 1 scope: which bots are due in Q1; whether the opening reviewer, UI, and job queue stay in Phase 1 | Open |
| I-6 | Engine speed: when to move hot paths to Rust/C/CUDA | Open: measure in Phase 1 |
| I-7 | IRB/consent for the review user study | Open: ask the instructor |
| I-8 | Fourth user class (Administrators / Maintainers) | Proposed |
| I-9 | Job queue library (RQ vs. Celery vs. in-process) | Open |
| I-10 | Collector verified in Google Chrome and Edge | Open |
| I-11 | Who signs "Approved By" in the version history | Open: ask the instructor |
| I-12 | Rule source of truth: rulebook vs. Colonist per-game settings; meaning of Colonist's `diceSetting` (possibly a "balanced dice" mode that is not standard 2d6) | Open |
| I-13 | Privacy: anonymize other players' usernames; keep chat; upload retention period | Open |
| I-14 | Team GPU inventory and university cluster access | Open |
| I-15 | Ownership of each component | Open |
| I-16 | Login method for Phase 2 accounts | Open |
| I-17 | Minimum games before showing habit diagnostics | Open |
| I-18 | Colonist ToS prohibits bots and scraping; the Collector may conflict with it (see 3.6). Real-player testing (FR-11) also needs review | Open: ask the instructor |
