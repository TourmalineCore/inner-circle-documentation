# How to Update Screenshot Tests with QR Codes

## Context

The [books-ui](https://github.com/TourmalineCore/inner-circle-books-ui) service generates QR codes based on the application URL (for example: `http://localhost:PORT/...`).

We use Cypress component tests with screenshot comparison (visual regression tests).

The QR code image depends on the full URL, including the port number. If the port changes, the generated QR code image also changes. Because of this, screenshot tests may fail even if the UI has not changed.

## Problem description

Locally (in the dev container), when running:

```bash
npm run cypress:run:component
```

the service usually starts on port `4005`.

In CI pipelines, port `4005` may already be in use (for example, due to parallel runs). In that case, the service starts on the next available port, typically `4006`.

As a result:

1. Locally generated screenshots contain QR codes with `localhost:4005`.
2. In CI, QR codes may contain `localhost:4006`.
3. Screenshot comparison fails due to different QR images.

## Temporary local change for fix difference

To generate screenshots that match CI behavior, you must temporarily change the port in `vite.config.ts`.

### Step 1. Open `vite.config.ts`

Find the following line:

```ts
const BOOKS_PORT = process.env.NODE_ENV === `production` ? LOCAL_ENV_PORT : 4005
```

### Step 2. Temporarily change it to:

```ts
const BOOKS_PORT = process.env.NODE_ENV === `production` ? LOCAL_ENV_PORT : 4006
```

This forces the service to run on port `4006` locally.

**Do not commit** this change.

## Update screenshots

After changing the port, run in the dev container:

```bash
npm run cypress:run:component
```

This will:

* Start the service on port `4006`.
* Generate QR codes with `localhost:4006`.
* Update screenshots.

## Revert the configuration

After screenshots are updated:

1. Revert `vite.config.ts` back to:

```ts
const BOOKS_PORT = process.env.NODE_ENV === `production` ? LOCAL_ENV_PORT : 4005
```

2. Make sure that `vite.config.ts` is not included in your commit.

3. You can commit updated screenshots.

## Notes

If CI starts using a different fallback port in the future, screenshots may fail again.  

This is a workaround for port-related QR instability.

