# swhurl-try-11

throwaway: team platform T2 caller (reports), delete me

A TypeScript app on the [swhurl platform](https://github.com/samclement/swhurl-platform), made from the [swhurl TypeScript template](https://github.com/samclement/swhurl-app-template-typescript). Every push to `main` is checked, published to `ghcr.io/<owner>/swhurl-try-11` and deployed to staging; production changes through **Promote to production**. What the image must provide, the checks and the dependency updates: the [template's guide](https://github.com/samclement/swhurl-app-template-typescript#readme). What the app needs from the platform: [`swhurl.yaml`](swhurl.yaml).

An HTTP service: `src/server.ts` answers on port 8080; the platform probes `GET /healthz`.

## Callers and calling other apps

`src/platform.ts` holds what the platform gives the app ([calling another app](https://github.com/samclement/swhurl-platform/blob/main/docs/apps.md#calling-another-app)):

- **Who is calling.** `identify(request.headers)` returns `{ type: "app", id: "<app>" }` for another app that sent a token the platform issued for this app alone, `{ type: "user", id: "<email>" }` for a signed-in person, or `anonymous`. A bad token throws `Unauthorized`, which `src/server.ts` answers with 401. Every log line and span carries `caller.type` and `caller.id`. For a route only apps may use, call `requireApp(caller)`.
- **Calling another app.** Add `calls: [orders]` to `swhurl.yaml` before the app is created (afterwards: `make app-calls` on the platform, or the app page's **Calls** form). Then `await service("orders").fetch("/path")` reaches the same environment's `orders` with this app's token for it; the callee's spans join the same trace. `GET /call/orders` in `src/server.ts` is an example to replace.

Locally there are no platform keys or tokens: requests without a token work as before and a request with one is refused; to call another app, set `SWHURL_SERVICE_<NAME>_URL` to a local stub.

## Local development

```bash
npm install
npm run dev                             # the SDK stays off locally (the image turns it on)
curl http://localhost:8080/healthz
npm run check                           # type-check, as CI does
npm run build && npm test               # the tests run against the built app
```

To see telemetry locally, `npm run build`, then `OTEL_TRACES_EXPORTER=console node --import ./dist/instrumentation.js dist/main.js` prints spans (or run an OpenTelemetry collector on `localhost:4318` and drop the variable).
