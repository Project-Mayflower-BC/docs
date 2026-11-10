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

- *Purpose:* A read-only Chromium browser extension that passively captures live Settlers of Catan matches played on Colonist.io.

- *Responsibilities:*
    - Attaches to the active Colonist tab via the Chrome Debugger API upon user initiation.
    - Passively monitors incoming/outgoing WebSocket frames and HTTP payloads representing game topology, dice events, building placements, public chat, and player trade transactions.
    - Sanitizes and strips personal credentials, auth tokens, and sensitive query strings.
    - Detects stream breaks, flagging gap markers when packet loss or browser disconnects occur.
    - Serializes and exports the validated session as a JSON to the user's local disk.

- *Interfaces:*
    - *Upstream:* Chrome Debugger / Network domain protocol events.
    - *Downstream:* Browser local download manager / Web App File Upload (UI-02).

#### *5.1.2 Design Considerations*
- *Zero-Interference Mandate:* Operates strictly in passive capture mode; never injects scripts into game memory, modifies DOM buttons, or interacts with Colonist servers.
- *Reconnection Handling:* If the tab crashes or disconnects mid-match, the collector creates an explicit gap marker. Upon reconnect, it links new events to the parent session to preserve data continuity.

#### *5.1.3 Class Diagram*

*\[Include a class diagram to illustrate the internal structure of the component, including classes, attributes, methods, and relationships.\]*

#### *5.1.4 Sequence Diagrams*

*\[Include sequence diagrams to depict the interactions between the component and other system components or external entities.\]*


### 5.1 \[Game Engine\]


### 5.3 \[Web Client & Visualization Platform\]


### 5.4 \[Background Simulation & Worker Pipeline\]


### 5.5 \[Review & Sparring Engine\]



### 5.x \[Component Name x\]

#### *5.x.1 Overview*

*\[Provide an overview of the component, including its purpose, responsibilities, and interfaces.\]*

#### *5.x.2 Design Considerations*

*\[Discuss the design considerations and constraints for the component.\]*

#### *5.x.3 Class Diagram*

*\[Include a class diagram to illustrate the internal structure of the component, including classes, attributes, methods, and relationships.\]*

#### *5.x.4 Sequence Diagrams*

*\[Include sequence diagrams to depict the interactions between the component and other system components or external entities.\]*

### 5.x \[Component Name x\]

*\[(Repeat the same structure for each component)\]*

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

