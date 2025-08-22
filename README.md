# OpenAPI Spec for Warpdrive API

## Loose spec

All endpoints require either:

- A valid JWT token in the `Authorization` header as a Bearer token.
- An API key in the `X-API-Key` header.

### /token

- **POST**: Exchange an API key for a JWT token. Intended to be called initially to obtain a token for subsequent requests, as X-API-Key triggers a db lookup, which is expensive.

Response:

- `token`: JWT token for authentication

### `/sessions`

- **GET**: List all running sessions for the current user
- **POST**: Create a new session.

Request:

- `machine_id`: Machine config ID
- `region`: Region config ID
- `no_join`: Forbids other terminals from joining the session
- `pub_key`: Public key for SSH access

Response:

- `session_id`: Unique identifier for the session
- `alias`: Alias for the session
- `region`: Container hosting region
- `no_join`: Whether other terminals can join
- `machine_id`: Machine configuration ID
- `started_at`: Timestamp of session creation

### `/sessions/{session_id}/approve`

- **POST**: Approve a session for joining.

Request:

- `pub_key`: Public key to add to the session
- `user`: VM user to create
- `client_ip`: IP address of the client requesting approval
- `joiner_uuid`: UUID tied to the joiner SSE subject

### `/sessions/{session_id}/reject`

- **POST**: Reject a session for joining.

Request:

- `joiner_uuid`: UUID tied to the joiner SSE subject

### `/sse/session/{session_id}/owner`

- **GET**: Subscribe to session owner events

Response:
SSE events for the session owner, including:

- ```json{
    "event": "join_requested",
    "data": {
      "alias": "Session Alias",
      "subject_uuid": "Joiner SSE Subject UUID",
      "region": "Joiner Region",
      "pub_key": "Joiner Public Key",
      "user": "Joiner Proposed VM User",
      "client_ip": "Joiner Client IP",
    }
  }
  ```

### `/sse/session/alias/{alias}/guest`

- **POST**: Request to join a session as a guest

Request:

- `pub_key`: Public key to add to the session
- `user`: VM user to create

Response:
SSE events for the session guest, including:

- ```json{
    "event": "join_request_outcome",
    "data": {
      "status": "approved | denied",
      "reason": "user_already_in_session | owner_denied_join | undefined"
    }
  }
  ```
