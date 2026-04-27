# Inner Circle UI Local Run

Goal: Agree on a secure and consistent approach to configuring CORS and authentication for running frontend projects + Dev Container.

Context discussion: why we need local run, bad dev experience.

## Where we are

- UI cannot be run easily locally against its BFF (need to run ui, layout-ui, api, and copy and pass tokens to local-storage manually)
- UI cannot be run easily locally against *local-env* (copy access token received from local-env to ui local storage, run ui)
- Cypress is used as a sandbox locally to do the CSS job
- Cypress e2e can be run locally against *local-env*
- To test something locally we build a docker image of ui, load it to *local-env*, and check

### Inconsistent CORS Against APIs

CORS issue in *time-ui*: why *books-api* has a permissive CORS setup (*) while *time-api* doesn’t?

Api returns CORS settings to browsers in response to PREFETCH requests (before actual requests),
browsers enforce received CORS policies.
E.g. *bad-example.com* cannot send requests to *example.com* because *example.com*'s CORS is restricted to only its own domain.
But you can send requests from *curl* to *example.com* and CORS will be ignored because there is no browser to protect you.

We want it to be configured the following way:
- When develop use * for CORS (local api and *local-env* api)
- In Prod allow only its domain (we shouldn't be able to call prod apis from locally run ui)
>Note: this is a separate issue

## Where we want to be

Dev Container for frontend (UI + layout-ui + all dev tools) for cross-platform DevExp.

### Dev Container Docker Compose

- Node of a specific version
- Cypress of a specific version
- layout-ui

### Important Modes

- **start** - to run the website against locally running api (by the example of *time-api* which is on 4507 and *time-ui* which is on 3507)
- **start-local-env** - to run the website against Local Env api (*time-ui* uses the same 3507 port)
- **cypress:run:e2e** - to run e2e tests against local ui (NOT *local-env* one!)
- **cypress:run:e2e:local-env** - to run e2e tests against running *local-env* (with real backend and ui as a part of it)

The problem of this naming convention is that e.g. cypress:run:e2e targets local *time-ui* 3507 port which migh be running in one of 2 modes: start and start-local-env. This is due to re-using the same port for both modes which is okeish here but not very explicit.

### Authentication

### Authentication npm run start mode

Calling local api on port 4507 in MockForDevelopment mode, using its own *layout-ui*, not calling *local-env*.

We can grab a fake token from api's *mock-server-initialization.json* where there are all permissions available. The api in MockForDevelopment doesn't need a real token.

We can manually put this accessToken to the ui's local storage.

How to automatically put the token when start to avoid manual burden? </br>

If we hardcode the fake token in ui, how do we operate when update api? We need to find a way not to forget to update it in ui too.
>Note: postpone automation until everything else works with manual copy&paste of the token to LS.

Options:
- Mock auth
&
- Turn off real auth in api
- Proxy auth (no running *local-env*, thus, it is a no-go)
- Run auth services as part of *docker-compose* (too heavy, it even includes employees service, it is a no-go)

**What to do with refresh?** </br>
Let's try to turn off refresh in this mode using a new env variable.
>Note: this is a separate issue

Need to complete&refine an issue regarding config&env where we need to get rid of the legacy setup and use the vite one.

### Authentication with npm run start-local-env mode

Need to get a real token from *local-env*. 

**start-local-env mode**

**Steps:**
1. Run *time-ui* locally on port 3507
1. *layout-ui* is used from *local-env* (via a new env var) (or proxy)
1. api is used from *local-env* (or proxy)
1. refresh is covered with proxy and thus is on and is real from *local-env*
1. Since there is no token in local storage of http://localhost:3507/time
1. *time-ui* has a configured proxy from http://localhost:3507/auth to http://localhost:30090/auth
1. Redirects to login page on http://localhost:3507/auth?redirectUrl=http://localhost:3507/time
1. Enter login and password from a real Local Env account
1. In creates a token in local storage of http://localhost:3507 and redirects to http://localhost:3507/time
1. All requests to api go to http://localhost:30090/api/time(sort-of) with this token

We want to be redirected not to employees but to the service where we were unauthenticated</br>
**Books issue**</br>
When I try to scan a QR and I'm not logged in to IC</br>
I pass login and pass</br>
And I am redirected to *employees page* but shoud be redirected to the *book page*</br>
>Note: this is a separate issue
>Note: need to check vulnerabilities of redirectUrl approach and how to handle them correctly.
