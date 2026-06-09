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

There are two independent exposure choices, plus the optional bundled
[oauth2-proxy](https://oauth2-proxy.github.io/oauth2-proxy/) for authentication:

1. **How the dashboard is exposed** — an `Ingress` (`ingress.enabled`) **or** a
   Gateway API `HTTPRoute` (`httpRoute.enabled`).
2. **How oauth2-proxy is exposed** (only when `oauth2-proxy.enabled`) — its own
   `Ingress` (`oauth2-proxy.ingress.enabled`) **or** `HTTPRoute`
   (`oauth2-proxy.gatewayApi.enabled`).

The dashboard's auth mechanism follows how the **dashboard** is exposed:

| Dashboard via | Auth mechanism | Configured by |
|---|---|---|
| Ingress | nginx `auth-url` / `auth-signin` annotations | `authHost` |
| HTTPRoute | Envoy Gateway `SecurityPolicy` (extAuth → oauth2-proxy Service) | `httpRoute.auth.enabled` (requires Envoy Gateway) |

It's recommended that the dashboard and oauth2-proxy share the same data plane (Ingress or Gateway API) to avoid complexity and potential issues with cookie domains, CORS, etc.

For the HTTPRoute happy path, oauth2-proxy must run in reverse-proxy mode with a
`redirect-url` so unauthenticated requests are redirected to the sign-in page — the
`SecurityPolicy` has no `auth-signin` equivalent of its own.