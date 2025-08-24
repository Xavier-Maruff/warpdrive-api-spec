# OpenAPI Spec for Warpdrive API

## Loose spec

All endpoints require either:

- A valid JWT token in the `Authorization` header as a Bearer token.
- An API key in the `X-API-Key` header.

### /token

- **POST**: Exchange an API key for a JWT token.

Response:

- `token`: JWT token for authentication

### `/sessions`

- **GET**: List all running sessions for the current user
  Request:
- `limit`: Maximum results to return (default: 100)
- `offset`: Results offset for pagination (default: 0)
- `statuses`: Comma-separated list of session statuses to filter by (default: all)

Response:

- Array of SessionDesc objects with fields: `session_id`, `alias`, `region`, `no_join`, `machine_id`, `image_id`, `image_name`, `started_at`, `status`

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

### `/sessions/{session_id}/guests`

- **GET**: List all guests in a session.
  Response:
- Array of GuestDesc objects with fields: `user_id`, `joined_at`

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

### `/user/guest_sessions`

- **GET**: List all sessions the current user has joined as a guest
  Request:
- `limit`: Maximum results to return (default: 100)
- `offset`: Results offset for pagination (default: 0)
  Response:
- Array of GuestSessionDesc objects with fields: `joined_at`, `session` (SessionDesc object)

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

### `/search/machines`

- **GET**: Search for available machine types

Request:

- `name`: Optional machine name filter
- `region`: Optional region filter
- `cloud_provider`: Optional cloud provider filter
- `vcpus`: Optional CPU count filter
- `memory_mib`: Optional memory size filter (in MiB)
- `gpu_count`: Optional GPU count filter
- `gpu_model`: Optional GPU model filter
- `limit`: Maximum results to return (default: 100)
- `offset`: Results offset for pagination (default: 0)

Response:

- `results`: Array of machine objects with fields: `id`, `name`, `description`, `href`, `cloud`, `not_in_regions`, `vcpus`, `memory_mib`, `gpu_count`, `gpu_model`

### `/search/images`

- **GET**: Search for available images

Request:

- `name`: Optional image name filter
- `cloud_provider`: Optional cloud provider filter
- `os`: Optional operating system filter
- `limit`: Maximum results to return (default: 100)
- `offset`: Results offset for pagination (default: 0)

Response:

- `results`: Array of image objects with fields: `id`, `name`, `description`, `os`, `href`, `cloud`

### `/search/machines/featured`

- **GET**: Get featured machine types

Response:

- Array of featured machine objects with fields: `id`, `name`, `description`, `href`, `cloud`, `not_in_regions`, `vcpus`, `memory_mib`, `gpu_count`, `gpu_model`

### `/search/images/featured`

- **GET**: Get featured images

Response:

- Array of featured image objects with fields: `id`, `name`, `description`, `os`, `href`, `cloud`
