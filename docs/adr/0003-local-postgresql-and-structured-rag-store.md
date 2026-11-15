### ADR-0003: Persist patient schedules in local PostgreSQL and store public leaflets as structured local JSON

- Status: accepted
- Date: 2026-11-15
- Deciders: ME, VV
- Related: FR-04, FR-18, NFR-01, NFR-04, NFR-06, NFR-08, DEC-3, DEC-4, R-02, R-05

#### Context and problem
MeowMed must store fictional patient medication schedules, reminder settings, and local activity logs while supporting offline fallback access (FR-18). At the same time, the RAG component needs fast, deterministic retrieval of official leaflet sections (indications, dosage, contraindications, interactions) to cite sources accurately (NFR-06). How should application data and leaflet text be stored?

#### Decision drivers
- FR-18 & NFR-01: Fast (<2s) local access to saved schedules even when external AI APIs are offline
- NFR-06 & DEC-4: Exact section citations from leaflets and strict "Leaflet unavailable" refusal for unknown drugs
- DEC-3 & NFR-04: Zero external cloud storage of patient data; local reproducibility across OS platforms

#### Considered options
1. Cloud-hosted database and live web scraping of drug leaflets on every user query
2. Local SQLite file for everything, with raw unparsed PDF leaflets
3. Local PostgreSQL database (accessed via `patient-db` MCP server) for schedules/logs, paired with pre-parsed, structured JSON leaflet files (accessed via `leaflet-RAG` MCP server)

#### Decision
Chosen option 3, because PostgreSQL provides reliable relational persistence for schedules and logs (fulfilling Section 6.3 of the specification), while structured JSON leaflets eliminate PDF parsing errors at runtime and enable deterministic interaction checks and exact section citations.

#### Consequences
- Good: Deterministic interaction blocking (FR-03), instant offline schedule viewing (FR-18), and clean separation of concerns via two dedicated MCP servers.
- Bad: Requires running a local PostgreSQL instance and manually curating/formatting the demo leaflet JSON files (`DOC-1`, `DOC-2`, `DOC-3`).

#### Confirmation
Verified that schedules load within 2 seconds with network disabled (NFR-01, FR-18) and that every RAG response cites a valid JSON leaflet section (NFR-06).
