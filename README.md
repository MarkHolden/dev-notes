# dev-notes
Everything I have ever learned (or at least things I need written down because I don't use enough)

### Error:
`env: bash\r: No such file or directory`
#### Solution:
- In VS Code, change the line endings to `LF` rather than `CLRF`
- Save
- Try again
- If it still happens, try the line endings for other scripts called by the script

### Error:
`sh: ts-mocha: command not found`
#### Solution:
`npm install`

### Error:
`error  Parsing error: Unexpected token :`
#### Solution:
change the `.eslintrc.js` file a(n) `.eslintrc.json` file. Ie. Entitlements Service.

### Error:
Oh Shit, I ran `docker-compose down`, and all my databases got deleted.
#### Solution:
Run the database scripts for those databases - you MUST select the right database on which
the scripts will be run!
For the queries service, you probably won't need them all.

### Error:
Getting a 401 against a service running locally, but no auth errors in the logs!
#### Solution:
Verify the appsettings.json has `ENV = local`

### Error:
Getting a 401 in a service running in docker, and 400 in the identities service (in docker), 401 in the factors service
#### Solution:
?

### Error:
localhost:3500 isn't working as a url in the reader config or in a call within the docker container
#### Solution:
Try http://host.docker.internal:3500

### Error:
`nodename nor servname provided, or not known`
#### Solution:
Make sure all the services that are being called are built and running in docker.

### Error:
No response returned from Swagger, or weird SSL error from Postman
#### Solution:
Verify that you are using `http` and not `https`.

### Error:
In Postman
```
"One or more validation errors occurred."
"'0xC2' is an invalid start of a value. Path: $ | LineNumber: 0 | BytePositionInLine: 0."
```
#### Solution:
Hit the `beautify` button (probably some janky whitespace character from teams)

### Error:
```
System.Net.Http.HttpRequestException: An error occurred while sending the request.
  ---> System.IO.IOException: The response ended prematurely.
```
#### Solution:
Look thru the error to find out what service is returning that error
Check docker logs of that service
Observe that there is some weird error
Rebuild the docker image with the correct sdk and aspnet images (Dockerfile)
Try again

### Error:
The NpgsqlDbType 'Citext' isn't present in your database. You may need to install an
extension or upgrade to a newer version.
(When you check and confirm that, yes, citext is present in the database)
#### Solution:
Restart computer
Drop Queries Database
Re-up all required docker containers
Run Queries scripts 001 (partially) and 037

### Error:
Could not load type 'Service.Base.Health.Checks.ResourceStoreWithJournalHealthCheck`1'
from assembly 'ServiceBase, Version=...
#### Solution:
Remove the following packages from all dependencies (the SDKs) and try again:
Data.Access
Service.Base

### Error (POSTGRESQL):
[42703] ERROR: column "name_of_some_cte" does not exist
#### Solution
Don't try to use ctes like variables
You have to
```sql
SELECT * FROM cte
```

### Postgres Scripts
#### When you want to search for a field in json:
```sql
SELECT t.*
    FROM public."Resources" t
    WHERE "Resource"::jsonb ->> 'name' = 'ROCKY MOUNTAIN STEEL SERV'
    LIMIT 501
```
#### When you want to search for a key in a JSON value
(such as searching for a string in a string array)
Use the `?` operator, AKA "Does the key/element string exist within the JSON value?" Operator
```sql
SELECT t.*, CTID
    FROM public.Stuff t
    WHERE t.Things::jsonb ? '31400'
```

#### When you want to filter by a value in an object in a json array
Use the `@>` operator, AKA "Does the first JSON value contain the second?" Operator
```sql
SELECT m."Resource"
    FROM "Resources" m
    WHERE (m."Resource"::jsonb ->> 'Widgets')::jsonb @> '[{"IsActive": true}]'
```
[Postgres Documentation 9.16. JSON Functions and Operators](https://www.postgresql.org/docs/current/functions-json.html)


### How do I "View Surrounding Logs" in New Relic?
- If you don't have an entity.name column, add it by clicking the "Add Column" button above the list.
- Click on the link for the desired log in the entity.name column.
- Click "Show surrounding logs".

### Service models and SDK Models:
#### Service Models
The service model allows you to add things like data conversion methods and extend the
AuditableBase without needing to hide fields.
Put custom methods on the service model, ie. HydrateAuditData().
Service model is also the one that extends AuditableBase.

#### SDK Models
The SDK models should be POCOs.
The service should use SDK models when it can to prevent duplication, but not at the
expense of encapsulation.
SDK models should have enums as strings.
Enums should also be in the SDK, but the model itself will not use them, it's just
for reference.

### PR Reviews
#### PRs in the Query Service
- Make sure there is a `GRANT` for `queries_search`. Should be either `EXECUTE` (on functions) or `select, insert, update, delete` on tables.

#### `OrdinalIgnoreCase` or `InvariantCultureIgnoreCase`?
[According to Doug](https://github.com/LegalShield/atlas-entitlements-reader/pull/103#discussion_r1170188702),
> https://stackoverflow.com/questions/492799/difference-between-invariantculture-and-ordinal-string-comparison
>
> Usually ordinal is what we want.


#### Danny's General Thoughts on Service Models and SDK models:
1. The noun service should have the Resource version which inherits AuditableBase
2. The SDK should have a non-resource version which pulls the AuditableBase fields into the class but does not have a dependency on the Resource.
3. The SDK should have the non-resource models that the resource can use

### Before Deploying:
Service/SDK changes may break readers! Regression testing for calling readers will find these problems.
- member migration
- provision
- orders (-orders-journal-reader-service)
- identity merge
- etc.

### IdentityId in the route
> [Removing the IdentityId from the route] is a breaking change. The identityId in the route params is used in the auth middleware to validate the jwt sub.
>
> If you absolutely must have a route that is identity-less, then you can add another route to this method.
>
> -- Danny

### Left Joins
> These left joins are expensive. Can we turn these into ctes and select from there? The execution plan will be bad here
>
> -- Danny

### Pagination
> IMO, take these out until we have a need. Pagination is inherently something I want to trim down our usage significantly on
>
> -- Danny

### Constants in C# should be `UPPERCASE_SNAKE_CASE`

### Scripting For Loop
```sh
for i in {1..10}; do LS_ENV=uat npm run test; done;
```

### Reattach logs to a running container:
```sh
docker-compose logs <service name from docker-compose.yml> -f
```

### Packages that should not be in SDKs:
```
Data.Access
Service.Base
```

These two packages will cause docker issues downstream
```Could not load type
'Service.Base.Helth.Checks.ResourceStoreWithJournalHealthCheck`1' from assembly 'ServiceBase, Version=x.x.x.x, Culture-neutral, PublicKeyToken=null
```

Nullable reference types: disable

Member migrations - check the price

### git merge strategy:
```sh
git config --get pull.rebase # should be false.
```

### Can't find the npm package name?
Try looking in the `package.json` for the SDK you want to pull in.

### dotnet commands:
Want to create a local nuget package for an SDK? This command puts it in the global packages folder
```sh
dotnet pack -c Release -o ../../local\ nuget
```

### Database weirdness
| What you are looking for | Name |
| --- | --- |
|MMS|classic_member|
|MMR|member_migration|

### New SDK urls:
[https://provison.api-internal.dev-shield-service.com](https://provison.api-internal.dev-shield-service.com)

[https://provison.api-internal.uat-shield-service.com](https://provison.api-internal.uat-shield-service.com)

[https://provison.api-internal.shield-service.com](https://provison.api-internal.shield-service.com)

### Common NpgSql Errors:

| Code | Error | Likely Solution |
| --- | --- | --- |
|42P01|undefined_table|Verify that the table exists|
|3D000|invalid_catalog_name|Verify that the database exists|
|42501|insufficient_privilege|Add grants (see entitlement service database.sql)|

### Acronyms:

| Acronym | Meaning |
| --- | --- |
|PDC|Product Development Committee|
|ET|Executive Team|
