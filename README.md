# Spring Looks API Gateway

A secure API Gateway for the Spring Looks e-commerce application, providing centralized routing, authentication, and monitoring for microservices.

The Spring Looks Microservices application was deployed to Azure using docker, container apps, a virtual network with subnets.

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

## Project Description

The Spring Looks API Gateway serves as the central entry point for all client requests to the Spring Looks e-commerce platform. It provides:

- **Request Routing**: Routes requests to appropriate microservices (Product, Order, Inventory)
- **Security**: JWT-based authentication using Microsoft Azure Entra ID
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

- ✅ **JWT Authentication** with Azure Entra ID integration
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
- **Microsoft Azure account** with Azure Entra ID (formerly Azure Active Directory) configured

## Project Setup

### Security First: Configure Azure Entra ID Credentials

This project uses Microsoft Azure Entra ID for authentication. Before running the application, you must set up an Azure Entra ID account and register an application in the Azure portal to obtain the required credentials.

Once your Azure Entra ID App Registration is in place, create a `.env.dev` file in the project root to securely configure the required credentials:

```bash
# Create .env.dev file (add to .gitignore)
cat > .env.dev << EOF
AZURE_CLIENT_ID=your-azure-app-client-id
AZURE_TENANT_ID=your-azure-tenant-id
AZURE_APP_ID_URI=api://your-azure-app-client-id
EOF
```

| Variable | Description |
|----------|-------------|
| `AZURE_CLIENT_ID` | The Application (client) ID of your Azure Entra ID App Registration |
| `AZURE_TENANT_ID` | The Directory (tenant) ID of your Azure Entra ID tenant |
| `AZURE_APP_ID_URI` | The Application ID URI configured for the resource server (e.g. `api://<client-id>`) |

> **Note**: Ensure `.env.dev` is added to your `.gitignore` file to prevent credentials from being committed to version control.

### 1. Start the API Gateway

```bash
# Start the gateway container
docker compose up -d

# Or run locally using Maven wrapper
./mvnw spring-boot:run
```

### 2. Start Dependent Microservices

Ensure the following services are running on their respective ports:
- Product Service: `http://localhost:8080`
- Order Service: `http://localhost:8081`
- Inventory Service: `http://localhost:8082`

## Configuration

### Environment Variables

You can override default configurations using environment variables or by updating your `.env.dev` file:

```bash
# Application Configuration
export SERVER_PORT=9000
export PRODUCT_SERVICE_URL=http://localhost:8080
export ORDER_SERVICE_URL=http://localhost:8081
export INVENTORY_SERVICE_URL=http://localhost:8082

# Azure Entra ID Configuration
export AZURE_CLIENT_ID=your-azure-app-client-id
export AZURE_TENANT_ID=your-azure-tenant-id
export AZURE_APP_ID_URI=api://your-azure-app-client-id
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

1. Client obtains a JWT access token from Azure Entra ID (via OAuth2 / OIDC)
2. Client includes token in the Authorization header: `Bearer <token>`
3. Gateway validates the token against Azure Entra ID
4. Valid requests are forwarded to the target services

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
| Spring Cloud Azure | Latest | Azure Entra ID / AAD integration |
| Resilience4j | 3.5.0 | Circuit breaker pattern |
| SpringDoc OpenAPI | 2.8.8 | API documentation |
| Spring Boot Actuator | 3.5.0 | Health monitoring |

### Infrastructure Dependencies

| Service | Version | Purpose |
|---------|---------|---------|
| Azure Entra ID | - | Authentication & authorisation (cloud-hosted) |
| Docker | Latest | Containerization |
| Java | 21+ | Runtime environment |

