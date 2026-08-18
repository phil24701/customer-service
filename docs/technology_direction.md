**Customer Support Platform**

*Portfolio Technology Direction & Constraints v1.0*

*GitHub Portfolio Project • August 12, 2026*

# 1. Purpose

Establishes the technology direction that should guide technical design
and Spec-Kitty. It does not pre-select every library or implementation
detail.

# 2. Technology Direction

  -----------------------------------------------------------------------
  Area                    Direction               Portfolio Intent
  ----------------------- ----------------------- -----------------------
  Frontend                React + TypeScript      Primary portfolio
                                                  requirement.

  Backend                 Node.js + TypeScript    Preferred backend
                                                  direction.

  API                     REST                    Primary application API
                                                  style.

  Database                PostgreSQL              System of record.

  Caching / transient     Redis                   Use where technically
  state                                           justified.

  Search                  PostgreSQL full-text    Technical design should
                          search and/or dedicated determine the
                          search                  appropriate level.

  Asynchronous processing Background jobs         Mechanism to be
                                                  selected during
                                                  technical design.

  Real-time               WebSockets or           Choose based on actual
                          Server-Sent Events      use cases.

  Authentication /        Application             Exact mechanism to be
  authorization           authentication with     selected.
                          RBAC                    

  Testing                 Unit + integration +    Frameworks to be
                          end-to-end              selected.

  Containers              Docker                  Primary
                                                  packaging/deployment
                                                  approach.

  CI/CD                   Automated               Platform details to be
                          repository-based CI/CD  selected.

  Observability           Structured logging plus Tooling to be selected.
                          practical               
                          health/diagnostic       
                          monitoring              

  External integration    Email integration       Demonstrate
                                                  external-service
                                                  integration.

  AI                      Bounded AI-assisted     Classification,
                          support features        summaries, suggested
                                                  responses, etc.; must
                                                  provide business value.
  -----------------------------------------------------------------------

# 3. Constraints

-   Do not replace specified primary technologies merely because another
    stack is marginally easier.

-   Do not add infrastructure solely to increase the technology count.

-   Every significant additional technology should have a clear
    architectural or portfolio justification.

-   Prefer actively maintained, documented, practical technologies.

-   Keep the architecture sophisticated enough to demonstrate
    engineering skill without creating enterprise-scale operational
    overhead.

# 4. Decisions for Technical Design

-   Backend framework and project structure within Node.js/TypeScript.

-   API validation, documentation, and error handling.

-   Authentication/session or token strategy.

-   ORM/query/data-access strategy for PostgreSQL.

-   Redis use cases.

-   Background-job mechanism.

-   WebSockets versus SSE.

-   Search implementation and indexing.

-   Attachment storage.

-   Email integration.

-   AI integration boundary and provider abstraction.

-   Testing framework and test architecture.

-   Container and CI/CD architecture.

-   Logging, health checks, metrics, and observability.

# 5. Architecture Decision Rule

Where this document names a broad technology but not a specific
implementation, Spec-Kitty may propose alternatives and tradeoffs.
Material final decisions should be approved during technical design and
recorded in an architecture decision record.
