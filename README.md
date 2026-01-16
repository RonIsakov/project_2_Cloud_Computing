# Pet Store Microservices Platform

A distributed pet store application built with microservices architecture, featuring containerized services orchestrated with Docker Compose and load-balanced through Nginx reverse proxy.

## Authors

- Ron Isakov
- Noam Kyram

## Architecture Overview

The system consists of the following components:

```
                    +----------------+
                    |    Nginx       |
                    | Reverse Proxy  |
                    |   (Port 80)    |
                    +-------+--------+
                            |
          +-----------------+-----------------+
          |                                   |
          v                                   v
+-------------------+               +-------------------+
|   Pet Store 1     |               |   Pet Store 2     |
|   (Port 5001)     |               |   (Port 5002)     |
+--------+----------+               +--------+----------+
         |                                   |
         +----------------+------------------+
                          |
                          v
               +--------------------+
               | MongoDB (Stores)   |
               |   (Port 27018)     |
               +--------------------+

          +-----------------+-----------------+
          |                                   |
          v                                   v
+-------------------+               +-------------------+
|   Pet Order 1     |               |   Pet Order 2     |
|   (Port 5003)     |               | (Internal Only)   |
+--------+----------+               +--------+----------+
         |                                   |
         +----------------+------------------+
                          |
                          v
               +----------------------+
               | MongoDB (Transactions)|
               |    (Port 27019)       |
               +----------------------+
```

## Services

### Pet Store Service
Flask-based REST API managing pet types and individual pets. Integrates with the Ninja Animals API for pet type information.

**Features:**
- CRUD operations for pet types and pets
- Image storage for pet pictures
- Integration with external Ninja Animals API
- Separate collections per store instance

### Pet Order Service
Flask-based REST API handling pet purchases and transaction management.

**Features:**
- Cross-store pet purchasing
- Transaction logging and retrieval
- Owner authentication for transaction queries
- Weighted load balancing (3:1 ratio between instances)

### Nginx Reverse Proxy
Entry point for external traffic with URL-based routing and HTTP method restrictions.

**Routing:**
- `/pet-types1` - Routes to Pet Store 1 (GET only)
- `/pet-types2` - Routes to Pet Store 2 (GET only)
- `/purchases` - Routes to Pet Order service (POST only)

### MongoDB Databases
- **mongodb-stores** - Stores pet types and pets data
- **mongodb-transactions** - Stores purchase transactions

## Technology Stack

| Component | Technology |
|-----------|------------|
| Backend | Python 3, Flask |
| Database | MongoDB 7.0 |
| Reverse Proxy | Nginx Alpine |
| Containerization | Docker |
| Orchestration | Docker Compose |
| External API | Ninja Animals API |

## Prerequisites

- Docker Engine 20.10+
- Docker Compose v2.0+

## Installation and Deployment

1. Clone the repository:
```bash
git clone <repository-url>
cd project_2
```

2. Build and start all services:
```bash
docker-compose up --build
```

3. To run in detached mode:
```bash
docker-compose up -d --build
```

4. To stop all services:
```bash
docker-compose down
```

5. To stop and remove volumes:
```bash
docker-compose down -v
```

## API Endpoints

### Pet Store Service

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/pet-types` | Create a new pet type |
| GET | `/pet-types` | List all pet types (supports filtering) |
| GET | `/pet-types/{id}` | Get a specific pet type |
| DELETE | `/pet-types/{id}` | Delete a pet type |
| POST | `/pet-types/{id}/pets` | Add a pet to a pet type |
| GET | `/pet-types/{id}/pets` | List pets of a pet type |
| GET | `/pet-types/{id}/pets/{name}` | Get a specific pet |
| PUT | `/pet-types/{id}/pets/{name}` | Update a pet |
| DELETE | `/pet-types/{id}/pets/{name}` | Delete a pet |
| GET | `/pictures/{filename}` | Retrieve a pet picture |

### Pet Order Service

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/purchases` | Create a new purchase |
| GET | `/transactions` | List transactions (requires OwnerPC header) |

### Via Nginx Reverse Proxy (Port 80)

| Method | Endpoint | Routes To |
|--------|----------|-----------|
| GET | `/pet-types1` | Pet Store 1 |
| GET | `/pet-types2` | Pet Store 2 |
| POST | `/purchases` | Pet Order Service (load balanced) |

## Configuration

### Environment Variables

**Pet Store Service:**
- `MONGO_URI` - MongoDB connection string
- `STORE_ID` - Store instance identifier (1 or 2)
- `OWNER_PASSWORD` - Password for owner operations
- `NINJA_API_KEY` - API key for Ninja Animals API

**Pet Order Service:**
- `MONGO_URI` - MongoDB connection string
- `OWNER_PASSWORD` - Password for transaction queries

## Port Mappings

| Service | Internal Port | External Port |
|---------|--------------|---------------|
| Nginx | 80 | 80 |
| Pet Store 1 | 8000 | 5001 |
| Pet Store 2 | 8000 | 5002 |
| Pet Order 1 | 8080 | 5003 |
| Pet Order 2 | 8080 | - |
| MongoDB Stores | 27017 | 27018 |
| MongoDB Transactions | 27017 | 27019 |

## Data Persistence

- Pet images are stored in Docker volumes (`pet-images-store1`, `pet-images-store2`)
- MongoDB data persists within containers (add volumes for production persistence)
- All services are configured with `restart: always` for automatic recovery

## Load Balancing

The Nginx reverse proxy implements weighted load balancing for the Pet Order service:
- `pet-order1`: weight 3 (75% of traffic)
- `pet-order2`: weight 1 (25% of traffic)

## Network

All services communicate over a custom bridge network (`petstore-network`), providing:
- Service discovery by container name
- Isolated network namespace
- Internal communication without port exposure

## License

This project was developed as part of a cloud computing course assignment.
