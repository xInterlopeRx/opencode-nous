# opencode-nous

Patch adding Nous Research Portal sign-in (OAuth) plus model discovery to
OpenCode V2, ported from the Nous device-code flow.

## Contents

- `nous-portal-provider.patch` — a `git format-patch` output containing a single
  commit (`feat(core): add Nous Portal device-code OAuth provider`) that can be
  applied with `git am` (or reviewed with `git apply --check`).

## What the patch adds

- **Nous Portal device-code OAuth** — registers a `Nous Portal device code`
  authentication method on the Nous provider plugin. Login requests a device
  code, then polls the token endpoint (handling `authorization_pending` and
  `slow_down`) until the user authorizes in the browser.
- **Token refresh** — refreshes the inference token via
  `POST {portal}/api/oauth/token` with `grant_type=refresh_token` and the
  `x-nous-refresh-token` header, rotating the stored refresh token.
- **Model discovery** — announces `/v1/models` from the inference endpoint for
  the active credential.

Security constraints in the implementation:

- Only HTTPS inference base URLs from a fixed host allowlist
  (`inference-api.nousresearch.com`, `welcome-api.nousresearch.com`) are used
  for discovery; other Portal-provided URLs are rejected.
- Access tokens and refresh tokens are never logged or exposed.

## Applying

```sh
git am nous-portal-provider.patch
```

## Tests

The patch includes `packages/core/test/plugin/provider-nous.test.ts` covering
method registration, the full connect → poll → refresh → discovery flow,
rotated refresh tokens, hermes-model filtering, and inference-URL allowlist
rejection.

## Notes

- Built against the OpenCode V2 `packages/core` provider architecture.
- The Portal client ID used (`hermes-cli`, scope `inference:invoke`) comes from
  the established Nous device flow; live acceptance by the Portal is
  unverified.