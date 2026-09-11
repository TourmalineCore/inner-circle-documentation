# 009: Local Run of Dependent Services

## Status

Accepted

## Context

books-ui cannot work alone. It needs two things:

- books-api - gives data about books
- layout-ui - gives the header, footer, and sidebar. The app loads it at runtime with module federation

Running everything by hand across three repositories was slow and error-prone, so the repo used to lean on `local-env` for everyday UI work. `local-env` works, but it is too slow for everyday UI work.

We wanted a new developer to get a working page with no manual cross-repository setup, and with hot reload.

## Decision

The repo is built around a Dev Container. Opening it in VS Code/Codespaces does the setup automatically:

1. `npm ci` installs dependencies (once, when the container is created)
2. `npm run create-config:local` builds `public/env-config.js` from the keys listed in `.env-vars`, taking their values from the container environment (`containerEnv` in `.devcontainer/devcontainer.json`)
3. `npm run local-services:up` downloads the compose files (and, for books-api, the mock server config) from GitHub as described in [008](./008-downloading-files-for-local-run.md), then starts books-api and layout-ui as containers

Steps 2 and 3 run on every container start, so each session begins with a fresh config and up to date images. `npm start` (`vite --host`) then just starts the dev server on port 3505; the config and containers are already in place.

The compose files are downloaded as-is; they build the services from source, which the developer does not have locally. Local overrides (`local-run/api-docker-compose.override.yml`, `local-run/layout-ui-docker-compose.override.yml`) replace the build step with ready images from `ghcr.io`.

By default we use the `latest` images, but `API_IMAGE_TAG` and `LAYOUT_IMAGE_TAG` let you pin a different tag - for example, the image built for an open pull request (published as `sha-<commit sha>`, in both short and full form). `API_REF` and `LAYOUT_REF` let you fetch the compose files themselves from another branch or commit instead of `master`. All four variables are set independently and can be combined.

### Authorization without auth-api

Instead of running the real auth service, the app logs itself in as a debug user.

The books-api mock config already has a ready login response with all permissions. `prepare-local-run` takes the token from it and writes it into `public/env-config.js` as `DEBUG_TOKEN`. `DEBUG_TOKEN` is not one of the keys listed in `.env-vars`, so a plain `create-config:local` does not produce it - `prepare-local-run` appends it afterwards, and both steps are needed for a working login.

This is controlled by the `DISABLE_DEBUG_TOKEN` flag, one of the keys in `.env-vars`/`containerEnv`. Production does not set this variable to enable it, so this login path is off there by default.

### Requests through the vite proxy

In the production environment, the books-ui, books-api, and layout-ui components share the same origin, so the application can access them using relative paths: /api/books and /layout/.... Locally, only the books-ui dev server is running, so it proxies these paths itself: /api/books to API_URL, and /layout to LAYOUT_UI_URL. This way, the relative paths resolve the same way locally as in production. Both variables live in .env.local and default to the container ports (http://localhost:6505 and http://localhost:6500).

### Running a service from its own repo instead of a container

Usually both dependencies run as containers, but you can point the proxy at a local checkout instead by overriding `API_URL` or `LAYOUT_UI_URL` and stopping the matching container:

- **layout-ui**: start it from its own repo with `npm run start:federation` (served on port 4500 - see the layout-ui README), stop the shared container with `npm run local-services:down:layout-ui`, then run books-ui with `LAYOUT_UI_URL=http://localhost:4500/layout npm start`, or change the value in `.env.local` and run `npm start`.
- **books-api**: start it from its own repo ([README](https://github.com/TourmalineCore/inner-circle-books-api#develop-inside-dev-container)), stop its container with `npm run local-services:down:api`, then run books-ui with `API_URL=http://localhost:4505 npm start`, or change the value in `.env.local` and run `npm start`. If your books-api checkout also changes the mock config, take it from there instead of GitHub with `API_LOCAL_PATH=../inner-circle-books-api npm run prepare-local-run` - this only works outside the Dev Container, since only the books-ui repo is mounted inside it.

layout-ui runs as a separate compose project (`-p inner-circle-layout-ui`), because when several UI services depend on layout-ui and run locally at the same time, it's better to start one shared container than a separate copy for each service. `npm run local-services:api` only stops the books-api project and leaves the shared layout-ui container running; `npm run local-services:down:layout-ui` stops that one too. `npm run local-services:down` stops everything.

### Advantages

- Opening the Dev Container is enough for a new developer, with no manual setup across repositories
- Normal hot reload works
- You do not need to run the auth service or enter login details
- You can test books-ui with a locally running API or layout-ui

### Disadvantages

- You need Docker and internet access, or the container setup fails
- The local debug login flow does not fully match production
- Ports are hardcoded in `vite.config.ts`, `.env.local`, and the compose files, and you have to keep them in sync by hand

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
