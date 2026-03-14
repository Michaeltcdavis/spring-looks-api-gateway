# Spring Looks API Gateway

A secure API Gateway for the Spring Looks e-commerce application, providing centralized routing, authentication, and monitoring for microservices.

## Table of Contents

- [Project Description](#project-description)
- [Architecture](#architecture)
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Project Setup](#project-setup)
- [Configuration](#configuration)
- [API Documentation](#api-documentation)
- [Microservices Integration](#microservices-integration)
- [Security](#security)
- [Monitoring & Health Checks](#monitoring--health-checks)
- [Dependencies](#dependencies)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)

## Project Description

The Spring Looks API Gateway serves as the central entry point for all client requests to the Spring Looks e-commerce platform. It provides:

- **Request Routing**: Routes requests to appropriate microservices (Product, Order, Inventory)
- **Security**: JWT-based authentication using Keycloak
- **Circuit Breaking**: Resilience4j for fault tolerance
- **API Documentation**: Aggregated Swagger UI for all services
- **Cross-Origin Resource Sharing**: CORS configuration for web clients
- **Health Monitoring**: Actuator endpoints for service health

## Architecture

```
Client → API Gateway (Port 9000) → Microservices
                                  ├── Product Service (Port 8080)
                                  ├── Order Service (Port 8081)
                                  └── Inventory Service (Port 8082)
```

## Features

- ✅ **JWT Authentication** with Keycloak integration
- ✅ **Circuit Breaker Pattern** with fallback responses
- ✅ **Aggregated API Documentation** via Swagger UI
- ✅ **CORS Support** for frontend applications
- ✅ **Health Monitoring** with Spring Boot Actuator
- ✅ **Resilience4j Integration** for fault tolerance
- ✅ **Service Discovery** and load balancing

## Prerequisites

Before setting up the project, ensure you have:

- **Java 21** or higher
- **Maven 3.8+**
- **Docker** and **Docker Compose**
- **Git**

## Project Setup

### Security First: Configure Keycloak Credentials

Before starting the containers, create a `.env` file in the project root to securely configure Keycloak admin credentials:

```bash
# Create .env file (add to .gitignore)
cat > .env << EOF
KEYCLOAK_ADMIN=your_admin_username
KEYCLOAK_ADMIN_PASSWORD=your_secure_password
MYSQL_ROOT_PASSWORD=your_mysql_root_password
MYSQL_PASSWORD=your_keycloak_db_password
EOF
```

> **Note**: Ensure `.env` is added to your `.gitignore` file to prevent credentials from being committed to version control.

### 1. Start Keycloak and MySQL

```bash
# Start Keycloak container with MySQL database
docker compose up -d
```

Wait for both containers to start completely (check with `docker compose logs -f`).

### 2. Configure Keycloak

1. Access Keycloak admin console: http://localhost:8181
2. Login with admin credentials:
   - Username: `${KEYCLOAK_ADMIN}` (default: set in docker-compose.yml)
   - Password: `${KEYCLOAK_ADMIN_PASSWORD}` (default: set in docker-compose.yml)
   
   > **Security Note**: The default credentials are configured in `docker-compose.yml`. 
   > For production environments, ensure you change these credentials and use secure password management.

3. Create a new realm:
   - Click "Create realm"
   - Realm name: `spring-looks-security`
   - Click "Create"

4. Create a client:
   - Go to Clients → Create client
   - Client ID: `spring-looks-client`
   - Client authentication: `ON`
   - Authentication flow: Enable "Service accounts roles"
   - Click "Save"

5. Configure client settings:
   - Set valid redirect URIs: `http://localhost:9000/*`
   - Set web origins: `http://localhost:9000`

### 3. Start the API Gateway

```bash
# Using Maven wrapper
./mvnw spring-boot:run

# Or using your IDE
# Import the project and run ApiGatewayApplication.java
```

### 4. Start Dependent Microservices

Ensure the following services are running on their respective ports:
- Product Service: `http://localhost:8080`
- Order Service: `http://localhost:8081`
- Inventory Service: `http://localhost:8082`

## Configuration

### Application Properties

Key configuration settings in `src/main/resources/application.properties`:

```properties
# Server Configuration
spring.application.name=api-gateway
server.port=9000

# OAuth2 Resource Server Configuration
spring.security.oauth2.resourceserver.jwt.issuer-uri=http://localhost:8181/realms/spring-looks-security

# Swagger UI Configuration
springdoc.swagger-ui.path=/swagger-ui.html
springdoc.api-docs.path=/api-docs

# Circuit Breaker Configuration
resilience4j.circuitbreaker.configs.default.failureRateThreshold=50
resilience4j.circuitbreaker.configs.default.waitDurationInOpenState=5s
```

### Environment Variables

You can override default configurations using environment variables:

```bash
# Application Configuration
export SERVER_PORT=9000
export KEYCLOAK_URL=http://localhost:8181/realms/spring-looks-security
export PRODUCT_SERVICE_URL=http://localhost:8080
export ORDER_SERVICE_URL=http://localhost:8081
export INVENTORY_SERVICE_URL=http://localhost:8082

# Keycloak Admin Configuration (for setup)
export KEYCLOAK_ADMIN=your_admin_username
export KEYCLOAK_ADMIN_PASSWORD=your_secure_password
```

> **Important**: Never commit credentials to version control. Use environment variables or secure credential management systems.

## API Documentation

### Swagger UI

Access the aggregated API documentation at: http://localhost:9000/swagger-ui.html

This provides a unified view of all microservice APIs:
- Product Service API
- Order Service API  
- Inventory Service API

### Individual Service Documentation

- Product Service: http://localhost:9000/aggregate/product-service/v3/api-docs
- Order Service: http://localhost:9000/aggregate/order-service/v3/api-docs
- Inventory Service: http://localhost:9000/aggregate/inventory-service/v3/api-docs

## Microservices Integration

### Route Configuration

The gateway routes requests based on path patterns:

| Path Pattern | Target Service | Description |
|-------------|----------------|-------------|
| `/api/product` | Product Service (8080) | Product catalog operations |
| `/api/order` | Order Service (8081) | Order management |
| `/api/inventory` | Inventory Service (8082) | Inventory tracking |

### Circuit Breaker

Each route is protected by a circuit breaker:
- **Failure Rate Threshold**: 50%
- **Wait Duration**: 5 seconds
- **Sliding Window**: 10 requests
- **Minimum Calls**: 5
- **Fallback**: Service unavailable message

## Security

### Authentication Flow

1. Client obtains JWT token from Keycloak
2. Client includes token in Authorization header: `Bearer <token>`
3. Gateway validates token with Keycloak
4. Valid requests are forwarded to target services

### Protected Endpoints

All API endpoints require authentication except:
- `/swagger-ui/**` - API documentation
- `/api-docs/**` - OpenAPI specifications
- `/aggregate/**` - Service documentation aggregation

### CORS Configuration

CORS is enabled for all origins with the following methods:
- GET, POST, PUT, DELETE, OPTIONS, HEAD

## Monitoring & Health Checks

### Health Endpoints

- **Application Health**: http://localhost:9000/actuator/health
- **Circuit Breakers**: http://localhost:9000/actuator/circuitbreakers
- **All Actuator Endpoints**: http://localhost:9000/actuator

### Health Check Example

```json
{
  "status": "UP",
  "components": {
    "circuitBreakers": {
      "status": "UP",
      "details": {
        "productServiceCircuitBreaker": "CLOSED"
      }
    }
  }
}
```

## Dependencies

### Core Dependencies

| Dependency | Version | Purpose |
|------------|---------|---------|
| Spring Boot | 3.5.0 | Application framework |
| Spring Cloud Gateway MVC | 2025.0.0-RC1 | API Gateway functionality |
| Spring Security OAuth2 | 3.5.0 | JWT authentication |
| Resilience4j | 3.5.0 | Circuit breaker pattern |
| SpringDoc OpenAPI | 2.8.8 | API documentation |
| Spring Boot Actuator | 3.5.0 | Health monitoring |

### Infrastructure Dependencies

| Service | Version | Purpose |
|---------|---------|---------|
| Keycloak | 26.2 | Authentication server |
| MySQL | 8.0 | Database for Keycloak |
| Docker | Latest | Containerization |
| Java | 21+ | Runtime environment |

## Troubleshooting

### Common Issues

#### 1. Gateway returns 401 Unauthorized
- **Cause**: Missing or invalid JWT token
- **Solution**: Ensure you're sending a valid Bearer token from Keycloak

#### 2. Service Unavailable responses
- **Cause**: Target microservices are down or circuit breaker is open
- **Solution**: 
  - Check if microservices are running
  - Check circuit breaker status at `/actuator/circuitbreakers`
  - Wait for circuit breaker to close

#### 3. Keycloak connection issues
- **Cause**: Keycloak server not accessible or authentication failed
- **Solution**:
  - Verify Keycloak is running: `docker compose ps`
  - Check Keycloak logs: `docker compose logs keycloak`
  - Ensure realm `spring-looks-security` exists
  - Verify admin credentials in `.env` file are correct

#### 4. Keycloak admin login fails
- **Cause**: Incorrect admin credentials
- **Solution**:
  - Check your `.env` file for correct `KEYCLOAK_ADMIN` and `KEYCLOAK_ADMIN_PASSWORD`
  - Restart containers after changing credentials: `docker compose down && docker compose up -d`
  - Wait for Keycloak to fully initialize before attempting login

#### 5. Port conflicts
- **Cause**: Port 9000 already in use
- **Solution**: Change server port in `application.properties` or stop conflicting process

### Logging

Enable debug logging by adding to `application.properties`:

```properties
logging.level.com.springlooks.gateway=DEBUG
logging.level.org.springframework.security=DEBUG
logging.level.org.springframework.cloud.gateway=DEBUG
```

## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/new-feature`
3. Commit your changes: `git commit -am 'Add new feature'`
4. Push to the branch: `git push origin feature/new-feature`
5. Submit a pull request

### Development Guidelines

- Follow Spring Boot best practices
- Add unit tests for new features
- Update documentation for configuration changes
- Ensure security configurations are not hardcoded
- Test circuit breaker scenarios

---

## Future Enhancements

This section outlines planned improvements and features for the Spring Looks API Gateway and microservices platform.

### Infrastructure & Deployment
- [ ] **Azure Cloud Deployment**
  - Deploy microservices to Azure Container Apps
  - Set up Azure Database for MySQL
  - Configure Azure Key Vault for secrets management
  - Implement Azure Application Gateway for external routing
  - see branch feat/deploy-scripts

### Authentication & Security
- [ ] **Migrate to Azure Entra ID**
  - Replace Keycloak with Azure Entra ID for authentication
  - Configure App Registrations for microservices
  - Implement Azure AD OAuth2 flows
  - Reduce infrastructure overhead and costs


### 🏗️ Architecture Improvements
- [ ] **Service Mesh Integration**
  - Investigate Istio or Linkerd for service-to-service communication
  - Implement advanced traffic management
  - Add mutual TLS between services


---

**Last Updated**: March 2026  
**Version**: 0.0.1-SNAPSHOT  
**Java Version**: 21+  
**Spring Boot Version**: 3.5.0
