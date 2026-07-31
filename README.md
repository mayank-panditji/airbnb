# 🏡 AIrbnb - Microservices Platform

[![Go Version](https://img.shields.io/badge/Go-1.20+-00ADD8?style=for-the-badge&logo=go&logoColor=white)](https://golang.org/)
[![Node.js](https://img.shields.io/badge/Node.js-18+-339933?style=for-the-badge&logo=node.js&logoColor=white)](https://nodejs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0+-3178C6?style=for-the-badge&logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![Redis](https://img.shields.io/badge/Redis-BullMQ-DC382D?style=for-the-badge&logo=redis&logoColor=white)](https://redis.io/)
[![Prisma](https://img.shields.io/badge/Prisma-ORM-2D3748?style=for-the-badge&logo=prisma&logoColor=white)](https://www.prisma.io/)

A distributed, high-performance **Microservices-Based Airbnb Clone** backend architecture designed for scale, resilience, and asynchronous processing. Built using **Go (Golang)** and **Node.js (TypeScript)**, featuring asynchronous job processing via **BullMQ/Redis**, ORM persistence (**Prisma**, **Sequelize**, **GORM**), rate limiting, and centralized correlation ID logging.

---

## 📐 System Architecture

```text
                             +-----------------------+
                             |   Client / Frontend   |
                             +-----------+-----------+
                                         |
                                         v
                             +-----------------------+
                             |     API Gateway       |
                             |   (Planned / Proxy)   |
                             +-----------+-----------+
                                         |
        +------------------+-------------+-------------+------------------+
        |                  |                           |                  |
        v                  v                           v                  v
+---------------+  +---------------+           +---------------+  +---------------+
| Auth Service  |  | Hotel Service |           |Booking Service|  |Review Service |
|  (Go - 3001)  |  |  (TS - 3000)  |           |  (TS - 3003)  |  | (Go - 8081)   |
+-------+-------+  +-------+-------+           +-------+-------+  +-------+-------+
        |                  |                           |                  |
        v                  v                           v                  v
+---------------+  +---------------+           +---------------+  +---------------+
|  Auth MySQL / |  | PostgreSQL /  |           | PostgreSQL /  |  | Review MySQL  |
|  PostgreSQL   |  |   Sequelize   |           |    Prisma     |  |    / GORM     |
+---------------+  +-------+-------+           +---------------+  +---------------+
                           |
                           v (BullMQ Jobs)
                   +---------------+           +---------------+
                   | Redis Queue   | --------> | Notification  |
                   | & Workers     |           | Service (3002)|
                   +---------------+           +---------------+
```

---

## 🛠️ Microservices Overview

The platform consists of **5 decoupled microservices**, each responsible for a distinct business domain:

| Service | Stack | Port | Primary Responsibilities |
| :--- | :--- | :--- | :--- |
| **🔑 Auth Service** | **Go (Chi)** | `3001` | User signup/login, JWT generation & verification, role management (`users`, `admin`), rate-limiting, and request logging. |
| **🏨 Hotel Service** | **Node.js (TypeScript)** | `3000` | Hotel & room listings, room availability scheduler, async room generation workers powered by **BullMQ & Redis**. |
| **📅 Booking Service**| **Node.js (TypeScript)** | `3003` | Reservation & booking workflow, availability verification, **Prisma ORM** data layer. |
| **⭐ Review Service** | **Go (Chi)** | `8081` | Property & host reviews, ratings management, user feedback persistence. |
| **📧 Notification** | **Node.js (TypeScript)** | `3002` | Asynchronous email queue processor (**BullMQ + Nodemailer**), HTML email template rendering (welcome emails, booking confirmations). |

---

## ✨ Key Features & Technical Highlights

- **Polyglot Microservices Architecture**: High-throughput authentication and review modules written in **Go**, paired with rich domain logic microservices in **TypeScript**.
- **Asynchronous Task Queues**: Heavy background tasks (room batch generation, email delivery) offloaded to **BullMQ** workers backed by **Redis**.
- **Distributed Correlation Tracking**: Middleware attaches a unique `X-Correlation-ID` to every HTTP request across Node services for end-to-end tracing.
- **Multiple ORMs & DBs**: Modular persistence layers using **Prisma ORM**, **Sequelize**, and **GORM**.
- **Security & Authorization**: Role-based access control (RBAC), JWT authentication, request body validation using **Zod** and Go DTOs, and CORS headers.
- **Unified Orchestration**: Launch all services simultaneously with a single command via `./run-all.sh`.

---

## 📁 Repository Structure

```text
AIrbnb/
└── backend/
    ├── authentication/       # Go service - JWT Auth, User & Role Management (Port 3001)
    ├── hotelservice/         # Node.js/TS service - Hotel & Room Management (Port 3000)
    │   └── HotelService/
    ├── bookingservice/       # Node.js/TS service - Booking Management (Port 3003)
    │   └── Bookingservice/
    ├── reviewservice/        # Go service - Ratings & Reviews (Port 8081)
    ├── Notificationservice/  # Node.js/TS service - BullMQ Email Worker (Port 3002)
    └── run-all.sh            # Orchestration script to run all backend services
```

---

## 🚀 Quick Start Guide

### Prerequisites

Ensure you have the following installed on your machine:
- **Go**: `1.20` or higher
- **Node.js**: `18.x` or higher & `npm` / `npx`
- **Redis**: Running locally or accessible via URL (`localhost:6379`)
- **PostgreSQL / MySQL**: Running locally or configured via `.env`

---

### 1️⃣ Clone & Navigate

```bash
git clone https://github.com/your-username/AIrbnb.git
cd AIrbnb/backend
```

---

### 2️⃣ Install Dependencies

#### Node.js Microservices

Run `npm install` inside each Node.js service directory:

```bash
# Hotel Service
cd hotelservice/HotelService && npm install && cd ../..

# Booking Service
cd bookingservice/Bookingservice && npm install && cd ../..

# Notification Service
cd Notificationservice && npm install && cd ..
```

#### Go Microservices

Go dependencies download automatically on run, or pre-fetch them:

```bash
cd authentication && go mod download && cd ..
cd reviewservice && go mod download && cd ..
```

---

### 3️⃣ Environment Setup

Configure `.env` files for each service as needed. Example key variables:

**`backend/authentication/.env`**
```env
PORT=3001
JWT_SECRET=your_jwt_secret_key
DB_URL=user:password@tcp(127.0.0.1:3306)/auth_db
```

**`backend/hotelservice/HotelService/.env`**
```env
PORT=3000
REDIS_HOST=127.0.0.1
REDIS_PORT=6379
DB_HOST=127.0.0.1
```

**`backend/bookingservice/Bookingservice/.env`**
```env
PORT=3003
DATABASE_URL="postgresql://user:password@localhost:5432/booking_db"
```

**`backend/Notificationservice/.env`**
```env
PORT=3002
REDIS_HOST=127.0.0.1
REDIS_PORT=6379
MAIL_USER=your_email@gmail.com
MAIL_PASS=your_app_password
```

---

### 4️⃣ Start All Microservices

Run the master orchestration script from the `backend/` folder:

```bash
chmod +x run-all.sh
./run-all.sh
```

Output:
```text
Starting Airbnb clone backend microservices...
Starting Authentication Service on port 3001...
Starting Review Service on port 8081...
Starting Booking Service on port 3003...
Starting Hotel Service on port 3000...
Starting Notification Service on port 3002...
All services started! Press Ctrl+C to stop all of them.
```

---

## 📡 Main API Routes

### 🔑 Authentication Service (`http://localhost:3001`)
- `POST /signup` - Register a new user account
- `POST /login` - Authenticate user & receive JWT token
- `GET /users` - Fetch all users
- `GET /users/{id}` - Fetch user by ID (Requires JWT + `users`/`admin` role)
- `GET /ping` - Health check endpoint

### 🏨 Hotel Service (`http://localhost:3000`)
- `GET /api/v1/hotels` - List all hotels
- `POST /api/v1/hotels` - Add a new hotel property
- `GET /api/v1/rooms` - Fetch rooms for a property
- `GET /api/v1/ping` - Health check

### 📅 Booking Service (`http://localhost:3003`)
- `POST /api/v1/bookings` - Create a room reservation
- `GET /api/v1/bookings/:id` - Get booking details
- `GET /api/v1/ping` - Health check

### ⭐ Review Service (`http://localhost:8081`)
- `POST /reviews` - Submit property review & rating
- `GET /reviews/hotel/:hotel_id` - Fetch reviews for a specific hotel

### 📧 Notification Service (`http://localhost:3002`)
- Internal background process listening on Redis queue for email events (Welcome emails, booking notifications).

---

## 🗺️ Roadmap & Next Steps

- [ ] **API Gateway**: Implement centralized gateway (e.g. Express Gateway / Kong / NGINX / Go Gateway) for request routing, auth verification, and rate limiting.
- [ ] **Frontend Web Application**: Build a modern, responsive user interface (React.js / Next.js + Tailwind CSS) featuring interactive search, property listings, booking modal, and user dashboards.
- [ ] **Containerization**: Create `Dockerfile`s for each service and a top-level `docker-compose.yml` for simplified deployment.
- [ ] **CI/CD Pipeline**: GitHub Actions for automated linting, testing, and continuous deployment.

---

## 📄 License

This project is open-source and available under the [MIT License](LICENSE).
