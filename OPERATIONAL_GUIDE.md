# DispatchIQ - Operational Guide

## Smart Rider Assignment & ETA Engine for Food Delivery

---

## Table of Contents

1. [Overview](#overview)
2. [System Requirements](#system-requirements)
3. [Starting the Application](#starting-the-application)
4. [Accessing the Application](#accessing-the-application)
5. [Core Concepts](#core-concepts)
6. [Step-by-Step Operations](#step-by-step-operations)
7. [API Reference](#api-reference)
8. [Troubleshooting](#troubleshooting)

---

## Overview

DispatchIQ is a smart logistics platform that automates rider assignment and ETA calculation for food delivery operations. The system uses intelligent algorithms to:

- **Automatically assign** the nearest available rider to orders
- **Calculate accurate ETAs** based on distance and traffic conditions
- **Track order lifecycle** from creation to delivery
- **Generate events** for real-time monitoring

### Architecture

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   Frontend      │────▶│   Backend API   │────▶│   H2 Database   │
│   (React)       │     │  (Spring Boot)  │     │   (In-Memory)   │
│   Port: 5173    │     │   Port: 8080    │     │                 │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

---

## System Requirements

| Component | Requirement |
|-----------|-------------|
| Java | JDK 8 or higher |
| Maven | 3.6+ |
| Node.js | 16+ (for frontend) |
| Browser | Chrome, Firefox, Edge |

---

## Starting the Application

### Step 1: Start the Backend

Open a terminal in the project directory and run:

```bash
cd dispatchiq-backend
mvn spring-boot:run -s settings.xml
```

Wait for the message: `Started DispatchIqApplication`

<!-- Screenshot: Backend startup console -->
> **[Add Screenshot Here]**: Terminal showing successful backend startup with Spring Boot banner

### Step 2: Start the Frontend (Optional)

Open another terminal and run:

```bash
cd dispatchiq-frontend
npm install
npm run dev
```

<!-- Screenshot: Frontend startup -->
> **[Add Screenshot Here]**: Terminal showing Vite dev server started on port 5173

---

## Accessing the Application

| Resource | URL | Description |
|----------|-----|-------------|
| **Swagger UI** | http://localhost:8080/swagger-ui.html | Interactive API documentation |
| **Frontend Dashboard** | http://localhost:5173 | Visual dashboard (if running) |
| **H2 Database Console** | http://localhost:8080/h2-console | Database browser |
| **Health Check** | http://localhost:8080/actuator/health | System health status |

### Swagger UI

<!-- Screenshot: Swagger UI main page -->
> **[Add Screenshot Here]**: Swagger UI showing all available API endpoints

### H2 Console Login

Use these credentials:
- **JDBC URL**: `jdbc:h2:mem:dispatchiq`
- **Username**: `sa`
- **Password**: *(leave empty)*

<!-- Screenshot: H2 Console login -->
> **[Add Screenshot Here]**: H2 Console login page with credentials filled

---

## Core Concepts

### Order Status Lifecycle

```
CREATED → RESTAURANT_ACCEPTED → PREPARING → READY_FOR_PICKUP → RIDER_ASSIGNED → PICKED_UP → OUT_FOR_DELIVERY → DELIVERED
                                                                      ↑
                                                          (Auto-assigned by system)
```

### Rider Status

| Status | Description |
|--------|-------------|
| `AVAILABLE` | Ready to accept orders |
| `ON_DELIVERY` | Currently delivering an order |
| `ON_BREAK` | Temporarily unavailable |
| `OFFLINE` | Not working |

### Vehicle Types

- `BICYCLE`
- `SCOOTER`
- `MOTORCYCLE`
- `CAR`

---

## Step-by-Step Operations

### Operation 1: Create a Restaurant

**Endpoint**: `POST /api/restaurants`

**Request Body**:
```json
{
  "name": "Pizza Palace",
  "latitude": 12.9716,
  "longitude": 77.5946,
  "averagePreparationTimeMinutes": 20,
  "isActive": true
}
```

**Steps in Swagger UI**:
1. Navigate to **restaurant-controller** section
2. Click **POST /api/restaurants**
3. Click **"Try it out"** button
4. Paste the JSON body
5. Click **"Execute"**

<!-- Screenshot: Creating restaurant in Swagger -->
> **[Add Screenshot Here]**: Swagger UI showing POST /api/restaurants with request body

**Expected Response**:
```json
{
  "id": 1,
  "name": "Pizza Palace",
  "latitude": 12.9716,
  "longitude": 77.5946,
  "averagePreparationTimeMinutes": 20,
  "isActive": true,
  "currentActiveOrders": 0,
  "createdAt": "2026-06-21T12:55:01.540"
}
```

---

### Operation 2: Create a Customer

**Endpoint**: `POST /api/customers`

**Request Body**:
```json
{
  "name": "John Doe",
  "address": "123 Main Street, Bangalore",
  "phone": "9876543210",
  "latitude": 12.9800,
  "longitude": 77.6000
}
```

<!-- Screenshot: Creating customer -->
> **[Add Screenshot Here]**: Swagger UI showing POST /api/customers with response

---

### Operation 3: Create a Rider

**Endpoint**: `POST /api/riders`

**Request Body**:
```json
{
  "name": "Rider One",
  "phone": "9123456789",
  "vehicleType": "MOTORCYCLE",
  "currentLatitude": 12.9720,
  "currentLongitude": 77.5950,
  "isOnline": true
}
```

**Important**: Rider will be created in `OFFLINE` status. You need to set them online.

<!-- Screenshot: Creating rider -->
> **[Add Screenshot Here]**: Swagger UI showing POST /api/riders

---

### Operation 4: Set Rider Online

**Endpoint**: `POST /api/riders/{id}/go-online`

This changes the rider status to `AVAILABLE` so they can receive orders.

<!-- Screenshot: Setting rider online -->
> **[Add Screenshot Here]**: Swagger UI showing go-online endpoint

---

### Operation 5: Create an Order

**Endpoint**: `POST /api/orders`

**Request Body**:
```json
{
  "restaurantId": 1,
  "customerId": 1,
  "orderValue": 350.00,
  "preparationTimeMinutes": 15
}
```

<!-- Screenshot: Creating order -->
> **[Add Screenshot Here]**: Swagger UI showing POST /api/orders with response

---

### Operation 6: Complete Order Workflow

Execute these endpoints in sequence:

| Step | Endpoint | Description |
|------|----------|-------------|
| 1 | `POST /api/orders/{id}/accept` | Restaurant accepts order |
| 2 | `POST /api/orders/{id}/prepare` | Start food preparation |
| 3 | `POST /api/orders/{id}/ready` | Food ready (rider auto-assigned!) |
| 4 | `POST /api/assignments/order/{orderId}/rider/{riderId}/accept` | Rider accepts |
| 5 | `POST /api/orders/{id}/pickup` | Rider picks up |
| 6 | `POST /api/orders/{id}/out-for-delivery` | Out for delivery |
| 7 | `POST /api/orders/{id}/deliver` | Order delivered |

<!-- Screenshot: Order workflow -->
> **[Add Screenshot Here]**: Swagger UI showing order workflow endpoints

---

## Complete Workflow Example

### Scenario: Process a Pizza Delivery

```
1. Restaurant "Pizza Palace" (ID: 1) is registered
2. Customer "John Doe" (ID: 1) places order worth ₹350
3. Rider "Rider One" (ID: 1) is online and available
4. System creates order and notifies restaurant
5. Restaurant accepts → prepares → marks ready
6. System auto-assigns nearest rider (0.06 km away)
7. Rider accepts → picks up → delivers
8. Order complete!
```

### Timeline View

```
12:58:46 - Order Created (CREATED)
12:59:02 - Restaurant Accepts (RESTAURANT_ACCEPTED)
12:59:05 - Preparation Started (PREPARING)
12:59:09 - Food Ready + Rider Assigned (RIDER_ASSIGNED)
12:59:33 - Rider Accepts Assignment (ACCEPTED)
12:59:33 - Order Picked Up (PICKED_UP)
12:59:33 - Out for Delivery (OUT_FOR_DELIVERY)
12:59:33 - Delivered (DELIVERED)
```

<!-- Screenshot: Order details showing complete workflow -->
> **[Add Screenshot Here]**: Order response showing DELIVERED status with all timestamps

---

## API Reference

### Restaurants

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/restaurants` | Create restaurant |
| GET | `/api/restaurants` | List all restaurants |
| GET | `/api/restaurants/{id}` | Get restaurant by ID |
| PUT | `/api/restaurants/{id}` | Update restaurant |
| DELETE | `/api/restaurants/{id}` | Delete restaurant |

### Customers

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/customers` | Create customer |
| GET | `/api/customers` | List all customers |
| GET | `/api/customers/{id}` | Get customer by ID |
| PUT | `/api/customers/{id}` | Update customer |
| DELETE | `/api/customers/{id}` | Delete customer |

### Riders

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/riders` | Create rider |
| GET | `/api/riders` | List all riders |
| GET | `/api/riders/{id}` | Get rider by ID |
| GET | `/api/riders/available` | List available riders |
| GET | `/api/riders/online` | List online riders |
| POST | `/api/riders/{id}/go-online` | Set rider online |
| POST | `/api/riders/{id}/go-offline` | Set rider offline |
| PATCH | `/api/riders/{id}/location` | Update rider location |

### Orders

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/orders` | Create order |
| GET | `/api/orders` | List all orders |
| GET | `/api/orders/{id}` | Get order by ID |
| GET | `/api/orders/active` | List active orders |
| POST | `/api/orders/{id}/accept` | Restaurant accepts |
| POST | `/api/orders/{id}/prepare` | Start preparing |
| POST | `/api/orders/{id}/ready` | Mark ready for pickup |
| POST | `/api/orders/{id}/pickup` | Mark picked up |
| POST | `/api/orders/{id}/out-for-delivery` | Mark out for delivery |
| POST | `/api/orders/{id}/deliver` | Mark delivered |
| POST | `/api/orders/{id}/cancel` | Cancel order |

### Assignments

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/assignments/order/{orderId}/assign` | Assign rider to order |
| POST | `/api/assignments/order/{orderId}/rider/{riderId}/accept` | Rider accepts |
| POST | `/api/assignments/order/{orderId}/rider/{riderId}/reject` | Rider rejects |
| GET | `/api/assignments/order/{orderId}` | Get assignments for order |

---

## Troubleshooting

### Issue: Port 8080 Already in Use

**Solution**: Stop the existing process
```powershell
Get-NetTCPConnection -LocalPort 8080 | ForEach-Object { Stop-Process -Id $_.OwningProcess -Force }
```

### Issue: 404 on Root URL (localhost:8080)

**Explanation**: This is normal - the backend is a REST API with no root page.

**Solution**: Use `/swagger-ui.html` or specific API endpoints.

### Issue: "No eligible riders available"

**Possible Causes**:
1. No riders created
2. Riders are offline
3. Riders are too far from restaurant (> 5km default)
4. Riders have max active orders

**Solution**: Create a rider, set them online, and ensure they're near the restaurant.

### Issue: Vehicle Type Error

**Valid Values**: `BICYCLE`, `SCOOTER`, `MOTORCYCLE`, `CAR`

**Note**: "BIKE" is not valid - use `MOTORCYCLE` or `BICYCLE`

### Issue: Customer Creation Fails

**Required Fields**:
- `name` (string)
- `address` (string) - This is often missed!
- `latitude` (number)
- `longitude` (number)

---

## Quick Reference Card

### Minimum Setup to Test

```bash
# 1. Create Restaurant
POST /api/restaurants
{"name":"Test Restaurant","latitude":12.97,"longitude":77.59,"averagePreparationTimeMinutes":15,"isActive":true}

# 2. Create Customer  
POST /api/customers
{"name":"Test Customer","address":"Test Address","latitude":12.98,"longitude":77.60}

# 3. Create & Activate Rider
POST /api/riders
{"name":"Test Rider","vehicleType":"MOTORCYCLE","currentLatitude":12.97,"currentLongitude":77.59}
POST /api/riders/1/go-online

# 4. Create Order
POST /api/orders
{"restaurantId":1,"customerId":1,"orderValue":100}

# 5. Process Order
POST /api/orders/1/accept
POST /api/orders/1/prepare
POST /api/orders/1/ready
POST /api/assignments/order/1/rider/1/accept
POST /api/orders/1/pickup
POST /api/orders/1/out-for-delivery
POST /api/orders/1/deliver
```

---

## Adding Screenshots

To add screenshots to this guide:

1. Take screenshot using `Win + Shift + S` (Windows) or `Cmd + Shift + 4` (Mac)
2. Save images in `docs/screenshots/` folder
3. Replace placeholder text with markdown image syntax:
   ```markdown
   ![Description](docs/screenshots/your-image.png)
   ```

### Recommended Screenshots

| Section | Screenshot Name | Description |
|---------|-----------------|-------------|
| Startup | `backend-startup.png` | Spring Boot startup console |
| Swagger | `swagger-main.png` | Swagger UI main page |
| Create Restaurant | `create-restaurant.png` | POST restaurant with response |
| Create Customer | `create-customer.png` | POST customer with response |
| Create Rider | `create-rider.png` | POST rider with response |
| Create Order | `create-order.png` | POST order with response |
| Order Delivered | `order-delivered.png` | Final order status |
| H2 Console | `h2-console.png` | Database tables view |

---

## Support

For issues or questions, refer to:
- **Swagger UI**: http://localhost:8080/swagger-ui.html
- **README.md**: Project documentation
- **Application Logs**: Check terminal output for errors

---

*Document Version: 1.0*  
*Last Updated: June 2026*
