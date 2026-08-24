# 008: Downloading Files from books-api and layout-ui for Local Run

## Status

Proposed

## Context

To run books-ui locally with a mocked API (`npm start`) and to run e2e tests, we need two files from `inner-circle-books-api`:

- `docker-compose.yml` - how to start books-api and its database and MockServer
- `e2e/mock-server-initialization.json` - the mocked endpoints, including the login response. books-ui takes the local debug token from it

Later we also needed `docker-compose.yml` from `inner-circle-layout-ui`.

We do not need other files from these repositories. These files must always match the current state of the branch we need (usually `master`), without any manual steps from the developer.

Git has two built-in ways to work with code from another repository inside your own: submodule and subtree. We looked at both of them.

## Decision

We download the files we need from GitHub one by one with a plain HTTP request, instead of connecting the whole repository.

A Node script (`local-run/prepare-local-run.js`) takes them from `raw.githubusercontent.com` on every local run and puts them into the `local-run/` folder. All three files are in `.gitignore`, because they are downloaded files, not our code.

By default the files come from `master`. The `API_REF` variable lets you choose another branch or commit, and `API_LOCAL_PATH` lets you take `mock-server-initialization.json` from a local books-api folder, so you can check changes that you have not pushed yet.

All of this is needed only for the local run.

### Advantages

- The files are always fresh, nobody has to update them by hand
- No code and no history from other repositories gets into the books-ui repository
- A new developer only runs `npm ci && npm start`, with no extra commands

### Disadvantages

- We need the internet on every start, not once during setup: if GitHub is not available, `npm start` fails
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
