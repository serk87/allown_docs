# Authentication

The MVP supports two authentication modes:

- Development session for preview and internal testing, enabled only with `ENABLE_DEV_SESSION=true`.
- Yandex ID OAuth for browser login.

## Yandex ID OAuth

Backend routes:

- `GET /auth/yandex/start`
- `GET /auth/yandex/callback`
- `GET /api/auth/session`

Required environment variables:

- `YANDEX_OAUTH_CLIENT_ID`
- `YANDEX_OAUTH_CLIENT_SECRET`
- `YANDEX_OAUTH_REDIRECT_URL`
- `AUTH_SESSION_SECRET` with at least 32 bytes of random material

The start route redirects to Yandex OAuth with `response_type=code`, the configured client id, redirect URI, requested scopes, and a signed state value. The callback route validates the signed state and state cookie, exchanges the code for an OAuth token, loads the Yandex profile through `login.yandex.ru/info`, links the Yandex subject to a local user, and issues cookies.

## Session Cookies

`plane_session` is an opaque HTTP-only session cookie. Only the hash is stored in PostgreSQL.

`plane_csrf` is a non-HTTP-only CSRF cookie. Cookie-authenticated mutation routes must send the same value in `X-CSRF-Token`, and the backend also verifies the token against the hashed CSRF value stored on the active session row.

New Yandex users are added to the default workspace with the `developer` role. Existing users are linked by email first, so their current workspace role is preserved.

## Preview Notes

Real Yandex OAuth requires an app registered in Yandex OAuth with the preview callback URL. Without real client credentials, the backend can still verify route wiring and redirects, but the full external callback cannot complete against Yandex.
