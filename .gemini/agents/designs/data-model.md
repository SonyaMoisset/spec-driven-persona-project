# Data Model: User Registration

This document defines the data model for the user registration service.

## 1. Tables / Collections

### `users`

This table stores user account information.

| Column | Type | Constraints | Description |
| :--- | :--- | :--- | :--- |
| `id` | `UUID` | `PRIMARY KEY` | Unique identifier for the user. |
| `email` | `TEXT` | `UNIQUE`, `NOT NULL` | User's email address. Case-insensitive unique index should be used. |
| `password_hash` | `TEXT` | `NOT NULL` | Salted hash of the user's password (e.g., using bcrypt). |
| `status` | `VARCHAR(20)` | `NOT NULL`, `DEFAULT 'pending'` | Account status. Can be `pending`, `active`, or `suspended`. |
| `created_at` | `TIMESTAMPTZ` | `NOT NULL`, `DEFAULT NOW()` | Timestamp when the user was created. |
| `updated_at` | `TIMESTAMPTZ` | `NOT NULL`, `DEFAULT NOW()` | Timestamp when the user was last updated. |

### `verification_tokens`

This table stores tokens used for email verification.

| Column | Type | Constraints | Description |
| :--- | :--- | :--- | :--- |
| `id` | `UUID` | `PRIMARY KEY` | Unique identifier for the token record. |
| `user_id` | `UUID` | `FOREIGN KEY (users.id)`, `ON DELETE CASCADE` | The user this token belongs to. |
| `token_hash` | `TEXT` | `UNIQUE`, `NOT NULL` | A cryptographic hash of the verification token. |
| `expires_at` | `TIMESTAMPTZ` | `NOT NULL` | The timestamp when this token expires. |
| `created_at` | `TIMESTAMPTZ` | `NOT NULL`, `DEFAULT NOW()` | Timestamp when the token was created. |

## 2. Indexes

- **`users(email)`**: A unique, case-insensitive index on the `email` column to ensure email uniqueness and fast lookups.
- **`verification_tokens(token_hash)`**: A unique index on the `token_hash` column for quick token validation.
- **`verification_tokens(user_id)`**: An index on the `user_id` column to quickly find tokens for a user.

## 3. Relationships

- A `user` can have multiple `verification_tokens` over time, but typically only one valid token at a time. (`one-to-many`)

## 4. Security & Data Handling

- **Encryption at Rest:** The database instance should have encryption at rest enabled to protect all data files.
- **Access Controls:** Database credentials must be strictly controlled. The API service should connect with a user that has the minimum required privileges (CRUD on its own tables).
- **PII (Personally Identifiable Information):** The `email` column contains PII and should be treated as sensitive.
- **Password Hashes:** The `password_hash` column is highly sensitive. Raw passwords are never stored.
- **Tokens:** Raw verification tokens are never stored, only their hashes.

## 5. Data Retention Policy

- **`users` table:** User records are kept indefinitely unless a user requests account deletion, in which case the record is anonymized or deleted in accordance with GDPR.
- **`verification_tokens` table:** Expired or used tokens should be periodically cleaned up. A background job should run daily to delete records where `expires_at` is in the past.
