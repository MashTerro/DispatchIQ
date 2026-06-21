# DispatchIQ - Smart Rider Assignment & ETA Engine

A production-quality full-stack logistics platform that simulates how food delivery companies automatically assign delivery riders to customer orders while minimizing delivery time and maximizing rider utilization.

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Getting Started](#getting-started)
- [API Documentation](#api-documentation)
- [Assignment Algorithm](#assignment-algorithm)
- [ETA Calculation](#eta-calculation)
- [Database Schema](#database-schema)
- [Project Structure](#project-structure)
- [Testing](#testing)
- [Docker Deployment](#docker-deployment)
- [Future Improvements](#future-improvements)

## Overview

DispatchIQ is a backend-focused logistics platform demonstrating:

- **REST APIs** with comprehensive CRUD operations
- **Event-Driven Architecture** for real-time tracking
- **Smart Assignment Engine** with multi-criteria rider selection
- **ETA Calculation** using Haversine distance formula
- **Clean Architecture** with separation of concerns
- **Automated Testing** with JUnit 5 and Mockito

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         FRONTEND                                 │
│                    React + TypeScript                            │
│              (Dashboard, Orders, Riders, etc.)                   │
└─────────────────────────────┬───────────────────────────────────┘
                              │ REST API
┌─────────────────────────────▼───────────────────────────────────┐
│                      API GATEWAY / CONTROLLERS                   │
│     RestaurantController  │  OrderController  │  RiderController │
│     CustomerController    │  DashboardController  │  etc.        │
└─────────────────────────────┬───────────────────────────────────┘
                              │
┌─────────────────────────────▼───────────────────────────────────┐
│                      SERVICE LAYER                               │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐  │
│  │   Order     │  │   Rider     │  │   Assignment Service    │  │
│  │   Service   │  │   Service   │  │   (Orchestration)       │  │
│  └─────────────┘  └─────────────┘  └─────────────────────────┘  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐  │
│  │  Restaurant │  │  Customer   │  │   Dashboard Service     │  │
│  │   Service   │  │   Service   │  │   (KPIs & Analytics)    │  │
│  └─────────────┘  └─────────────┘  └─────────────────────────┘  │
└─────────────────────────────┬───────────────────────────────────┘
                              │
┌─────────────────────────────▼───────────────────────────────────┐
│                      DISPATCH ENGINE                             │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │              SMART RIDER ASSIGNMENT                      │    │
│  │  ┌─────────────────┐    ┌──────────────────────────┐    │    │
│  │  │  Stage 1:       │ -> │  Stage 2:                │    │    │
│  │  │  Filter         │    │  Rank & Select           │    │    │
│  │  │  Eligible       │    │  Best Rider              │    │    │
│  │  │  Riders         │    │                          │    │    │
│  │  └─────────────────┘    └──────────────────────────┘    │    │
│  └─────────────────────────────────────────────────────────┘    │
│  ┌─────────────────────┐  ┌────────────────────────────────┐    │
│  │  Distance Calculator│  │  ETA Calculator                │    │
│  │  (Haversine)        │  │  (Prep + Travel Time)          │    │
│  └─────────────────────┘  └────────────────────────────────┘    │
└─────────────────────────────┬───────────────────────────────────┘
                              │
┌─────────────────────────────▼───────────────────────────────────┐
│                      SCHEDULERS                                  │
│  ┌─────────────────────────┐  ┌─────────────────────────────┐   │
│  │ Queue Processor         │  │ Assignment Timeout Checker  │   │
│  │ (Every 30 seconds)      │  │ (Every 30 seconds)          │   │
│  └─────────────────────────┘  └─────────────────────────────┘   │
└─────────────────────────────┬───────────────────────────────────┘
                              │
┌─────────────────────────────▼───────────────────────────────────┐
│                      EVENT SERVICE                               │
│              Publishes events for all operations                 │
│    OrderCreated, RiderAssigned, OrderDelivered, etc.            │
└─────────────────────────────┬───────────────────────────────────┘
                              │
┌─────────────────────────────▼───────────────────────────────────┐
│                      DATA LAYER                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                    JPA Repositories                      │    │
│  │  Restaurant │ Customer │ Rider │ Order │ Event │ Queue   │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────┬───────────────────────────────────┘
                              │
┌─────────────────────────────▼───────────────────────────────────┐
│                      PostgreSQL Database                         │
└─────────────────────────────────────────────────────────────────┘
```

## Features

### Core Features

- **Order Management**: Full lifecycle from creation to delivery
- **Restaurant Management**: CRUD operations with location tracking
- **Customer Management**: Customer profiles with order history
- **Rider Management**: Real-time status and location tracking
- **Smart Assignment Engine**: Two-stage rider selection algorithm
- **ETA Calculation**: Dynamic delivery time estimation
- **Waiting Queue**: Automatic retry for unassigned orders
- **Event Tracking**: Complete audit trail of all operations

### Dashboard Features

- Real-time KPIs (Active Orders, Available Riders, etc.)
- Live orders table with status tracking
- Waiting queue visualization
- Event stream for monitoring

### Assignment Engine Features

- Distance-based filtering (configurable radius)
- Multi-criteria ranking (distance, rating, workload, idle time)
- Automatic reassignment on rider unavailability
- Cooldown period to prevent rapid reassignment

## Tech Stack

### Backend
- **Java 21** - Latest LTS version
- **Spring Boot 3.2** - Application framework
- **Spring Data JPA** - Data persistence
- **PostgreSQL** - Relational database
- **MapStruct** - DTO mapping
- **Lombok** - Boilerplate reduction
- **SpringDoc OpenAPI** - API documentation

### Frontend
- **React 18** - UI library
- **TypeScript** - Type safety
- **Tailwind CSS** - Utility-first styling
- **TanStack Query** - Data fetching
- **React Router** - Navigation
- **Recharts** - Data visualization
- **Lucide React** - Icons

### DevOps
- **Docker** - Containerization
- **Docker Compose** - Multi-container orchestration
- **JUnit 5** - Unit testing
- **Mockito** - Mocking framework

## Getting Started

### Prerequisites

- Java 21+
- Node.js 18+
- Docker & Docker Compose
- Maven 3.9+

### Quick Start with Docker

```bash
# Clone the repository
git clone <repository-url>
cd Rider_Management

# Start all services
docker-compose up -d

# Access the application
# Frontend: http://localhost:3000
# Backend API: http://localhost:8080
# Swagger UI: http://localhost:8080/swagger-ui.html
```

### Manual Setup

#### Backend

```bash
cd dispatchiq-backend

# Start PostgreSQL (or use Docker)
docker run -d --name postgres -e POSTGRES_DB=dispatchiq \
  -e POSTGRES_USER=dispatchiq -e POSTGRES_PASSWORD=dispatchiq123 \
  -p 5432:5432 postgres:16-alpine

# Build and run
mvn clean install
mvn spring-boot:run
```

#### Frontend

```bash
cd dispatchiq-frontend

# Install dependencies
npm install

# Start development server
npm run dev
```

### Generate Test Data

Use the simulation endpoint to generate test data:

```bash
curl -X POST http://localhost:8080/api/simulation/full-setup
```

Or click "Generate Test Data" button in the sidebar.

## API Documentation

### REST Endpoints

#### Restaurants
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | /api/restaurants | List all restaurants |
| GET | /api/restaurants/{id} | Get restaurant by ID |
| POST | /api/restaurants | Create restaurant |
| PUT | /api/restaurants/{id} | Update restaurant |
| DELETE | /api/restaurants/{id} | Delete restaurant |

#### Customers
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | /api/customers | List all customers |
| GET | /api/customers/{id} | Get customer by ID |
| POST | /api/customers | Create customer |
| PUT | /api/customers/{id} | Update customer |
| DELETE | /api/customers/{id} | Delete customer |

#### Riders
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | /api/riders | List all riders |
| GET | /api/riders/available | List available riders |
| POST | /api/riders | Create rider |
| PATCH | /api/riders/{id}/location | Update location |
| PATCH | /api/riders/{id}/status | Update status |
| POST | /api/riders/{id}/go-online | Set online |
| POST | /api/riders/{id}/go-offline | Set offline |

#### Orders
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | /api/orders | List all orders |
| GET | /api/orders/active | List active orders |
| POST | /api/orders | Create order |
| POST | /api/orders/{id}/accept | Restaurant accepts |
| POST | /api/orders/{id}/prepare | Start preparation |
| POST | /api/orders/{id}/ready | Mark ready |
| POST | /api/orders/{id}/pickup | Mark picked up |
| POST | /api/orders/{id}/deliver | Mark delivered |
| POST | /api/orders/{id}/cancel | Cancel order |

#### Dashboard
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | /api/dashboard/kpis | Get KPIs |
| GET | /api/dashboard/waiting-queue | Get waiting queue |
| GET | /api/dashboard/events/recent | Get recent events |

## Assignment Algorithm

The Dispatch Engine uses a two-stage algorithm:

### Stage 1: Filter Eligible Riders

A rider is eligible if ALL conditions are met:
- ✅ Status = AVAILABLE
- ✅ Is Online = true
- ✅ Distance to Restaurant ≤ Max Radius (default: 5 km)
- ✅ Active Deliveries < Max Capacity (default: 2)
- ✅ Not assigned within cooldown period (default: 60 seconds)

### Stage 2: Rank Eligible Riders

Riders are ranked by (in priority order):
1. **Distance** - Nearest to restaurant (ascending)
2. **Rating** - Highest rating (descending)
3. **Workload** - Lowest active orders (ascending)
4. **Idle Time** - Longest waiting (descending)
5. **ID** - Lowest ID (tiebreaker for determinism)

```java
// Ranking comparator
Comparator
    .comparingDouble(RiderCandidate::getDistanceToRestaurantKm)
    .thenComparing(Comparator.comparingDouble(RiderCandidate::getRating).reversed())
    .thenComparingInt(RiderCandidate::getActiveOrders)
    .thenComparing(Comparator.comparingLong(RiderCandidate::getIdleTimeMinutes).reversed())
    .thenComparing(c -> c.getRider().getId())
```

### No Rider Available Flow

When no eligible rider exists:
1. Order enters Waiting Queue
2. `NoRiderAvailable` event published
3. Queue processor retries every 30 seconds
4. Assignment continues until rider available

## ETA Calculation

### Formula

```
ETA = Preparation Time + Travel to Restaurant + Travel to Customer
```

### Distance Calculation (Haversine Formula)

```java
double dLat = Math.toRadians(lat2 - lat1);
double dLon = Math.toRadians(lon2 - lon1);

double a = Math.sin(dLat / 2) * Math.sin(dLat / 2) +
           Math.cos(Math.toRadians(lat1)) * Math.cos(Math.toRadians(lat2)) *
           Math.sin(dLon / 2) * Math.sin(dLon / 2);

double c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
double distance = EARTH_RADIUS_KM * c; // 6371 km
```

### Travel Time Calculation

```java
// Default rider speed: 30 km/h
double hours = distanceKm / speedKmh;
int minutes = (int) Math.ceil(hours * 60);
```

## Database Schema

### Entity Relationship Diagram

```
┌──────────────┐       ┌──────────────┐       ┌──────────────┐
│  RESTAURANT  │       │   CUSTOMER   │       │    RIDER     │
├──────────────┤       ├──────────────┤       ├──────────────┤
│ id           │       │ id           │       │ id           │
│ name         │       │ name         │       │ name         │
│ latitude     │       │ address      │       │ latitude     │
│ longitude    │       │ latitude     │       │ longitude    │
│ prep_time    │       │ longitude    │       │ status       │
│ cuisine      │       │ phone        │       │ vehicle_type │
│ active_orders│       │ email        │       │ rating       │
└──────┬───────┘       └──────┬───────┘       │ active_orders│
       │                      │               └──────┬───────┘
       │                      │                      │
       └──────────┬───────────┘                      │
                  │                                  │
           ┌──────▼───────┐                          │
           │DELIVERY_ORDER│◄─────────────────────────┘
           ├──────────────┤
           │ id           │
           │ restaurant_id│       ┌──────────────┐
           │ customer_id  │       │  ASSIGNMENT  │
           │ rider_id     │◄──────┤──────────────┤
           │ status       │       │ id           │
           │ order_value  │       │ order_id     │
           │ eta_minutes  │       │ rider_id     │
           │ distances    │       │ status       │
           └──────┬───────┘       │ distance     │
                  │               └──────────────┘
                  │
           ┌──────▼───────┐       ┌──────────────┐
           │DISPATCH_EVENT│       │WAITING_QUEUE │
           ├──────────────┤       ├──────────────┤
           │ id           │       │ id           │
           │ event_type   │       │ order_id     │
           │ order_id     │       │ position     │
           │ rider_id     │       │ retry_count  │
           │ metadata     │       │ entered_at   │
           │ timestamp    │       └──────────────┘
           └──────────────┘
```

## Project Structure

```
Rider_Management/
├── dispatchiq-backend/
│   ├── src/main/java/com/dispatchiq/
│   │   ├── config/          # Configuration classes
│   │   ├── controller/      # REST controllers
│   │   ├── dto/             # Data transfer objects
│   │   ├── engine/          # Dispatch & ETA engines
│   │   ├── entity/          # JPA entities
│   │   ├── exception/       # Custom exceptions
│   │   ├── mapper/          # MapStruct mappers
│   │   ├── repository/      # JPA repositories
│   │   ├── scheduler/       # Scheduled tasks
│   │   └── service/         # Business logic
│   ├── src/main/resources/
│   │   └── application.yml
│   ├── src/test/            # Unit tests
│   ├── Dockerfile
│   └── pom.xml
│
├── dispatchiq-frontend/
│   ├── src/
│   │   ├── components/      # Reusable components
│   │   ├── pages/           # Page components
│   │   ├── services/        # API services
│   │   └── types/           # TypeScript types
│   ├── Dockerfile
│   └── package.json
│
├── docker-compose.yml
└── README.md
```

## Testing

### Running Tests

```bash
cd dispatchiq-backend

# Run all tests
mvn test

# Run specific test class
mvn test -Dtest=DispatchEngineTest

# Run with coverage
mvn test jacoco:report
```

### Test Coverage

- **DistanceCalculatorTest** - Haversine formula validation
- **DispatchEngineTest** - Assignment algorithm tests
- **OrderServiceTest** - Order lifecycle tests
- **RiderServiceTest** - Rider status management tests

## Docker Deployment

### Build Images

```bash
# Build all images
docker-compose build

# Build specific service
docker-compose build backend
```

### Run Services

```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f backend

# Stop services
docker-compose down

# Stop and remove volumes
docker-compose down -v
```

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| SPRING_DATASOURCE_URL | jdbc:postgresql://localhost:5432/dispatchiq | Database URL |
| SPRING_DATASOURCE_USERNAME | dispatchiq | Database user |
| SPRING_DATASOURCE_PASSWORD | dispatchiq123 | Database password |

## Future Improvements

- [ ] WebSocket for real-time updates
- [ ] Redis caching for performance
- [ ] Kafka for event streaming
- [ ] Kubernetes deployment configs
- [ ] Role-based authentication (JWT)
- [ ] GraphQL API
- [ ] Mobile app (React Native)
- [ ] Traffic-aware ETA calculation
- [ ] Machine learning for demand prediction
- [ ] Multi-region support
- [ ] Rate limiting
- [ ] Circuit breaker patterns

## License

MIT License - See LICENSE file for details

---

Built with dedication for demonstrating production-quality backend engineering practices.
