layout: default
title: E-Commerce Microservices Backend
---

# 🧩 E-Commerce Microservices Backend (Java + Spring Boot)

A **backend-only** microservices architecture built using **Java + Spring Boot**, demonstrating real-world design patterns for scalable backend systems.  
It includes **User**, **Product**, and **Payment** services — communicating via REST APIs — integrated with **MySQL**, **Redis**, and **Elasticsearch**.

---

## 🔗 Repositories

- **User Service (internal)** → https://github.com/zaidsamad12/UserService
  *Provides user context, roles, and authentication — consumed internally by Product Service.*

- **Product Service (public API)** → https://github.com/zaidsamad12/ProductService_Proxy
  *Manages products, search, caching, and payment initiation.*

- **Payment Service (external integrations)** → https://github.com/zaidsamad12/PaymentService
  *Integrates with Razorpay and Stripe to create and verify payment links.*

---

## 🏗️ System Architecture

                ┌─────────────────────────────┐
                │       User Service (INT)     │
                │ - User profiles, roles       │
                │ - Called internally by       │
                │   Product Service            │
                └──────────────┬───────────────┘
                               │ REST (internal)
                               ▼
                ┌─────────────────────────────┐
                │      Product Service         │
                │ - CRUD + search on products  │
                │ - MySQL (source of truth)    │
                │ - Redis (cache layer)        │
                │ - Elasticsearch (search/sort)│
                │ - Secured via JWT auth       │
                │ - Calls Payment Service      │
                └──────────────┬───────────────┘
                               │ REST
                               ▼
                ┌─────────────────────────────┐
                │      Payment Service         │
                │ - Integrates Razorpay/Stripe │
                │ - Generates payment link     │
                │ - Exposes /webhooks for      │
                │   payment verification       │
                └─────────────────────────────┘

**MySQL** → Main database (source of truth)  
**Redis** → Cache for product reads  
**Elasticsearch** → Full-text search + sorting (`title.keyword`, `price`)  
**JWT-based Auth** → Enforced on `/products/**` endpoints  

---

## ⚙️ Tech Stack

| Category | Technologies |
|-----------|---------------|
| **Language** | Java 17 |
| **Frameworks** | Spring Boot, Spring Data JPA, Spring Security |
| **Databases** | MySQL (RDS/local) |
| **Cache** | Redis |
| **Search** | Elasticsearch |
| **Auth** | OAuth2 Resource Server (JWT) |
| **Build** | Maven |
| **Integration** | Razorpay & Stripe APIs |

---

## 🔐 Endpoint Overview

### Product Service
| Endpoint | Description | Auth |
|-----------|--------------|------|
| `GET /products/{id}` | Get product by ID (cached) | ✅ JWT |
| `GET /products` | List all products (cached, paginated) | ✅ JWT |
| `POST /products` | Add new product (saves to DB + ES) | ✅ JWT |
| `PUT /products/{id}` | Update product (DB + ES sync) | ✅ JWT |
| `GET /search?q={text}` | Search products via Elasticsearch | ❌ Public |

### Payment Service
| Endpoint | Description |
|-----------|-------------|
| `POST /payments` | Create payment order (Razorpay/Stripe) |
| `POST /webhooks/**` | Receive gateway callbacks (signature verified) |

---

## 💳 Payment Flow

```mermaid
sequenceDiagram
  participant PS as Product Service
  participant Pay as Payment Service
  participant RP as Razorpay
  participant ST as Stripe

  PS->>Pay: Create payment (amount, currency, metadata)
  alt Razorpay
    Pay->>RP: Create order via Razorpay API
    RP-->>Pay: { payment_link }
  else Stripe
    Pay->>ST: Create checkout session
    ST-->>Pay: { checkout_url }
  end
  Pay-->>PS: { payment_link }
  RP-->>Pay: webhook (success/failure)
  ST-->>Pay: webhook (success/failure)
  Pay-->>PS: update payment status

<!-- Mermaid JS (client-side render) --> <script src="https://unpkg.com/mermaid@10/dist/mermaid.min.js"></script> <script> mermaid.initialize({ startOnLoad: true, securityLevel: 'loose' }); </script>
