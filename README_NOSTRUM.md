# DB_NAME and DB_AUTHSOURCE

Clarifies environment variables and provides a reproducible manual workflow to create the MongoDB user when the init script does not run.

## Environment variables
- DB_NAME: target application database name (example: myapp).
- DB_AUTHSOURCE: authentication database where the user is created (typically `admin`).

## When the init script fails
If the container's initialization scripts under `/docker-entrypoint-initdb.d` did not run or failed, create the user manually using mongosh.

## Manual user creation (commands)
1. Attach to the running MongoDB container and open mongosh (adjust container name and credentials):
```
docker exec -it <mongodb-container> mongosh --username "<rootUser>" --password "<rootPassword>" --authenticationDatabase "<DB_AUTHSOURCE>"
```
2. Switch to the auth database and create the application user. Replace values as needed:
```
use <DB_AUTHSOURCE>

db.createUser({
    user: "appuser",
    pwd: "strongPassword",
    roles: [
        { role: "read", db: "<DB_NAME>" },
    ]
})
```

## Verify access
From the application host or another client, authenticate against the auth database:
```
mongosh "mongodb://appuser:strongPassword@<host>:<port>/<DB_NAME>?authSource=<DB_AUTHSOURCE>"
```
Run a simple read/write check:
```
use <DB_NAME>
db.test.insertOne({ok:1})
db.test.findOne({ok:1})
```

## Troubleshooting
- Inspect container logs for init errors:
```
docker logs <mongodb-container>
```
- Confirm initializer files exist and are readable in the container at `/docker-entrypoint-initdb.d`.
- If re-running initialization is required, remove the database volume so initialization runs on a fresh data directory:
```
docker-compose down -v
docker-compose up -d
```
- Ensure file types in `/docker-entrypoint-initdb.d` are supported (`.js`, `.sh`, `.mongosh`), executable when required, and owned by the container user.
