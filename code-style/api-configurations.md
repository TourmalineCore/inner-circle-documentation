## Configurations

- DbOnly - spins up only database and its pgAdmin in docker compose, especially useful to add migrations
- MockForDevelopment - used locally when you run the service in Visual Studio e.g. in Debug and don't want to spin up any external deps
- MockForTests - used locally to run the service in isolation (no external deps) and later separately run its Karate tests
- MockForPullRequest - used in PR pipeline to run the service in isolation (no external deps) and its Karate tests

| Configuration | Db in Docker Compose | Api in Docker Compose | Api in Local Env | MockServer for External Deps | Run Karate Tests | PgAdmin |
| :---------------- | :------: | :------: | :------: | :------: | :------: | :------: |
| DbOnly                 |   Yes   |   No   |   No   |   No   |   No   |   Yes  |
| MockForDevelopment     |   Yes   |   No   |   No   |   Yes  |   No   |   Yes  |
| MockForTests           |   Yes   |   Yes  |   No   |   Yes  |   No   |   No   |
| MockForPullRequest     |   Yes   |   Yes  |   No   |   Yes  |   Yes  |   No   |