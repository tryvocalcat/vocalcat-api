# MeetNearMe API Documentation

## Overview

The **VocalCat API** provides endpoints for authentication and social media integration. It allows clients to authenticate users, manage social accounts, generate OAuth URLs for social authentication, and create content. This API simplifies user management and social media operations for services integrated with MeetNearMe. 

## Base URL

The base URL for the API is:

```
https://beta.vocalcat.com/v1/meet
```

## Authentication

All API requests require authentication via an **access token**. This token must be included in the request headers or query parameters.

---

## Endpoints

### 1. Request Access Token

**POST** `/auth/token`

#### Request

To authenticate a client and generate an access token for the user, submit a `POST` request with the following JSON payload:

```json
{
  "clientId": "your-client-id",
  "userId": "user-id",
  "clientKey": "your-client-key"
}
```

- `clientId`: Your unique client identifier (obtained from the settings integration tab).
- `userId`: The unique identifier for the user within your system. It must be alphanumeric and can include hyphens, underscores, and the "@" symbol. The `userId` cannot exceed 125 characters and must not contain spaces. If the user does not exist, they will be created automatically.
- `clientKey`: Your secret client key (obtained from the settings integration tab).

#### Response

On success, the server will return the following JSON response:

```json
{
  "AccessCode": "encoded-access-token",
  "ExpiresAt": "2025-03-18T12:00:00Z"
}
```

- `AccessCode`: The access token for authenticating subsequent API requests.
- `ExpiresAt`: The expiration timestamp of the access token.

#### Description

This endpoint authenticates the client and generates an access token for the user. The token is valid until the `ExpiresAt` timestamp.

---

### 2. Retrieve Token (GET)

**GET** `/auth/token`

#### Request

This is an alternative method to retrieve an access token by passing parameters via the query string.

**Query Parameters:**

- `clientId`: Your client ID.
- `userId`: Your user ID.
- `clientKey`: Your client key.

#### Response

The response will be identical to the **POST** `/auth/token` endpoint.

#### Description

Use this method to retrieve an access token by passing the required parameters in the query string.

---

### 3. Get User's Linked Social Accounts

**GET** `/socials`

#### Headers

```http
X-ClientId: your-client-id
X-AccessCode: encoded-access-token
```

#### Response

On success, the server will return a JSON array containing the user's linked social accounts:

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

- `id`: The unique identifier for the social account.
- `platform`: The name of the social platform (e.g., "LinkedIn").
- `username`: The username of the user on the social platform.
- `linked`: A boolean indicating whether the account is successfully linked.

#### Description

This endpoint returns a list of social media accounts associated with the authenticated user.

---

### 4. Get Social OAuth URL

**GET** `/social/auth`

#### Query Parameters

- `socialName`: The name of the social network for which you want the OAuth URL. Supported platforms: "facebook", "linkedin". 

#### Headers

```http
X-ClientId: your-client-id
X-AccessCode: encoded-access-token
```

#### Response

On success, the server will return a JSON object containing the OAuth URL:

```json
{
  "OAuthUrl": "https://www.linkedin.com/oauth/authorize?client_id=..."
}
```

- `OAuthUrl`: The URL for the OAuth authorization page of the specified social network.

#### Description

This endpoint generates and returns the OAuth authorization URL for the specified social network. You can use this URL to authenticate users via OAuth.

---

### 5. Create Social Media Content

**POST** `/`

#### Headers

```http
X-ClientId: your-client-id
X-AccessCode: encoded-access-token
```

#### Request

To create a new post for social media, send a `POST` request with the following JSON body:

```json
{
  "contentBody": "This is a test post.",
  "contentTitle": "Test Title",
  "targets": [
    {
      "socialAccountId": 123,
      "body": "This is the content to be published independently of what the main content has.",
      "autoSchedule": true,
      "scheduleTime": "2025-03-20T10:00:00Z"
    }
  ]
}
```

- `contentBody`: The body of the content to post.
- `contentTitle`: The title of the content.
- `targets`: A list of target social media accounts and their posting details.
  - `socialAccountId`: The ID of the social account to target.
  - `targetMethod`: The method used to publish the content. Supported methods: `copy`, `schedule`.
  - `body`: The content body for the post.
  - `autoSchedule`: A boolean flag indicating whether to automatically schedule the post.
  - `scheduleTime`: The scheduled time for posting in ISO 8601 format (`YYYY-MM-DDTHH:mm:ssZ`).

#### Response

On success, the server will return the following response:

```json
{
  "contentId": 456,
  "status": "scheduled"
}
```

- `contentId`: The unique identifier for the created content.
- `status`: The current status of the content. Possible values: `scheduled`, `posted`, `failed`.

#### Description

This endpoint allows clients to create content and schedule it for publication on linked social media accounts. The content can be posted immediately or scheduled for a future time.

This endpoint provides flexibility for creating social media content. You can include additional details like `title` and `imageUrl`, set up custom targets, schedule posts, or publish them immediately by using the current time as `scheduleTime`. The auto-scheduling feature simplifies the process when `schedulePost` is `true` and `scheduleTime` is omitted. Targets can be tailored using publishing methods like `copy`, `optimize`, and `override`.

#### Request Examples

1. **Basic Example with Minimum Information and Two Targets**

```json
{
  "body": "A quick update for my followers!",
  "targets": [
    {
      "socialAccountId": 101,
      "targetMethod": "copy",
      "schedulePost": false
    },
    {
      "socialAccountId": 102,
      "targetMethod": "copy",
      "schedulePost": false
    }
  ]
}
```

This example includes only the body and targets. Each target will use the same content from the body. It does not specify a title or image, and the posts will not be scheduled.

---

2. **Example Without Schedule Time**

```json
{
  "body": "Don't forget to check out our latest offers!",
  "title": "Latest Offers",
  "targets": [
    {
      "socialAccountId": 201,
      "targetMethod": "optimize",
      "schedulePost": true
    }
  ]
}
```

Here, the `scheduleTime` is omitted, and since `schedulePost` is set to `true`, the system will use the auto-schedule setting. The system will use AI to optimize the content to the right platform.

---

3. **Example for Immediate Publishing**

```json
{
  "body": "This is an urgent update, going live now!",
  "title": "Urgent Update",
  "targets": [
    {
      "socialAccountId": 301,
      "targetMethod": "copy",
      "schedulePost": true,
      "scheduleTime": "2025-04-02T17:09:00Z"
    }
  ]
}
```

In this example, `scheduleTime` is set to the current timestamp (`2025-04-02T17:09:00Z`), ensuring the post is published immediately.

---

### 6. Get User Content

**GET** `/content`

#### Headers

```http
X-ClientId: your-client-id
X-AccessCode: encoded-access-token
```

#### Response

On success, the server will return a JSON array containing the user's content:

```json
[
  {
    "contentId": 456,
    "contentBody": "This is a test post.",
    "contentTitle": "Test Title",
    "status": "scheduled",
    "createdAt": "2025-03-18T12:00:00Z"
  }
]
```

- `contentId`: The unique identifier for the content.
- `contentBody`: The body of the content.
- `contentTitle`: The title of the content.
- `status`: The current status of the content. Possible values: `scheduled`, `posted`, `failed`.
- `createdAt`: The timestamp when the content was created.

#### Description

This endpoint retrieves the list of content created by the user, along with its details and current status. Proper authentication headers are required to access this data.

---

### 7. Get Content by ID

**GET** `/content/{contentId}`

#### Parameters

- `contentId`: The unique identifier for the content. This is passed as a path parameter.

#### Headers

```http
X-ClientId: your-client-id
X-AccessCode: encoded-access-token
```

#### Response

On success, the server will return a JSON object containing the details of the specified content:

```json
{
  "contentId": 456,
  "contentBody": "This is a test post.",
  "contentTitle": "Test Title",
  "status": "scheduled",
  "createdAt": "2025-03-18T12:00:00Z"
}
```

- `contentId`: The unique identifier for the content.
- `contentBody`: The body of the content.
- `contentTitle`: The title of the content.
- `status`: The current status of the content. Possible values: `scheduled`, `posted`, `failed`.
- `createdAt`: The timestamp when the content was created.

#### Description

This endpoint retrieves the details of a specific content item based on its unique identifier. Proper authentication headers are required to access this data.

---

## Authentication & Security

- **API keys and access tokens** are required for all endpoints. The access token is obtained through the `/auth/token` endpoint.
- Requests without valid credentials will return a **401 Unauthorized** error.

---

## Example Use Case

Here’s an example of using the API to create and schedule a social media post:

1. **Authenticate the user** by making a `POST` request to `/auth/token` with your client ID, user ID, and client key.
2. **Retrieve the OAuth URL** for LinkedIn by calling the `/social/auth` endpoint with `socialName=facebook`. Use the url to add a new social account.
3. **Get the user's social accounts** by sending a `GET` request to `/socials` with the appropriate headers (`X-ClientId` and `X-AccessCode`).
4. **Create and schedule content** by calling the `/` endpoint with the content details, social account ID, and scheduling information.
