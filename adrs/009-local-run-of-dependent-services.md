# 009: Local Run of Dependent Services

## Status

Proposed

## Context

books-ui cannot work alone. It needs three things:

- books-api - gives data about books
- an auth service - gives an authorization token
- layout-ui - gives the header, footer, and sidebar. The app loads them at runtime with module federation

Before this change, `npm start` only started the vite dev server. The developer had to start books-api, auth-api, and layout-ui by hand. It was hard to get a working page quickly, so everyone used local-env instead.

local-env works, but it is too slow for everyday UI work.

We wanted `npm start` to start a working app with one command, with no manual steps, and with hot reload.

## Decision

`npm start` starts everything books-ui needs, then starts the vite dev server:

1. `local-run/prepare-local-run.js` downloads the files we need and gets a token from `e2e/mock-server-initialization.json`. It processes the token and adds it to the config files. Which files it downloads, and why one by one instead of through submodule or subtree, is described in [008](./008-downloading-files-for-local-run.md)
2. `docker compose` starts layout-ui and books-api with its database and mock server
3. `vite --host` starts the books-ui dev server on port 4005

The compose files we download build the services from source, but we do not have that source locally. So next to them we keep two override files (`local-run/api-docker-compose.override.yml`, `local-run/layout-ui-docker-compose.override.yml`), and they replace the build step with ready images from `ghcr.io`.

By default we use the `latest` images, but for books-api you can set the tag with the `IMAGE_TAG` variable - for example, to test the image built for a specific PR. You can also override the books-api files: `API_REF` downloads `docker-compose.yml` and `mock-server-initialization.json` from another branch or commit instead of `master`, and `API_LOCAL_PATH` takes `mock-server-initialization.json` straight from your local books-api folder, so you can check changes you have not pushed yet (more about the download mechanism itself in [008](./008-downloading-files-for-local-run.md)). layout-ui has no such switches: its compose file always comes from `master`, and the image in the override file is fixed at `latest`.

### Authorization without auth-api

Instead of running the real auth service, the app logs itself in as a debug user.

The books-api mock server already has a ready login response with all permissions. `prepare-local-run.js` takes the token from it and writes two values into `env-config.js`:

- `LOCAL_DEBUG_TOKEN` - the token in the form the mock server returns
- `LOCAL_DEBUG_JWT` - the same token, wrapped as an unsigned JWT

The app puts `LOCAL_DEBUG_JWT` into `authService`, which sends it in the `Authorization` header. The wrapping is needed because the app reads permissions from the token with `jwtDecode`, and that library only understands the JWT format.

`LOCAL_DEBUG_TOKEN` goes into the `X-DEBUG-TOKEN` header, which books-api checks.

This is controlled by the `DISABLE_DEBUG_TOKEN` flag: the debug login only turns on when the flag is explicitly set to `false` in `.config-local`. Production never sets this variable, so this login path is off there by default.

### Requests through the vite proxy

In production, books-ui, books-api, and layout-ui sit behind one ingress, so the app calls them with relative paths: `/api/books` and `/layout/...`. Locally these paths do not exist, there is only the books-ui dev server on port 4005.

So the dev server proxies `/api/books` to books-api, and `/layout` to layout-ui. The relative paths work the same locally as in production.

### Three modes

Usually both dependencies run as containers, but you can swap either one for a local checkout.

| Command | books-api | layout-ui |
| --- | --- | --- |
| `npm start` | container, port 6505 | container, port 4406 |
| `npm run start:for-local-layout-ui` | container, port 6505 | `vite preview`, port 4006 |
| `npm run start:for-local-books-api` | local `dotnet run`, port 7000 | container, port 4406 |

The `LOCAL_LAYOUT_UI` and `LOCAL_BOOKS_API` variables switch the proxy targets in `vite.config.ts`.

You start the service you want to test with books-ui locally yourself, in its own repository. For books-api, that means `docker compose --profile MockForDevelopment up` for the database and mock server, then `dotnet run --project ./Api`.

If books-ui runs inside a Dev Container while books-api runs locally on the host, `localhost` inside the container points to the container itself, not to the host machine, so books-api becomes unreachable. `vite.config.ts` checks the `/.dockerenv` file to detect that it runs inside a Dev Container, and in that case it uses `host.docker.internal` instead of `localhost` in the proxy.

layout-ui runs as a separate compose project (`-p inner-circle-layout-ui`), because when several UI services depend on layout-ui and run locally at the same time, it is better to start one shared container than a copy for each service. `npm run local-services:down` only stops the books-api project and leaves the shared layout-ui container running. To stop that one too, there is a separate command, `npm run local-services:down:layout-ui`.

### Working with a local layout-ui

layout-ui needed two additions for this:

- the `npm run start:federation` command. A plain `npm start` does not work here: `vite-plugin-federation` only creates the remote file `inner_circle_layout_ui.js` during a build, and the dev server does not serve it
- its own debug login and a stub route, so layout-ui can render on its own, without a host app and without an auth service

`start:federation` runs in three steps: first `vite build --mode development` builds the project once, then `vite build --watch` and `vite preview --port 4006 --strictPort` run together. `vite preview` serves layout-ui on port 4006, and `vite build --watch` watches the app and rebuilds it on every change, so the process keeps running while the app is up. There is no hot reload, so you need to refresh the page by hand.

The `--mode development` flag is also needed so that `.env.development` loads, with `VITE_DISABLE_DEBUG_TOKEN=false`, which turns on the debug login in layout-ui.

### Advantages

- `npm ci && npm start` is enough for a new developer, with no manual setup across repositories
- Normal hot reload works
- You do not need to run the auth service or enter login details
- You can test books-ui with a locally running api or layout-ui

### Disadvantages

- You need Docker and internet access, or `npm start` fails
- The local debug login flow does not fully match production
- Ports are hardcoded in `vite.config.ts` and in the compose files, and you have to keep them in sync by hand

## Alternatives

### Use local-env for everyday development

Start the whole system in kind through helmfile.

#### Advantages

- Closest to production: real ingress, real auth-api, all services together

#### Disadvantages

- To see a change, you have to rebuild the image and redeploy it
- No hot reload
- Heavy on your computer: it starts the whole system, even though you only need a few services

### Start every service by hand

Start every service the UI depends on manually.

#### Advantages

- No extra tooling in the repository
- Full control over what runs and how

#### Disadvantages

- Many manual steps across several repositories before anything works
- A developer cannot get a working page quickly

### Mock the API in the frontend

Intercept requests in the browser instead of running books-api.

#### Advantages

- You do not need Docker or a backend at all
- Fast startup

#### Disadvantages

- The mocked responses live in books-ui and drift away from the real API as it changes
- It does not test the real request path: headers, status codes, errors

### Run auth-api locally instead of the debug token

Start the real auth service together with the rest.

#### Advantages

- The local login flow matches production exactly
- There is no second login path in the app's code

#### Disadvantages

- One more service, with its own database and secrets, to run and maintain locally
- It duplicates what the books-api mock server already gives us
