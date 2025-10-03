# ADR-001: Core Technology Stack Selection

**Date:** 2025-10-03
**Status:** Accepted
**Deciders:** Avery (Solutions Architect)

## Context
We need to select the primary technologies for the backend of the new user registration service. The choice will influence development speed, performance, scalability, and security. The key components to decide on are the programming language/framework for the API and the database system.

## Decision
We will use the following technology stack:
- **API Framework:** **Node.js with Express.js**
- **Database:** **PostgreSQL**

## Rationale
This combination was chosen for several reasons:

1.  **Performance:** Node.js's non-blocking, event-driven architecture is highly efficient for I/O-bound operations like handling API requests and database queries, which will be the primary workload of this service.
2.  **Ecosystem:** The JavaScript/Node.js ecosystem (NPM) is vast and mature, providing battle-tested libraries for everything from password hashing (`bcrypt`) and data validation (`express-validator`) to database access (`pg`). This accelerates development and improves security by relying on well-maintained packages.
3.  **Data Integrity:** As user data is inherently relational (users have emails, passwords, statuses), a relational database like PostgreSQL is a natural fit. It allows us to enforce strong data integrity constraints, such as ensuring every user has a unique email address at the database level.
4.  **Scalability:** Both Node.js (when run in a stateless manner) and PostgreSQL are highly scalable. A stateless Node.js API can be scaled horizontally with ease, and PostgreSQL has a proven track record of scaling to handle very large workloads.
5.  **Developer Pool:** JavaScript and Node.js are extremely popular, making it easier to find developers who are already familiar with the technology.

## Alternatives Considered

1.  **Python (Django/FastAPI) with PostgreSQL:**
    - **Pros:** Python is also a very popular language with excellent libraries. FastAPI offers performance comparable to Node.js. Django provides a more batteries-included framework which can speed up initial development.
    - **Cons:** Django can be more monolithic and opinionated, which might be overkill for a simple microservice. While FastAPI is excellent, the Node.js ecosystem for web API development is arguably more extensive.

2.  **Node.js with MongoDB (NoSQL):**
    - **Pros:** MongoDB offers flexible schema design and can be very fast for simple reads/writes.
    - **Cons:** The data for a user system is highly structured and relational. Enforcing a unique constraint on the email field is critical, which is a core strength of relational databases. Using a NoSQL database here would require implementing that logic at the application level, adding complexity and potential for race conditions. The benefits of a flexible schema do not outweigh the need for strong data integrity in this case.

## Consequences

**Positive:**
- We can build a fast, scalable, and secure API quickly.
- The architecture is simple and easy to understand.
- We are using a widely-supported and well-documented set of technologies.

**Negative:**
- As a dynamically typed language, JavaScript can be prone to certain types of runtime errors. This will be mitigated by using a validation library for all inputs and potentially introducing TypeScript later if the project grows in complexity.

## Security Implications
- The chosen stack has well-understood security best practices.
- We must be diligent in selecting and managing third-party NPM packages to avoid supply chain attacks.
- Secure configuration of the Node.js server (e.g., disabling the `x-powered-by` header) and PostgreSQL (e.g., network access controls) is critical.
