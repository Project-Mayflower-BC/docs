# Catan Bot Design Document

## *Team name: Mayflower*

## *Team members:*

## *Noah Levine, Caleb Weisgerber, Ryan Massenburg, Emmett Oliver*

## 

## 1\. Introduction

*\[Provide an overview of the design document, including its purpose, scope, and intended audience.\]*

## 2\. System Overview (Architectural Design)

*\[Describe the high-level architecture of the system, including its major components, layers, and interactions. Include a system architecture diagram to illustrate the overall structure.\]*

## 3\. Stakeholders (Architectural Design)

*\[List the stakeholders involved in the project and their respective concerns or requirements.\]*

## 4\. Quality Attributes (Architectural Design)

*\[Identify and describe the quality attributes addressed by the architecture, such as performance, scalability, security, and maintainability.\]*

## 5\. Detailed Component Designs (Detailed Design)

### 5.1 \[The Collector (Client-side recording extension)\]

#### *5.1.1 Overview*

- **Purpose:** A read-only Chromium browser extension that passively captures live Settlers of Catan matches played on Colonist.io.

- **Responsibilities:**
    - Attaches to the active Colonist tab via the Chrome Debugger API upon user initiation.
    - Passively monitors incoming/outgoing WebSocket frames and HTTP payloads representing game topology, dice events, building placements, public chat, and player trade transactions.
    - Sanitizes and strips personal credentials, auth tokens, and sensitive query strings.
    - Detects stream breaks, flagging gap markers when packet loss or browser disconnects occur.
    - Serializes and exports the validated session as a JSON to the user's local disk.

- **Interfaces:**
    - *Upstream:* Chrome Debugger / Network domain protocol events.
    - *Downstream:* Browser local download manager / Web App File Upload (UI-02).

#### *5.1.2 Design Considerations*
- **Zero-Interference Mandate:** Operates strictly in passive capture mode; never injects scripts into game memory, modifies DOM buttons, or interacts with Colonist servers.
- **Reconnection Handling:** If the tab crashes or disconnects mid-match, the collector creates an explicit gap marker. Upon reconnect, it links new events to the parent session to preserve data continuity.

#### *5.1.3 Class Diagram*

*\[Include a class diagram to illustrate the internal structure of the component, including classes, attributes, methods, and relationships.\]*

#### *5.1.4 Sequence Diagrams*

*\[Include sequence diagrams to depict the interactions between the component and other system components or external entities.\]*


### 5.2 \[Game Engine\]

#### *5.2.1 Overview*

- **Purpose:** A custom, deterministic headless rules engine implementing official base 4-player Settlers of Catan mechanics without dependencies on third-party game libraries.

- **Responsibilities:**
    - Maintains the canonical authoritative board graph (19 hexes, 54 vertices, 72 edges, harbors, and robber positions).
    - Generates legal candidate actions for any game state (settlement/city placement distance rules, connected road chains, maritime trades, dev card play limits).
    - Executes actions deterministically given explicit random seeds or historic dice rolls.
    - Derives player-specific Hidden-Information Views (FR-05) by stripping opponent hand contents and unplayed development cards.

- **Interfaces:**
    - *Input:* `apply_action(state, action)`, `replay_session(json_payload)`.
    - *Output:* `get_legal_actions(state) -> List[Action]`, `get_player_view(state, seat_id) -> PlayerView`.

#### *5.2.2 Design Considerations*
- **Speed & State Copying:** Rollouts require executing hundreds of simulated games per second; state representations prioritize shallow-copy efficiency or bitboard representations to eliminate object creation overhead.
- **Historic Replay vs. Simulation:** Replaying uploaded Colonist games uses the recorded dice rolls directly, whereas Monte Carlo rollouts simulate dice using pseudo-random generators.

#### *5.2.3 Class Diagram*

*\[Include a class diagram to illustrate the internal structure of the component, including classes, attributes, methods, and relationships.\]*

#### *5.2.4 Sequence Diagrams*

*\[Include sequence diagrams to depict the interactions between the component and other system components or external entities.\]*


### 5.3 \[Web Client & Visualization Platform\]

#### *5.3.1 Overview*
- **Purpose:** A responsive single-page web application providing file upload handling, visual 2D board playback, evaluation graphs, and practice sparring controls.

- **Responsibilities:**
    - Provides file ingestion interfaces (UI-02) with client-side schema pre-validation.
    - Renders an interactive SVG/HTML5 Canvas board displaying hex terrains, dice chits, settlements, roads, cities, and the robber.
    - Visualizes review analytics: win-probability line charts, move grading badges (Best, Mistake, Blunder), resource starvation alerts, and trade utility explanations (UI-04, UI-05).
    - Houses the Practice Table (UI-06) for live sparring matches against bot opponents.

- **Interfaces:**
    - REST API client communicating with backend endpoints (`/api/v1/sessions`, `/api/v1/reviews`).
    - WebSocket client receiving real-time bot moves during practice sparring.

#### *5.3.2 Design Considerations*
- **No-Freeze Rendering:** Playback scrubbing and timeline navigation must execute entirely on the client without firing network requests once the review payload is loaded
- **Asynchronous Job Feedback:** Polling / WebSocket event handlers keep the UI updated with background simulation progress.

#### *5.3.3 Class Diagram*

*\[Include a class diagram to illustrate the internal structure of the component, including classes, attributes, methods, and relationships.\]*

#### *5.3.4 Sequence Diagrams*

*\[Include sequence diagrams to depict the interactions between the component and other system components or external entities.\]*


### 5.4 \[Asynchronous Job Dispatch & Worker Queue\]

#### *5.4.1 Overview*
- **Purpose:** A distributed background compute pipeline that processes resource-intensive simulation tasks without blocking the web server.

- **Responsibilities:**
    - Accepts review evaluation requests from the API layer and pushes standardized job tickets to a message broker.
    - Coordinates worker nodes running rollout tasks across multiple CPU cores.
    - Persists completed evaluation summaries to the database and signals completion to the client.

- **Interfaces:**
    - *Broker Interface:* Redis queue / Celery task runner.
    - *Persistence Interface:* PostgreSQL database connection for writing completed review summaries.

#### *5.4.2 Design Considerations*
- **Budget & Local-First:** Must execute cleanly on local developer machines via Docker Compose at no cloud cost during early phases, scaling to distributed cloud workers in Phase 3.
- **Task Timeout Protection:** Strict timeout thresholds (e.g. ≤ 30s for opening review, ≤ 10m for full match review) prevent rogue simulations from starving the worker pool.

#### *5.4.3 Class Diagram*

*\[Include a class diagram to illustrate the internal structure of the component, including classes, attributes, methods, and relationships.\]*

#### *5.4.4 Sequence Diagrams*

*\[Include sequence diagrams to depict the interactions between the component and other system components or external entities.\]*


### 5.5 \[Review & Sparring Engine\]

#### *5.5.1 Overview*
- **Purpose:** A strategic intelligence layer responsible for generating move assessments, ranking setup choices, and executing automated bot decisions.

- **Responsibilities:**
    - Enforces the common bot execution interface: `choose_move(view, time_limit)` and `win_probability(view)` across all bot architectures.
    - Computes Expected Value shifts ($\Delta\text{EV}$) to identify blunders and generate plain-language advice.
    - Executes opening placement audits (FR-07) assessing total production pips, port synergies, and resource starvation risks.
    - Hosts the standardized agent pool including the random baseline and candidate bot models (AlphaBeta, MCTS, AlphaZero-style, and PPO).
    - Powers rollout simulations in Phase 1 and the 4-player Practice Sparring table in Phase 2.

- **Interfaces:**
    - *Input:* Filtered `PlayerView` instances from the Engine.
    - *Output:* Scored move candidates, win probability curves, and qualitative blunder tags.

#### *5.5.2 Design Considerations*
- **Imperfect Information Compliance:** Bot decision logic can only access sanitized `PlayerView` data; omniscient peeking at unrevealed opponent cards is blocked at the interface level.
- **Variance Isolation ("About Equal" Threshold):** Evaluator applies statistical confidence intervals so moves with minor EV differences within dice roll variance are not falsely identified as blunders.

#### *5.5.3 Class Diagram*

*\[Include a class diagram to illustrate the internal structure of the component, including classes, attributes, methods, and relationships.\]*

#### *5.5.4 Sequence Diagrams*

*\[Include sequence diagrams to depict the interactions between the component and other system components or external entities.\]*


### 5.x \[Component Name x\]

#### *5.x.1 Overview*

*\[Provide an overview of the component, including its purpose, responsibilities, and interfaces.\]*

#### *5.x.2 Design Considerations*

*\[Discuss the design considerations and constraints for the component.\]*

#### *5.x.3 Class Diagram*

*\[Include a class diagram to illustrate the internal structure of the component, including classes, attributes, methods, and relationships.\]*

#### *5.x.4 Sequence Diagrams*

*\[Include sequence diagrams to depict the interactions between the component and other system components or external entities.\]*


## 6\. Data Model (Detailed Design)

*\[Describe the data model used by the system, including entities, attributes, relationships, and constraints. Include entity-relationship diagrams (ERDs) to visualize the data model.\]*

## 7\. User Interface Design (Detailed Design)

*\[Describe the user interface design for the system, including wireframes, mockups, or prototypes. Discuss usability considerations and design principles applied.\]*

## 8\. Error Handling and Exception Management (Detailed Design)

*\[Describe how errors and exceptions are handled within the system. Include error handling strategies, exception classes, and error messages.\]*

## 9\. Security Design (Detailed Design)

*\[Discuss the security mechanisms and measures implemented in the system to protect against unauthorized access, data breaches, and other security threats. Include authentication, authorization, encryption, and auditing mechanisms.\]*

## 10\. Performance Optimization (Detailed Design)

*\[Describe the strategies and techniques used to optimize system performance, including caching, indexing, query optimization, and resource management.\]*

## 11\. Deployment Architecture (Detailed Design)

*\[Describe the deployment architecture of the system, including hardware infrastructure, network topology, and deployment configurations. Include deployment diagrams to visualize the deployment architecture.\]*

## 12\. Integration and Interfaces (Detailed Design)

*\[Describe how the system integrates with external systems or services. Discuss communication protocols, APIs, and data formats used for integration.\]*

## 13\. Testing Strategy (Detailed Design)

*\[Describe the testing strategy for the system, including unit testing, integration testing, system testing, and acceptance testing. Discuss testing tools, frameworks, and methodologies employed.\]*

## 14\. Maintenance and Support (Detailed Design)

*\[Discuss the maintenance and support considerations for the system, including version control, documentation, bug tracking, and release management processes.\]*

## 15\. Glossary

*\[Provide a glossary of terms and acronyms used throughout the document for clarity and consistency.\]*

## 16\. References

*\[Include any references or resources consulted during the design process.\]*

