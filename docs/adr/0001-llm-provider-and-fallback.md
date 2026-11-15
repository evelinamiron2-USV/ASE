### ADR-0001: Use Gemini Flash as primary LLM and Mistral as fallback behind an adapter

- Status: accepted
- Date: 2026-11-15
- Deciders: ME, VV
- Related: NFR-02, DEC-1, R-01, GR-02, GR-03

#### Context and problem
MeowMed relies on an LLM to generate daily medication schedule drafts and simplify official drug package leaflets. Because this is a student teaching prototype, we are restricted to free-tier APIs, which enforce strict per-minute and per-day rate limits (HTTP 429). How do we keep the application responsive and available when the primary provider is throttled or temporarily down?

#### Decision drivers
- NFR-02: Graceful degradation and availability during AI service interruptions
- DEC-1: Zero budget / free-tier LLM providers only
- NFR-07: Maintainable, modular backend architecture

#### Considered options
1. Use Gemini API directly inside each agent without a fallback
2. Use Gemini Flash as primary provider and Mistral Small as fallback behind a unified LLM Adapter (with retry/backoff and circuit breaker)
3. Host a local open-weight LLM on student laptops

#### Decision
Chosen option 2, because it satisfies NFR-02 within the free-tier constraint without requiring high-end GPU hardware on student machines. The adapter isolates all A2A agents from provider-specific REST schemas.

#### Consequences
- Good: Centralized retry, timeout, and circuit-breaker logic; switching models requires only a configuration change.
- Bad: Two API keys to configure and test; slightly different phrasing between Gemini and Mistral outputs requires flexible integration tests.

#### Confirmation
Verified via automated smoke tests and simulated HTTP 429 failures (`FAKE_LLM_FAILURES=2` and `99`) in the MCP Inspector and Lab 6 test suite.
