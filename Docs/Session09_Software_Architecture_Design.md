# Session 9 - Software Architecture Design and Design Patterns
## Smart Hospital Management System

**Subject:** Software Development Process (SDP)
**Batch:** 2026-27 IT-B
**Enrollment No.:** 202401100600082

---

## Table of Contents
1. [Introduction](#introduction)
2. [Architecture 1 - Layered Architecture](#1-layered-architecture)
3. [Architecture 2 - Client-Server Architecture](#2-client-server-architecture)
4. [Architecture 3 - Microservices Architecture (Conceptual)](#3-microservices-architecture-conceptual)
5. [Design Patterns Used](#4-design-patterns-used)
6. [Comparison Summary](#5-comparison-summary)

---

## Introduction

The **Smart Hospital Management System** is a cloud-native platform (hosted on AWS) that enables:
- **Patients** to view their medical history, book recommended diagnostic tests online, and receive medicine/treatment reminders.
- **Hospital Staff / Admin** to maintain patient records, recommend tests, issue prescriptions, and manage schedules.

This document presents the system architecture from three complementary viewpoints:

| Architecture Style | Purpose |
|---|---|
| Layered (N-Tier) | Separation of concerns - Presentation, Business Logic, Data |
| Client-Server | How clients (browsers/mobile) communicate with backend services |
| Microservices (Conceptual) | Independent, deployable services aligned to business capabilities |

---

## 1. Layered Architecture

A **Layered (N-Tier) Architecture** organises the system into horizontal layers where each layer only communicates with the layer directly below it.

### 1.1 Layer Diagram

```
+------------------------------------------------------------------+
|                     PRESENTATION LAYER                           |
|   +--------------------+        +--------------------------+    |
|   |   Patient Portal   |        |  Staff / Admin Portal    |    |
|   |  (React SPA)       |        |  (React SPA)             |    |
|   |  - My Records      |        |  - Patient List          |    |
|   |  - Book a Test     |        |  - Add/Update Records    |    |
|   |  - Reminders       |        |  - Manage Bookings       |    |
|   +--------------------+        +--------------------------+    |
|            Hosted on Amazon S3 + CloudFront (CDN)               |
+-----------------------------+------------------------------------+
                              |  HTTPS / REST API calls
+-----------------------------v------------------------------------+
|                      APPLICATION LAYER                           |
|                  (Business Logic / Services)                     |
|                                                                  |
|  +------------------+  +--------------+  +-----------------+   |
|  |  Patient Records |  | Test Booking |  | Reminder Service|   |
|  |     Service      |  |   Service    |  |(Notifications)  |   |
|  | - CRUD records   |  | - List tests |  | - Schedule SMS  |   |
|  | - Attach docs    |  | - Book slot  |  | - Send Email    |   |
|  | - Role checks    |  | - Update     |  | - EventBridge   |   |
|  +------------------+  +--------------+  +-----------------+   |
|               AWS Lambda Functions (Serverless)                 |
|               Amazon API Gateway (Auth: Cognito)                |
+-----------------------------+------------------------------------+
                              |  DB queries / File operations
+-----------------------------v------------------------------------+
|                        DATA LAYER                                |
|                                                                  |
|  +------------------------------+   +------------------------+  |
|  |       Amazon RDS             |   |      Amazon S3         |  |
|  |   (PostgreSQL / MySQL)       |   |  (Document Storage)    |  |
|  |                              |   |                        |  |
|  |  Tables:                     |   |  Objects:              |  |
|  |  - patients                  |   |  - prescriptions/      |  |
|  |  - staff                     |   |  - reports/            |  |
|  |  - test_catalog              |   |  - scans/              |  |
|  |  - bookings                  |   |                        |  |
|  |  - prescriptions             |   |                        |  |
|  |  - reminders                 |   |                        |  |
|  +------------------------------+   +------------------------+  |
+------------------------------------------------------------------+
```

### 1.2 Layer Responsibilities

| Layer | Technology | Responsibility |
|---|---|---|
| **Presentation** | React SPA, S3, CloudFront | UI rendering, form inputs, role-specific views |
| **Application** | AWS Lambda, API Gateway, Cognito | Business rules, auth enforcement, service orchestration |
| **Data** | Amazon RDS, Amazon S3 | Persistent storage, file storage, transactions |

### 1.3 Key Design Principles
- **Strict Layer Isolation** - the Presentation Layer never directly touches the database.
- **Role-Based Access Enforcement** at the Application Layer (not just hidden in UI).
- **Stateless Lambda** functions keep the Application Layer horizontally scalable.

---

## 2. Client-Server Architecture

The **Client-Server Architecture** describes how the two types of clients communicate with the centralised backend server cluster over the network.

### 2.1 High-Level Client-Server Diagram

```
  CLIENT SIDE                           SERVER SIDE
  -----------                           -----------------------------------------------
                                                              +------------------+
  +---------------+   HTTPS Request                          |   Amazon Cognito |
  | Patient       | ------------------------------------------> (Authentication/ |
  | Browser/App   | <------------------------------------------  Authorization)  |
  |               |   JWT Token / 401 Unauthorized           +--------+---------+
  +---------------+                                                   | Verified Token
                                                                       |
  +---------------+   HTTPS + JWT Token                     +---------v--------+
  | Staff / Admin | ------------------------------------------> Amazon API     |
  | Browser/App   | <------------------------------------------  Gateway       |
  |               |   JSON Response / Error                  +---------+--------+
  +---------------+                                                    | Route to Lambda
                                                                        |
                                                       +----------------v-------------+
                                                       |         AWS Lambda           |
                                                       |     (Application Logic)      |
                                                       |                              |
                                                       |  /patients/* -> Records Fn   |
                                                       |  /tests/*    -> Booking Fn   |
                                                       |  /reminders/*-> Reminder Fn  |
                                                       +----------------+-------------+
                                                                        |
                                                      +-----------------v-------------+
                                                      |         Data Stores           |
                                                      |  +-----------+ +-----------+  |
                                                      |  | Amazon    | | Amazon S3 |  |
                                                      |  |   RDS     | |           |  |
                                                      |  +-----------+ +-----------+  |
                                                      +-------------------------------+
```

### 2.2 Communication Flow (Step-by-Step)

```
Step 1:  Client opens the portal (Patient or Staff)
Step 2:  Client is redirected to Amazon Cognito Hosted UI for login
Step 3:  Cognito validates credentials and returns a JWT (ID + Access Token)
Step 4:  Client stores the JWT and includes it in every API request header:
           Authorization: Bearer <JWT>
Step 5:  API Gateway validates the JWT via Cognito Authorizer
         -> If invalid/expired -> 401 Unauthorized returned to client
         -> If valid -> request routed to the appropriate Lambda function
Step 6:  Lambda function performs role check from JWT claims
         (e.g., group = Patient cannot access other patients data)
Step 7:  Lambda queries Amazon RDS or Amazon S3 as needed
Step 8:  Lambda returns a JSON response to API Gateway
Step 9:  API Gateway returns the response to the client
Step 10: Client renders the data in the UI
```

### 2.3 Client Capabilities by Role

| Feature | Patient Client | Staff Client | Admin Client |
|---|---|---|---|
| View own medical records | Yes | Yes (all patients) | Yes |
| Create/update records | No | Yes | Yes |
| Book a test slot | Yes | No | Yes |
| View/manage all bookings | No | Yes | Yes |
| Manage staff accounts | No | No | Yes |
| Receive reminders | Yes | No | No |

### 2.4 Notification Sub-System (Server-Initiated)

```
  Amazon EventBridge Scheduler
         |
         |  Cron trigger (e.g., daily 8 AM)
         v
  Reminder Lambda
         |
         +---> Amazon SNS ---> Patient Mobile (SMS)
         |
         +---> Amazon SES ---> Patient Email
```

---

## 3. Microservices Architecture (Conceptual)

In the **Microservices Architecture**, the application logic is decomposed into small, independently deployable services. Each service owns its own data and communicates with other services via events or API calls.

### 3.1 Service Decomposition Diagram

```
                       +------------------------------------+
                       |          API GATEWAY               |
                       |      (Single Entry Point)          |
                       +----------------+-------------------+
                                        |
           +----------------------------+---------------------------+
           |                            |                          |
           v                            v                          v
+--------------------+    +---------------------+    +--------------------+
|   Auth Service     |    |  Patient Records    |    |  Test Booking      |
|                    |    |     Service         |    |    Service         |
| - Registration     |    |                     |    |                    |
| - Login (Cognito)  |    | - Create patient    |    | - List test catalog|
| - Token issuance   |    | - View history      |    | - Book a slot      |
| - Role management  |    | - Upload docs       |    | - Cancel booking   |
|                    |    | - Search records    |    | - Track status     |
| Data Store:        |    |                     |    |                    |
| Cognito User Pool  |    | Data Store:         |    | Data Store:        |
|                    |    | RDS (patients)      |    | RDS (test_catalog, |
+--------------------+    | S3 (documents)      |    |      bookings)     |
                          +----------+----------+    +---------+----------+
                                     |                         |
                            Emits: PatientCreated    Emits: BookingMade
                                     |                         |
                          +----------v-------------------------v----------+
                          |       Event Bus (Amazon EventBridge)         |
                          +-----------------------------+------------------+
                                                        |
                                           +------------v-----------+
                                           |    Reminder Service    |
                                           |                        |
                                           | - Listen for events    |
                                           | - Schedule reminders   |
                                           | - Send SMS (SNS)       |
                                           | - Send Email (SES)     |
                                           |                        |
                                           | Data Store:            |
                                           | RDS (reminders,        |
                                           |      prescriptions)    |
                                           +------------------------+
```

### 3.2 Microservice Definitions

| Service | Responsibility | Tech Stack | Data Store |
|---|---|---|---|
| **Auth Service** | User registration, login, token management, role assignment | Amazon Cognito, Lambda | Cognito User Pool |
| **Patient Records Service** | CRUD on patient profiles, medical history, document upload/download | Lambda, API Gateway | RDS (patients), S3 |
| **Test Booking Service** | Browse test catalogue, create/cancel bookings, update booking status | Lambda, API Gateway | RDS (test_catalog, bookings) |
| **Reminder Service** | Subscribe to events, schedule and send medicine/treatment reminders | Lambda, EventBridge, SNS, SES | RDS (reminders, prescriptions) |
| **Notification Service** | Delivery of emails/SMS (abstracted from Reminder Service) | SNS, SES | None (stateless) |

### 3.3 Inter-Service Communication

```
  Synchronous (REST):
  ------------------
  Client -> API Gateway -> Lambda (any service)
  (Used for: real-time queries, CRUD operations)

  Asynchronous (Event-Driven):
  ----------------------------
  Patient Records Service --[PatientCreated]--> EventBridge --> Reminder Service
  Test Booking Service    --[BookingMade]-----> EventBridge --> Reminder Service

  (Used for: decoupled notifications, background jobs)
```

### 3.4 Benefits of Microservices for this System

| Benefit | Application |
|---|---|
| **Independent deployment** | Reminder Service can be updated without touching Patient Records |
| **Fault isolation** | If Test Booking fails, patients can still view their medical history |
| **Scalability** | Test Booking Service scaled up during high-demand periods |
| **Technology flexibility** | Reminder Service can switch notification providers without impacting others |

---

## 4. Design Patterns Used

| Pattern | Where Applied | Rationale |
|---|---|---|
| **MVC (Model-View-Controller)** | Frontend React portals | Separates UI (View), data fetching (Model), and user actions (Controller) |
| **Repository Pattern** | Lambda functions accessing RDS | Abstracts DB queries behind a data-access layer; easy to swap DB or mock in tests |
| **Facade Pattern** | API Gateway | Provides a single unified API surface, hiding internal Lambda complexity |
| **Observer Pattern** | EventBridge event bus | Services subscribe to events emitted by other services (loose coupling) |
| **Strategy Pattern** | Notification delivery | Switches between SMS (SNS) or Email (SES) based on patient preference |
| **Singleton Pattern** | DB connection pool in Lambda | Reuses a single RDS connection pool across warm Lambda invocations |

---

## 5. Comparison Summary

| Dimension | Layered | Client-Server | Microservices |
|---|---|---|---|
| **Primary goal** | Separation of concerns | Network communication model | Business capability decomposition |
| **Coupling** | Medium (layer-to-layer) | Medium (client-server contract) | Low (event-driven, independent) |
| **Scalability** | Per-layer | Server-side | Per-service |
| **Deployment** | All layers together | Server independently | Each service independently |
| **Best for** | Explaining internal structure | Explaining network topology | Explaining service boundaries |
| **Complexity** | Low | Medium | High |

---

## Deliverable Summary

This document satisfies the **Session 9 Deliverable: System Architecture Diagram** by presenting:

1. Layered Architecture - 3-tier view (Presentation -> Application -> Data)
2. Client-Server Architecture - Network-level view of Patient/Staff portals communicating with AWS backend
3. Microservices (Conceptual) - 4 independent services with event-driven inter-service communication
4. Design Patterns - 6 patterns identified and applied to the Smart Hospital Management System

---

*Prepared for SDP Session 9 | Batch 2026-27 IT-B | Roll No. 202401100600082*
