# GitHub Webhook Dashboard API Documentation

## SQL Schema

Run this in your Supabase SQL Editor to create the table:

```sql
-- GitHub Webhook Events Table
CREATE TABLE github_webhook_events (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  event_type VARCHAR(100) NOT NULL,
  delivery_id VARCHAR(255) UNIQUE,
  action VARCHAR(100),
  repository_id BIGINT,
  repository_name VARCHAR(255),
  repository_full_name VARCHAR(255),
  repository_owner VARCHAR(255),
  repository_private BOOLEAN DEFAULT false,
  repository_url TEXT,
  sender_login VARCHAR(255),
  sender_id BIGINT,
  sender_avatar_url TEXT,
  hook_id BIGINT,
  payload JSONB NOT NULL,
  received_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_github_events_event_type ON github_webhook_events(event_type);
CREATE INDEX idx_github_events_repository ON github_webhook_events(repository_full_name);
CREATE INDEX idx_github_events_sender ON github_webhook_events(sender_login);
CREATE INDEX idx_github_events_received_at ON github_webhook_events(received_at DESC);
```

---

## API Endpoints

### 1. List Events (Paginated)

```
GET /api/webhook/github/events
```

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | number | 1 | Page number |
| `limit` | number | 20 | Items per page (max 100) |
| `event_type` | string | - | Filter by event type (push, ping, pull_request, etc.) |
| `repository` | string | - | Filter by repository name (partial match) |
| `sender` | string | - | Filter by sender username (partial match) |
| `start_date` | ISO date | - | Filter events after this date |
| `end_date` | ISO date | - | Filter events before this date |

**Response:**

```json
{
  "data": [
    {
      "id": "uuid",
      "event_type": "push",
      "action": null,
      "delivery_id": "abc-123",
      "repository_name": "dynamic-ui",
      "repository_full_name": "AchyuthMohan/dynamic-ui",
      "repository_owner": "AchyuthMohan",
      "sender_login": "AchyuthMohan",
      "sender_avatar_url": "https://avatars.githubusercontent.com/u/75477017?v=4",
      "hook_id": 669971994,
      "received_at": "2026-08-24T18:12:12Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 150,
    "totalPages": 8
  }
}
```

---

### 2. Get Single Event

```
GET /api/webhook/github/events/{id}
```

**Response:**

```json
{
  "data": {
    "id": "uuid",
    "event_type": "ping",
    "action": null,
    "delivery_id": "abc-123",
    "repository_name": "dynamic-ui",
    "repository_full_name": "AchyuthMohan/dynamic-ui",
    "repository_owner": "AchyuthMohan",
    "repository_private": false,
    "repository_url": "https://github.com/AchyuthMohan/dynamic-ui",
    "sender_login": "AchyuthMohan",
    "sender_id": 75477017,
    "sender_avatar_url": "https://avatars.githubusercontent.com/u/75477017?v=4",
    "hook_id": 669971994,
    "payload": { "/* full GitHub webhook payload */" },
    "received_at": "2026-08-24T18:12:12Z"
  }
}
```

---

### 3. Dashboard Stats

```
GET /api/webhook/github/stats
```

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `start_date` | ISO date | Filter stats from this date |
| `end_date` | ISO date | Filter stats until this date |

**Response:**

```json
{
  "totalEvents": 1250,
  "uniqueRepositories": 12,
  "uniqueSenders": 5,
  "eventsByType": {
    "push": 800,
    "pull_request": 250,
    "ping": 50,
    "issues": 150
  },
  "recentActivity": {
    "2026-08-24": 45,
    "2026-08-23": 32,
    "2026-08-22": 28,
    "2026-08-21": 15,
    "2026-08-20": 22,
    "2026-08-19": 18,
    "2026-08-18": 10
  }
}
```

---

### 4. Repositories Summary

```
GET /api/webhook/github/repositories
```

**Query Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `limit` | number | 10 | Max repositories to return (max 50) |

**Response:**

```json
{
  "data": [
    {
      "repository_id": 1208266425,
      "repository_name": "dynamic-ui",
      "repository_full_name": "AchyuthMohan/dynamic-ui",
      "repository_owner": "AchyuthMohan",
      "repository_url": "https://github.com/AchyuthMohan/dynamic-ui",
      "repository_private": false,
      "event_count": 156,
      "event_types": {
        "push": 120,
        "pull_request": 30,
        "ping": 6
      },
      "last_activity": "2026-08-24T18:12:12Z"
    }
  ],
  "total": 12
}
```

---

## Event Types Reference

| Event Type | Description |
|------------|-------------|
| `ping` | Webhook created/tested |
| `push` | Code pushed to repository |
| `pull_request` | PR opened/closed/merged |
| `issues` | Issue created/updated |
| `create` | Branch/tag created |
| `delete` | Branch/tag deleted |
| `release` | Release published |
| `fork` | Repository forked |
| `star` | Repository starred |

---

## Error Responses

All endpoints return errors in this format:

```json
{
  "error": "Error message describing what went wrong"
}
```

| Status Code | Description |
|-------------|-------------|
| 400 | Bad request / Invalid parameters |
| 404 | Resource not found |
| 500 | Server error |
