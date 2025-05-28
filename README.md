# Spring Looks API Gateway

## Index

### Project Description
### Project Setup
### Dependencies
### Demo

## Project Description
The Gateway is used to manage API calls from the client and security for the Spring Looks Website.

## Project Setup
1. Start the keycloak container with "docker compose up -d" in the main project directory.
2. Go to the uri used for the keycloak server (localhost:8181)
2. Add a realm for the Spring-looks application security
3. Add a client (client authentication ON, Authentication Flow being Service Accounts Roles)
4. Start the Application with "./mvnw spring-boot:run" or through your IDE (Other Spring Looks projects will also need to be started)
5. Requests made to the Gateway will return "401 unauthorized" unless the request is sent with an auth token with credentials

## Dependencies
1. Keycloak
2. MySQL
3. Docker
4. Spring Gateway MVC
5. Spring OAUTH2 Resource Server

## Demo
TBC