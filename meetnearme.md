# MeetNearMe API Documentation

## Overview
The `MeetNearMe` provides authentication and social integration endpoints for the **MeetNearMe** service. This API allows clients to authenticate users, retrieve social accounts, generate OAuth URLs for social authentication, and create content.

## Base URL
```
https://beta.vocalcat.com/v1/meet
```

## Endpoints

### 1. Request Access Token

**POST** `/auth/token`

#### Request
```json
{
  "clientId": "your-client-id",
  "userId": "user-id",
  "clientKey": "your-client-key"
}
```

clientId and clientKey comes from the settings integration tab
userId is a identifier from your system for a user, only accepts alphanumeric, - _  @, no more than 125 characters, and no spaces. if the user does not exists it gets created.

#### Response
```json
{
  "AccessCode": "encoded-access-token",
  "ExpiresAt": "2025-03-18T12:00:00Z"
}
```

#### Description

Authenticates a client and generates an access code for the user.

### 2. Retrieve Token (GET)
**GET** `/auth/token`

#### Request
Query Parameters:
- `clientId`: Client ID
- `userId`: User ID
- `clientKey`: Client Key

#### Response
Same as **POST** `/auth/token`

#### Description
Alternative method to request an access token via query parameters.

---

### 3. Get User's Social Accounts
**GET** `/socials`

#### Headers
```http
X-ClientId: your-client-id
X-AccessCode: encoded-access-token
```

#### Response
```json
[
  {
    "id": 123,
    "platform": "LinkedIn",
    "username": "user123",
    "linked": true
  }
]
```

#### Description
Returns a list of social accounts linked to the authenticated user.

---

### 4. Get Social OAuth URL
**GET** `/social/auth`

#### Query Parameters
- `socialName`: Name of the social network (e.g., "linkedin")

#### Headers
```http
X-ClientId: your-client-id
X-AccessCode: encoded-access-token
```

#### Response
```json
{
  "OAuthUrl": "https://www.linkedin.com/oauth/authorize?client_id=..."
}
```

#### Description
Returns the OAuth authorization URL for the specified social network.

---

### 5. Create Social Media Content
**POST** `/`

#### Headers
```http
X-ClientId: your-client-id
X-AccessCode: encoded-access-token
```

#### Request
```json
{
  "contentBody": "This is a test post.",
  "contentTitle": "Test Title",
  "targets": [
    {
      "socialAccountId": 123,
      "targetMethod": "copy",
      "body": "This is a copied post.",
      "autoSchedule": true,
      "scheduleTime": "2025-03-20T10:00:00Z"
    }
  ]
}
```

#### Response
```json
{
  "contentId": 456,
  "status": "scheduled"
}
```

#### Description
Creates content for social media accounts with optional scheduling.

---

## Authentication & Security
- **API keys and access codes** are required for all endpoints.
- Requests without valid credentials will return **401 Unauthorized**.

## Error Handling
Errors are returned in the following format:
```json
{
  "error": "Invalid credentials",
  "code": 401
}
```
