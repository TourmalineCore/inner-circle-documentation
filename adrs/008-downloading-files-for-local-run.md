# 008: Downloading Files from books-api and layout-ui for Local Run

## Status

Accepted

## Context

To run books-ui locally with a mocked API and layout-ui, and to run e2e tests, we need files from two other repositories:

- from `inner-circle-books-api`: `docker-compose.yml` and the mock server initialization config (the mocked endpoints, including the login response that books-ui takes its local debug token from)
- from `inner-circle-layout-ui`: `docker-compose.yml`

We do not need other files from these repositories. These files must always match the current state of the branch we need (usually `master`), without any manual steps from the developer.

Git has two built-in ways to work with code from another repository inside your own: submodule and subtree. We looked at both of them.

## Decision

We download the files we need from GitHub one by one with a plain HTTP request, instead of connecting the whole repository.

A Node script, run through `npm run prepare-local-run` (`node --env-file=.env.local local-run/prepare-local-run.js`), fetches them from `raw.githubusercontent.com` and puts them into the `local-run/` folder. They are not committed - they are downloaded on demand, not our code.

By default the files come from `master`. `API_REF` and `LAYOUT_REF` let you choose another branch or commit for books-api's and layout-ui's files, one variable per repository. `API_LOCAL_PATH` additionally lets you take the mock server initialization config from a local books-api folder, so you can check changes that you have not pushed yet - layout-ui has no local file to fetch, so it has no such switch.

`prepare-local-run` runs as the first step of `npm run local-services:up` (which also starts the books-api and layout-ui containers), and it also runs automatically every time the Dev Container starts. All of this is needed only for the local run.

### Advantages

- The files are always fresh, nobody has to update them by hand
- No code and no history from other repositories gets into the books-ui repository
- A new developer only needs the Dev Container - the files are fetched automatically, with no extra commands

### Disadvantages

- We need the internet on every start, not once during setup: if GitHub is not available, the local run fails
- We depend on the current state of `master` in another repository (for example, if somebody renames a file or changes it in a way that does not work for us)

## Alternatives

### Git Submodule

It connects another repository inside ours as a separate folder. A `.gitmodules` file appears in the root with the address of the repository and the path to the folder. Git keeps the information about the fixed commit in a separate record (gitlink).

When somebody clones our repository, the folder stays empty until the developer runs `git submodule update --init`. Nothing updates by itself: you have to go into the folder, pull the new commit and commit the updated link in your own repository.

#### Advantages

- The repository does not grow, it keeps only a link to a commit, not the files themselves
- The version is fixed clearly, everybody gets exactly the commit that is written down
- Commits from the other repository do not go into our history

#### Disadvantages

- There is an extra step after cloning, and it is easy to forget it
- It connects the whole repository
- You can update it only by hand

### Git Subtree

It adds the files of another repository into ours as normal files. The files look like we wrote them ourselves.

You update them with the `git subtree pull` command, and it creates a merge commit.

#### Advantages

- A normal `git clone` gives you all the files at once, with no extra steps
- You can edit the files in your own repository and commit them as usual

#### Disadvantages

- It adds the whole repository, not the files we chose
- The repository grows: every update adds the files into the history
- You can update it only by hand
- You cannot tell the files of the other repository apart from our own code
