# tekton dashboard helm chart

## Setup
First add the Jenkins X chart repository

```sh
helm repo add jxgh https://jenkins-x.github.io/tekton-dashboard-helm-chart/
```
If it already exists be sure to update the local cache
```
helm repo update
```

## Basic install
```
helm upgrade --install tekton-dashboard jxgh/tekton-dashboard-helm-chart
```

## Exposure & authentication

The dashboard can be exposed two ways — pick one:

- **Ingress** (`ingress.host`) — the default. nginx-based, with optional
  [oauth2-proxy](https://oauth2-proxy.github.io/oauth2-proxy/) authentication wired
  through the `authHost` annotations.
- **Gateway API `HTTPRoute`** (`httpRoute.enabled`) — requires
  [Envoy Gateway](https://gateway.envoyproxy.io/) as the Gateway API provider.

The Ingress path and the bundled oauth2-proxy are unchanged; configure them via their
existing values. The sections below cover the `HTTPRoute` path only.

### HTTPRoute authentication

Protect the route with an Envoy Gateway `SecurityPolicy` by enabling **one** of the
two mutually exclusive blocks under `httpRoute.auth`. Leave both disabled for an
unauthenticated route (e.g. when auth is enforced upstream); enabling both fails the
render.

| Block | Mechanism | oauth2-proxy? | Use for |
|---|---|---|---|
| `httpRoute.auth.extAuth.enabled` | [`SecurityPolicy.extAuth`](https://gateway.envoyproxy.io/latest/api/extension_types/#extauth) → bundled oauth2-proxy | Required | GitHub (OAuth2, not OIDC, so it must go through oauth2-proxy). Defaults are pre-filled for oauth2-proxy. |
| `httpRoute.auth.oidc.enabled` | [`SecurityPolicy.oidc`](https://gateway.envoyproxy.io/latest/api/extension_types/#oidc) — native Envoy Gateway OIDC | Not needed | Any OIDC provider (Okta, Google, Entra, Keycloak, …). Fill in your provider details. |

#### `oidc` notes

- The Secret referenced by `httpRoute.auth.oidc.clientSecret.name` **must** expose the
  client secret under the key `client_secret` (an Envoy Gateway requirement; override
  with `httpRoute.auth.oidc.clientSecret.key`).
- `redirectURL` defaults to `https://<httpRoute.host>/oauth2/callback`. It must be a
  path the HTTPRoute matches — Envoy intercepts it; it never reaches the dashboard.
- Authorization/token endpoints are auto-discovered from
  `<issuer>/.well-known/openid-configuration` unless set explicitly.

#### `github` (extAuth) notes

- `extAuth.backendRef.name` defaults to the bundled oauth2-proxy Service
  (`<release>-oauth2-proxy`).
- Envoy's extAuth returns only allow/deny and has no `auth-signin` redirect of its own,
  so oauth2-proxy must run in reverse-proxy mode with a `redirect-url`, and its
  `/oauth2/*` sign-in endpoints must be reachable by the browser.