# LinkedIn ChatGPT Action Setup

This guide connects the LinkedIn Developer app **Kim Career Automation** to a Custom GPT / ChatGPT Action.

## 1. LinkedIn app values

Use these values in the GPT Action OAuth setup:

```text
Client ID: 77ovquwfa9esh0
Client Secret: keep private; do not commit to GitHub or paste into chat
Authorization URL: https://www.linkedin.com/oauth/v2/authorization
Token URL: https://www.linkedin.com/oauth/v2/accessToken
Scopes: openid profile email w_member_social
```

If LinkedIn rejects a scope during login, remove the rejected scope and retry with the scopes visible in the LinkedIn Developer Portal under **Auth → OAuth 2.0 scopes**.

## 2. OpenAPI schema

Use this file as the GPT Action schema:

```text
linkedin-chatgpt-action.openapi.yaml
```

Raw URL:

```text
https://raw.githubusercontent.com/OgeonX-Ai/OgeonX-Ai/main/linkedin-chatgpt-action.openapi.yaml
```

## 3. GPT / ChatGPT Action OAuth fields

In the GPT editor / action authentication settings, select **OAuth**.

Fill:

```text
Client ID: 77ovquwfa9esh0
Client Secret: paste your current LinkedIn primary client secret here only in the GPT editor secret field
Authorization URL: https://www.linkedin.com/oauth/v2/authorization
Token URL: https://www.linkedin.com/oauth/v2/accessToken
Scope: openid profile email w_member_social
```

Do not place the client secret in the OpenAPI schema.

## 4. Redirect URL step

After saving the GPT/action, ChatGPT will use a callback URL like:

```text
https://chat.openai.com/aip/{g-YOUR-GPT-ID}/oauth/callback
```

or:

```text
https://chatgpt.com/aip/{g-YOUR-GPT-ID}/oauth/callback
```

Copy the exact callback URL from the GPT editor or OAuth error message and add it to LinkedIn Developer Portal:

```text
LinkedIn Developer Portal → Your app → Auth → Authorized redirect URLs for your app
```

Keep the existing Postman callback too if you still want to test manually:

```text
https://oauth.pstmn.io/v1/callback
```

## 5. First test

Ask the GPT/action:

```text
Get my LinkedIn user info.
```

This should call:

```http
GET https://api.linkedin.com/v2/userinfo
Authorization: Bearer <access_token>
```

## 6. Posting test

Only test posting after user review, because it publishes to LinkedIn.

The post endpoint is:

```http
POST https://api.linkedin.com/v2/ugcPosts
X-Restli-Protocol-Version: 2.0.0
Authorization: Bearer <access_token>
```

Posting requires:

```text
w_member_social
```

and an author URN such as:

```text
urn:li:person:<member-id>
```

## 7. Security notes

- Never commit the LinkedIn Client Secret.
- If the secret appears in screenshots, chats, GitHub, logs, or browser history, rotate it in LinkedIn Developer Portal.
- Start with profile read tests before enabling posting workflows.
- Treat posting as a manual-confirmation workflow unless the use case is fully controlled.

## 8. Limitations

This LinkedIn app does not grant LinkedIn Easy Apply automation or Jobs API access. It supports only the products/scopes approved in the LinkedIn Developer Portal, mainly OpenID profile/email and Share on LinkedIn.
