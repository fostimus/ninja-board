# Quarkus Migration
## Goals
* Enhance by adopting a MicroService Architecture
* Allow single bundled deployment
* Enhance performance by using Red Hat Technology (Quarkus)
* Prioritising simplicity and maintainability

## Requirements
* OpenJDK 11 - [Download from Red Hat](https://developers.redhat.com/products/openjdk/download)

## Structure

This project has multiple components.
So far they are: 3 Microservices, 3 supporting shared libraries.

* Microservices that can be bundled user the `all-service` module
    * `user-service` - contains features related to the user such as Registration.
    * `scorecard-service` - groups features related to scorecard such as "Point Addition".
    * `data-service` - manages data and offer CRUD operations on entities.
* Shared Libraries
    * `entities` - shared data model
    * `data-client` - interface for accessing the CRUD services of `data-service`
    * `test-helper` - helper classes for testing
    
`all-service` is a special project that bundles all the available microservices. In addition, it includes an Open API
specification available at `/openapi`. Running it in dev mode (`mvn quarkus:dev -f all-service`) will enable Swagger UI 
at `/swagger-ui`.

The Shared Libraries are regular java projects indexed with jandex so that they can be Quarkus enhanced later.
The services are Quarkus Project and inherit from `quarkus-pom`.

## Create Environment variables before running
### Linux
```shell script
export USERS_LDAP_PROVIDER=i-wont-tell-you-the-real-value
export USERS_LDAP_BASEDN=i-really-wont-tell-you-the-real-value
export DATABASE_FILE=your-database-file-location # not required
mvn install
java -jar all-service/build/all-service-2.0.0-SNAPSHOT-runner.jar
```

### Windows (PowerShell)
```shell script
[System.Environment]::SetEnvironmentVariable('USERS_LDAP_PROVIDER','i-wont-tell-you-the-real-value',
[System.EnvironmentVariableTarget]::User) # Need to set only once in user scope
[System.Environment]::SetEnvironmentVariable('USERS_LDAP_BASEDN','i-really-wont-tell-you-the-real-value',
[System.EnvironmentVariableTarget]::User) # Need to set only once in user scope
[System.Environment]::SetEnvironmentVariable('DATABASE_FILE','use-your-ninja-database.json',
[System.EnvironmentVariableTarget]::User) # Need to set only once in user scope
```

Note that the default location for the database file is: database.json. A good database example can be found 
[here](test-helper/src/main/resources/com/redhat/services/ninja/test/database.json).

## Building and running
```shell script
mvn install
java -jar all-service/target/all-service-2.0.0-SNAPSHOT-runner.jar
```

For native application:
```shell script
mvn install
mvn package -Pnative -f all-service
./all-service/target/all-service-2.0.0-SNAPSHOT-runner
```

## Testing

### Important Note
* Endpoint start with /data are for internal use only.
* A scorecard cannot be created directly, creating a user creates its scorecard
* The only way to modify a scorecard is by incrementing

### Operations

Refer to [OpenAPI Spec](all-service/src/main/resources/META-INF/openapi.yaml).

### Linux
```shell script
curl localhost:8080/user/some-user-id-that-do-exist
curl localhost:8080/user/some-user-id-that-dont-exist
curl -H "Content-Type: application/json" --data '{"username":"some-user-id-that-do-exist"}' localhost:8080/user
curl localhost:8080/user/mail/some-mail-that-exist
curl localhost:8080/data/user
curl localhost:8080/data/scorecard
curl localhost:8080/data/event
```

### Windows (PowerShell)
```shell script
Invoke-RestMethod -Uri http://localhost:8080/user/some-user-id-that-do-exist
Invoke-RestMethod -Uri http://localhost:8080/user/some-user-id-that-dont-exist
Invoke-RestMethod -ContentType 'application/json' -Body $(@{username='some-user-id-that-do-exist'} | ConvertTo-Json) -Uri http://localhost:8080/user -Method POST
Invoke-RestMethod -Uri http://localhost:8080/user/mail/some-mail-that-exist
Invoke-RestMethod localhost:8080/data/user
Invoke-RestMethod localhost:8080/data/scorecard
Invoke-RestMethod localhost:8080/data/event
```

Or (PowerShell)
```shell script
curl http://localhost:8080/user/some-user-id-that-do-exist
curl http://localhost:8080/user/some-user-id-that-dont-exist
curl http://localhost:8080/user/mail/some-mail-that-exist
curl http://localhost:8080/user -ContentType 'application/json' -Body $(@{username='some-user-id-that-do-exist'} | ConvertTo-Json) -Method POST
curl http://localhost:8080/data/user
curl http://localhost:8080/data/scorecard
curl http://localhost:8080/data/event
```