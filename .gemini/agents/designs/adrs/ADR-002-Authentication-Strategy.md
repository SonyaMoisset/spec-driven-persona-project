# ADR-002: Authentication and Token Strategy

**Date:** 2025-10-03
**Status:** Accepted
**Deciders:** Avery (Solutions Architect)

## Context
We need to define the strategy for handling tokens for two separate concerns: (1) verifying a user's email address after registration, and (2) authenticating a user for accessing protected API endpoints after they have logged in. These have different security requirements and lifecycles.

## Decision
1.  **Email Verification:** We will use **single-use, securely generated, random string tokens** that are hashed before being stored in the database. A `verification_tokens` table will store the token hash, a foreign key to the `users` table, and an expiration timestamp.
2.  **API Authentication:** For authenticating users to access protected endpoints (post-login), we will use **JSON Web Tokens (JWTs)**. The login endpoint (to be designed later) will issue a short-lived JWT to the user upon successful authentication.

## Rationale

### For Email Verification Tokens:
- **Security:** Storing a hash of the token (`token_hash`) instead of the raw token follows the security best practice of never storing secrets in plaintext. If the database is compromised, the tokens cannot be easily used.
- **State Management:** Tying the token to a `user_id` and an `expires_at` timestamp in a dedicated table provides clear state management. It is easy to check if a token is valid, not expired, and belongs to the correct user. It also allows for a clean-up process for expired tokens.
- **Single-Use:** This approach makes it simple to invalidate a token after it has been used by deleting the record from the `verification_tokens` table, preventing replay attacks.

### For API Authentication Tokens (JWTs):
- **Statelessness:** JWTs are the industry standard for stateless API authentication. The token itself contains the necessary user information (e.g., user ID, roles), allowing the API to remain stateless. This is crucial for horizontal scalability, as any API instance can validate the token without needing to query a central session store.
- **Performance:** Because the token is self-contained, the API can avoid a database lookup on every request for user session information, improving performance.
- **Ecosystem:** There is mature library support for JWTs in Node.js (`jsonwebtoken`) and across all major languages and frameworks.

## Alternatives Considered

1.  **Using JWTs for Email Verification:**
    - **Pros:** We would only need one token mechanism in the application.
    - **Cons:** This is not what JWTs are designed for. JWTs are typically bearers of identity, not single-use action tokens. It would be complex to enforce single-use semantics for a JWT, as they are valid until they expire. It also unnecessarily exposes user information in the verification link.

2.  **Using Opaque Session Tokens (Session IDs) for API Authentication:**
    - **Pros:** This is a more traditional and very secure method. The token is just a random string, and the server stores all session information, which gives the server more control (e.g., to instantly revoke a session).
    - **Cons:** This approach makes the API stateful. It requires a shared session store (like Redis) that all API instances can access. This adds another piece of infrastructure to manage and can introduce a performance bottleneck, contradicting our goal of a simple, stateless, and easily scalable architecture.

## Consequences

**Positive:**
- We are using the right tool for each job, leading to a more secure and robust design.
- The architecture remains stateless and scalable.
- The verification process is secure and auditable.

**Negative:**
- We need to manage two different token mechanisms, which adds a small amount of complexity to the codebase.

## Security Implications
- **JWT Security:** The JWT signing secret must be a high-entropy string and managed securely (e.g., via AWS Secrets Manager). The `alg` header should be enforced as `HS256` or stronger, and the `none` algorithm must be rejected.
- **Verification Token Security:** The random string for verification must be generated from a cryptographically secure source. The expiration time must be reasonably short (e.g., 24 hours) to limit the window of opportunity for an attacker.
- **Token Transmission:** Both token types must only ever be transmitted over HTTPS.
