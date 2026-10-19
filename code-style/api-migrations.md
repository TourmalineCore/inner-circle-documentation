# Migrations

Go through [these](https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/?tabs=dotnet-core-cli) Migrations chapters to pick up some basic understanding of Migrations.

Connection string is taken from `MockForDevelopment` configuration due to `ASPNETCORE_ENVIRONMENT` env variable in `removeEnv` of Dev Container. 

#### Add Migration

To add a new migration with the domain changes execute the following script in Terminal:
```cli
dotnet ef migrations add <YOUR_NEW_MIGRATION_NAME> --startup-project ./Api/Api.csproj --project ./Application/Application.csproj --context AppDbContext --verbose
```

#### Remove Migration

To remove the last migration execute the following script in Terminal:
```cli
dotnet ef migrations remove --startup-project ./Api/Api.csproj --project ./Application/Application.csproj --context AppDbContext --verbose
```
>Note: You cannot do that if this migration applied to production!

#### Update Database

To apply pending migrations execute the following script in Terminal:
```cli
dotnet ef database update --startup-project ./Api/Api.csproj --project ./Application/Application.csproj --context AppDbContext --verbose
```