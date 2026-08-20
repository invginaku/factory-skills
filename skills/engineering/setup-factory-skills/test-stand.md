# Test stand

Where `investigate` reproduces behaviour, and what it may do there. Written in the repo as `docs/agents/test-stand.md`.

Credentials never live in this file. It names the environment variables that carry them; the values sit in an untracked env file that each person fills in for themselves.

## Environment

- **App URL** (the sign-in page): `<https://...>`, from `$PROBE_APP_URL`
- **API base** (the services the app calls): `<https://...>`, from `$PROBE_API_CLUSTER`
- **Allowed hosts**: `<host, host, host>`, from `$PROBE_ALLOWED_HOSTS`. These are the only hosts the agent may reach. Production is absent from this list by construction.

## Account

A dedicated test account, never a person's own session.

- Login and password: `$PROBE_EMAIL` / `$PROBE_PASSWORD`
- Sign-in flow: fill the email and password fields on the app URL, submit, wait for the redirect.
- Access token: the app stores it in `localStorage` under `<key>`. Read it inside `page.evaluate` and send it as `Authorization: Bearer <token>`. It stays inside the page context.

## Mutations

`$PROBE_ALLOW_MUTATIONS` gates everything in this section. When it is `false`, the agent describes the mutation it needs and stops.

**Safe here** (each one paired with its cleanup):

- `<add to cart>` → cleanup: `<remove from cart>`
- `<create a draft>` → cleanup: `<cancel the draft>`

**Never**: paying, moving money, messaging real people, changing account settings or permissions, deleting data the agent did not create, and anything else that cannot be undone.

## Reproduction notes

Repo-specific facts worth caching for whoever reproduces here: which endpoints answer which question, which routes have usable test data, known quirks of the stand.

- `<POST /some-service/search>`: `<what it answers>`
- `<a route with reliable data>`: `<why it is useful>`
