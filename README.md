<div align="center">

# 🏦 RevPay — Digital Payment Platform

### Microservices Architecture | Spring Boot 3 | Angular | Docker

![Java](https://img.shields.io/badge/Java-21-orange?style=for-the-badge&logo=openjdk)
![Spring Boot](https://img.shields.io/badge/Spring_Boot-3.3.6-brightgreen?style=for-the-badge&logo=springboot)
![Spring Cloud](https://img.shields.io/badge/Spring_Cloud-2023.0.3-brightgreen?style=for-the-badge&logo=spring)
![Angular](https://img.shields.io/badge/Angular-21-red?style=for-the-badge&logo=angular)
![MySQL](https://img.shields.io/badge/MySQL-8.0-blue?style=for-the-badge&logo=mysql)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=for-the-badge&logo=docker)

**RevPay** is a full-stack digital payment platform modernized from a monolithic architecture into a cloud-native microservices system. It supports personal and business users with wallets, transfers, invoices, loans, and real-time notifications.

</div>

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Architecture](#-architecture)
- [Services](#-services)
- [Tech Stack](#-tech-stack)
- [Getting Started](#-getting-started)
  - [Prerequisites](#prerequisites)
  - [Run with Docker](#run-with-docker-recommended)
  - [Run Locally](#run-locally-development)
- [API Endpoints](#-api-endpoints)
- [Project Structure](#-project-structure)
- [Environment Configuration](#-environment-configuration)
- [Database](#-database)
- [Security](#-security)
- [Features](#-features)

---

## 🌐 Overview

RevPay was originally a Spring Boot monolith. This project modernizes it into **6 independent microservices** following Domain-Driven Design principles. Each service owns its bounded context, its own database schema, and its own lifecycle.

### Key Design Decisions

| Decision | Choice | Reason |
|----------|--------|--------|
| Service Communication | OpenFeign (synchronous) | Simple, readable, fail-fast |
| Service Discovery | Eureka Server | Spring Cloud native, dashboard included |
| API Routing | Spring Cloud Gateway | Reactive, circuit breaker support |
| Authentication | JWT (HS256, stateless) | No session, no DB call on every request |
| Database per Service | MySQL (separate schemas) | Bounded context isolation |
| Containerization | Docker + docker-compose | One command to run everything |
| CORS | Gateway only | Single source of truth, no duplicate headers |

---

## 🏗️ Architecture

```
Browser (localhost:4200 dev / localhost:80 docker)
         │
         ▼
┌─────────────────────────────────────────────────┐
│              Nginx (Docker only)                │
│         Serves Angular + Proxies /api/*         │
└─────────────────┬───────────────────────────────┘
                  │ /api/*
                  ▼
┌─────────────────────────────────────────────────┐
│              API Gateway  :8080                 │
│   Routing · CORS · Circuit Breaker · Eureka     │
└──┬──────┬──────┬──────┬──────┬──────────────────┘
   │      │      │      │      │
   ▼      ▼      ▼      ▼      ▼
[User] [Wallet] [Invoice] [Loan] [Notify]
:8081  :8082    :8083    :8084  :8085
  │      │        │        │       │
  ▼      ▼        ▼        ▼       ▼
[revpay_users] [revpay_wallet] [revpay_invoice]
               [revpay_loan]  [revpay_notifications]
                       │
                       ▼
              ┌─────────────────┐
              │  MySQL :3306    │
              │  (5 schemas)    │
              └─────────────────┘
                       │
                       ▼
              ┌─────────────────┐
              │  mysql_data     │
              │  Docker Volume  │
              └─────────────────┘

Eureka Server :8761  ←  all services register here
```

### Feign Call Map

```
wallet-service   →  user-service        (auto-create wallet)
wallet-service   →  notification-service (transaction alerts)
wallet-service   →  invoice-service      (dashboard counts)
wallet-service   →  loan-service         (dashboard counts)

invoice-service  →  user-service         (validate users)
invoice-service  →  wallet-service       (execute payment)
invoice-service  →  notification-service (invoice alerts)

loan-service     →  user-service         (get borrower name)
loan-service     →  wallet-service       (credit/debit wallet)
loan-service     →  notification-service (loan status alerts)

user-service     →  loan-service         (pending loan count for admin)
```

---

## 🔧 Services

| Service | Port | Database | Responsibility |
|---------|------|----------|----------------|
| **User Service** | 8081 | revpay_users | Auth, registration, JWT generation, admin management |
| **Wallet Service** | 8082 | revpay_wallet | Balances, transfers, top-up, payment methods, dashboard |
| **Invoice Service** | 8083 | revpay_invoice | Invoice lifecycle, money requests |
| **Loan Service** | 8084 | revpay_loan | Loan applications, EMI schedule, auto-debit |
| **Notification Service** | 8085 | revpay_notifications | In-app notifications (passive receiver) |
| **API Gateway** | 8080 | None | Routing, CORS, circuit breaker, load balancing |
| **Eureka Server** | 8761 | None | Service registry and discovery |

---

## 💻 Tech Stack

### Backend
- **Java 21** — LTS version with virtual threads support
- **Spring Boot 3.3.6** — Core framework
- **Spring Cloud 2023.0.3** — Gateway, Eureka, OpenFeign, Resilience4j
- **Spring Security** — JWT-based stateless authentication
- **Spring Data JPA + Hibernate** — ORM and database access
- **JJWT 0.11.5** — JWT generation and validation
- **Lombok** — Boilerplate reduction
- **Maven** — Build tool

### Frontend
- **Angular 21** — SPA framework
- **jsPDF + jspdf-autotable** — Client-side PDF export
- **Nginx** — Production static file serving + API proxy

### Infrastructure
- **MySQL 8.0** — Relational database (5 schemas)
- **Docker + Docker Compose** — Containerization
- **Eureka Server** — Service discovery
- **Spring Cloud Gateway** — API Gateway (WebFlux/reactive)

---

## 🚀 Getting Started

### Prerequisites

| Tool | Version | Required For |
|------|---------|-------------|
| Java JDK | 21+ | Running locally |
| Maven | 3.8+ | Building JARs |
| Docker Desktop | Latest | Docker setup |
| Node.js + npm | 20+ | Frontend (local only) |
| MySQL | 8.0 | Running locally (not needed for Docker) |

---

### Run with Docker (Recommended)

**Everything starts with one command.**

#### Step 1 — Clone the repository
```bash
git clone https://github.com/your-username/revpay-microservices.git
cd revpay-microservices
```

#### Step 2 — Build all JARs
```bash
mvn clean install -DskipTests
```

#### Step 3 — Start all containers
```bash
docker-compose up --build
```

> First build takes **10–15 minutes** (downloads base images, runs npm install).  
> Subsequent starts take **~2 minutes** (cached layers).

#### Step 4 — Verify everything is running
```bash
docker ps
```

All 9 containers should show status **Up**:

```
revpay-mysql        ← Up (healthy)
revpay-eureka       ← Up
revpay-user         ← Up
revpay-notification ← Up
revpay-wallet       ← Up
revpay-invoice      ← Up
revpay-loan         ← Up
revpay-gateway      ← Up
revpay-frontend     ← Up
```

#### Step 5 — Access the application

| URL | What |
|-----|------|
| http://localhost | Angular Frontend |
| http://localhost:8080 | API Gateway |
| http://localhost:8761 | Eureka Dashboard (admin / admin123) |

#### Default Admin Account
```
Email:    admin@revpay.com
Password: Admin@123456
```

---

#### Docker Commands Reference

```bash
# Start everything (first time)
docker-compose up --build

# Start everything (after first build)
docker-compose up

# Start in background
docker-compose up -d

# Stop everything (DATA IS SAFE)
docker-compose down

# Stop and DELETE all data ⚠️
docker-compose down -v

# View logs of all services
docker-compose logs -f

# View logs of one service
docker-compose logs -f wallet-service

# Rebuild only one service after code change
mvn clean install -DskipTests
docker-compose up --build --no-deps -d wallet-service

# Check running containers
docker ps

# Free build cache (safe, frees 5-7GB)
docker builder prune -f
```

---

### Run Locally (Development)

#### Step 1 — Start MySQL
Make sure MySQL 8.0 is running on port 3306.  
Databases are created automatically — no manual setup needed.

#### Step 2 — Build all services
```bash
cd revpay-microservices
mvn clean install -DskipTests
```

#### Step 3 — Start services in order (separate terminals)

```bash
# Terminal 1 — ALWAYS FIRST
cd eureka-server && mvn spring-boot:run

# Terminal 2
cd user-service && mvn spring-boot:run

# Terminal 3
cd wallet-service && mvn spring-boot:run

# Terminal 4
cd invoice-service && mvn spring-boot:run

# Terminal 5
cd loan-service && mvn spring-boot:run

# Terminal 6
cd notification-service && mvn spring-boot:run

# Terminal 7 — ALWAYS LAST
cd api-gateway && mvn spring-boot:run
```

#### Step 4 — Start Angular frontend
```bash
cd revpay-frontend
npm install
ng serve
```

Open http://localhost:4200

> ⚠️ **Order matters.** Gateway must start last. Eureka must start first.  
> Wait for Eureka dashboard to show all services before testing.

---

## 📡 API Endpoints

All endpoints go through the API Gateway on port **8080**.

### Authentication (Public)
```
POST   /api/auth/register/personal   Register personal account
POST   /api/auth/register/business   Register business account
POST   /api/auth/login               Login, returns JWT token
POST   /api/auth/forgot-password     Request password reset
POST   /api/auth/reset-password      Reset password with token
```

### Wallet & Transactions (JWT Required)
```
GET    /api/accounts                        Get my wallet accounts
POST   /api/transactions/transfer           Send money by email
POST   /api/transactions/top-up             Add funds via payment method
GET    /api/transactions/history            Paginated transaction history
GET    /api/transactions/export             Export with date range filter
POST   /api/payments/methods               Add card or bank account
GET    /api/payments/methods               List saved payment methods
DELETE /api/payments/methods/{id}          Remove payment method
GET    /api/dashboard                       Full dashboard
```

### Invoices (JWT Required)
```
POST   /api/invoices                        Create invoice
POST   /api/invoices/{id}/send             Send to recipient
POST   /api/invoices/{id}/pay              Pay invoice
POST   /api/invoices/{id}/dispute          Dispute invoice
POST   /api/invoices/{id}/cancel           Cancel invoice
GET    /api/invoices/issued                My created invoices
GET    /api/invoices/received              Invoices to pay
POST   /api/money-requests                 Request money from user
POST   /api/money-requests/{id}/accept    Accept money request
POST   /api/money-requests/{id}/reject    Reject money request
```

### Loans (JWT Required)
```
POST   /api/loans/apply                    Apply for loan
GET    /api/loans                          My loan applications
GET    /api/loans/{id}                     Loan + EMI schedule
POST   /api/emis/{emiId}/pay              Pay EMI
POST   /api/emis/loan/{loanId}/auto-debit/toggle  Toggle auto-debit
```

### Notifications (JWT Required)
```
GET    /api/notifications                  All notifications
GET    /api/notifications/unread-count     Unread badge count
POST   /api/notifications/{id}/read       Mark as read
POST   /api/notifications/read-all        Mark all as read
DELETE /api/notifications/{id}            Delete notification
```

### Admin (ADMIN role required)
```
GET    /api/admin/users                   List all users
POST   /api/admin/users/{id}/toggle       Enable/disable user
GET    /api/admin/stats                   Platform statistics
GET    /api/admin/loans                   All loan applications
POST   /api/admin/loans/{id}/approve      Approve loan
POST   /api/admin/loans/{id}/reject       Reject loan
```

---

## 📁 Project Structure

```
revpay-microservices/
│
├── pom.xml                          ← Parent POM (all modules)
├── docker-compose.yml               ← All 9 containers
│
├── eureka-server/                   ← Service Registry (port 8761)
│   ├── Dockerfile
│   └── src/main/resources/
│       ├── application.yml
│       └── application-docker.properties
│
├── api-gateway/                     ← API Gateway (port 8080)
│   ├── Dockerfile
│   └── src/main/resources/
│       ├── application.yml          ← Routes + CORS + Circuit Breaker
│       └── application-docker.properties
│
├── user-service/                    ← Auth + Admin (port 8081)
│   ├── Dockerfile
│   └── src/main/java/com/revpay/
│       ├── controller/              ← AuthController, AdminController
│       ├── service/                 ← AuthService, AdminService
│       ├── security/                ← JwtUtil, JwtAuthFilter, SecurityConfig
│       ├── entity/                  ← User, Account
│       ├── repository/
│       └── dto/
│
├── wallet-service/                  ← Transactions + Dashboard (port 8082)
│   ├── Dockerfile
│   └── src/main/java/com/revpay/
│       ├── controller/              ← AccountController, TransactionController
│       ├── service/                 ← TransactionService, DashboardService
│       ├── client/                  ← UserServiceClient, NotificationServiceClient
│       └── entity/                  ← Account, Transaction, PaymentMethod
│
├── invoice-service/                 ← Invoices + Money Requests (port 8083)
│   ├── Dockerfile
│   └── src/main/java/com/revpay/
│
├── loan-service/                    ← Loans + EMI (port 8084)
│   ├── Dockerfile
│   └── src/main/java/com/revpay/
│
├── notification-service/            ← Notifications (port 8085)
│   ├── Dockerfile
│   └── src/main/java/com/revpay/
│
└── revpay-frontend/ (separate repo or folder)
    ├── Dockerfile                   ← Multi-stage build
    ├── nginx.conf                   ← Angular routing + API proxy
    └── src/
```

---

## ⚙️ Environment Configuration

Each service has two configuration files:

| File | Used When | Key Settings |
|------|-----------|-------------|
| `application.properties` | Local development | `localhost` URLs |
| `application-docker.properties` | Docker (`SPRING_PROFILES_ACTIVE=docker`) | Container name URLs |

### Local vs Docker Config Difference

```properties
# LOCAL (application.properties)
spring.datasource.url=jdbc:mysql://localhost:3306/revpay_wallet
eureka.client.service-url.defaultZone=http://localhost:8761/eureka/

# DOCKER (application-docker.properties)
spring.datasource.url=jdbc:mysql://mysql:3306/revpay_wallet
eureka.client.service-url.defaultZone=http://eureka-server:8761/eureka/
```

### JWT Configuration (same across all services)
```properties
app.jwt.secret=UmV2UGF5U3VwZXJTZWNyZXRLZXlGb3JKV1RUb2tlbkdlbmVyYXRpb25NdXN0QmVBdExlYXN0MjU2Qml0c0xvbmc=
app.jwt.expiration-ms=86400000        # 24 hours
app.jwt.refresh-expiration-ms=604800000  # 7 days
```

---

## 🗄️ Database

**One MySQL container — 5 separate schemas.**

| Schema | Owned By | Key Tables |
|--------|----------|------------|
| `revpay_users` | User Service | users, accounts |
| `revpay_wallet` | Wallet Service | accounts, transactions, payment_methods |
| `revpay_invoice` | Invoice Service | invoices, invoice_items, money_requests |
| `revpay_loan` | Loan Service | loans, emis |
| `revpay_notifications` | Notification Service | notifications |

Schemas are **created automatically** on first connection via:
```
createDatabaseIfNotExist=true  (in JDBC URL)
spring.jpa.hibernate.ddl-auto=update  (Hibernate creates tables)
```

> ⚠️ `ddl-auto=update` is for development only.  
> Use Flyway or Liquibase for production schema migrations.

---

## 🔐 Security

### JWT Authentication Flow
```
1. POST /api/auth/login → User Service validates credentials
2. BCryptPasswordEncoder (strength 12) verifies password
3. JWT generated: { sub: email, role: PERSONAL/BUSINESS/ADMIN, exp: 24h }
4. All subsequent requests include: Authorization: Bearer {token}
5. JwtAuthFilter in each service validates signature — NO database call
6. Email + role extracted from claims, set in SecurityContext
```

### Auth Strategy Per Service

| Service | Strategy |
|---------|----------|
| User Service | DaoAuthenticationProvider — queries DB, verifies BCrypt hash |
| All other services | JWT claims only — zero DB queries for auth |

### Role-Based Access Control

| Role | Access |
|------|--------|
| `PERSONAL` | Own wallet, transfers, invoices, money requests |
| `BUSINESS` | All personal features + business invoices, loans |
| `ADMIN` | All features + user management, loan approval |

### Internal API Security
Internal endpoints (`/api/*/internal/**`) are `permitAll()` — no JWT required.  
They are unreachable from outside — the API Gateway has no routes pointing to them.  
Only accessible within the Docker bridge network.

---

## ✨ Features

### Personal Users
- ✅ Register and login with JWT authentication
- ✅ Wallet balance management
- ✅ Send and receive money by email
- ✅ Top-up wallet via saved payment methods
- ✅ Full transaction history with credit/debit display
- ✅ Export transactions as PDF (jsPDF)
- ✅ Send and respond to money requests
- ✅ Apply for personal loans with EMI schedule
- ✅ Manual and auto-debit EMI payments
- ✅ Real-time in-app notifications with bell badge
- ✅ Forgot/reset password flow

### Business Users
- ✅ All personal features
- ✅ Create invoices with line items and tax
- ✅ Send, pay, dispute, and cancel invoices
- ✅ Business loan applications
- ✅ Dashboard with balance, pending counts, recent transactions

### Admin
- ✅ View and manage all users
- ✅ Enable/disable user accounts
- ✅ View and approve/reject loan applications
- ✅ Platform statistics (user counts, pending loans)

### Infrastructure
- ✅ Eureka service discovery
- ✅ API Gateway with 14 routes
- ✅ Resilience4j circuit breakers on all routes
- ✅ Centralized CORS in Gateway
- ✅ Docker Compose — one command startup
- ✅ Data persistence via named Docker volumes
- ✅ Environment-specific configuration (local/docker profiles)
- ✅ Admin account seeded on first startup

---

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/your-feature`)
3. Commit your changes (`git commit -m 'Add your feature'`)
4. Push to the branch (`git push origin feature/your-feature`)
5. Open a Pull Request

---

## 📄 License

This project is for educational purposes as part of a Cognizant training program.

---

<div align="center">

**Built with Spring Boot · Angular · Docker · Spring Cloud**

</div>
